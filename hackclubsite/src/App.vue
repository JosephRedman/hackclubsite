<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import './style.css'


const TZ = 'Europe/London'
const joinCode = 'YOUR-CODE' // replace with your real join code
const now = ref(new Date())
let timer

onMounted(() => {
  timer = setInterval(() => (now.value = new Date()), 1000)
})
onUnmounted(() => clearInterval(timer))

// Get the wall-clock parts of a date as seen in London
function londonParts(date) {
  const parts = new Intl.DateTimeFormat('en-GB', {
    timeZone: TZ,
    year: 'numeric', month: '2-digit', day: '2-digit',
    hour: '2-digit', minute: '2-digit', second: '2-digit',
    hourCycle: 'h23',
  }).formatToParts(date)
  return Object.fromEntries(parts.map(p => [p.type, Number(p.value)]))
}

// Convert a London wall-clock time into a real Date (handles GMT/BST)
function londonToDate(y, m, d, h, min) {
  const guess = Date.UTC(y, m - 1, d, h, min)
  const p = londonParts(new Date(guess))
  const shown = Date.UTC(p.year, p.month - 1, p.day, p.hour, p.minute)
  return new Date(guess - (shown - guess))
}

// Find the next Friday session that hasn't finished yet
function getNextSession(current) {
  const p = londonParts(current)
  for (let i = 0; i < 8; i++) {
    const base = new Date(Date.UTC(p.year, p.month - 1, p.day + i))
    if (base.getUTCDay() !== 5) continue // 5 = Friday
    const y = base.getUTCFullYear()
    const m = base.getUTCMonth() + 1
    const d = base.getUTCDate()
    const start = londonToDate(y, m, d, 13, 30)
    const end = londonToDate(y, m, d, 14, 0)
    if (end > current) return { start, end }
  }
  return null
}

const session = computed(() => getNextSession(now.value))

const inProgress = computed(() =>
  session.value && now.value >= session.value.start && now.value < session.value.end
)

const countdown = computed(() => {
  if (!session.value) return { days: 0, hours: 0, minutes: 0, seconds: 0 }
  const total = Math.max(0, Math.floor((session.value.start - now.value) / 1000))
  return {
    days: Math.floor(total / 86400),
    hours: Math.floor((total % 86400) / 3600),
    minutes: Math.floor((total % 3600) / 60),
    seconds: total % 60,
  }
})

const pad = n => String(n).padStart(2, '0')

const sessionDate = computed(() =>
  session.value
    ? session.value.start.toLocaleString('en-GB', { weekday: 'long', day: 'numeric', month: 'long', timeZone: TZ })
    : ''
)
</script>

<template>
  <div>
    <img class="hc-flag" src="/hc-flag.svg" alt="Hack Club flag" />

    <header class="hero">
      <div class="container">
        <span class="eyebrow">Hack Club</span>
        <h1>Huish Episcopi <span>Hack Club</span></h1>
        <p>Every Friday lunchtime in IT1. Bring a laptop, an idea, and your lunch.</p>
        <a class="btn btn-primary" href="#join">Join us</a>
        <div class="join-code-badge" id="join">
          Join code: <code>{{ joinCode }}</code>
        </div>
      </div>
    </header>

    <section class="section">
      <div class="container">
        <h2 class="section-title">Next session</h2>
        <p class="section-subtitle" v-if="session">
          {{ sessionDate }}, 13:30 to 14:00 in IT1
        </p>

        <div v-if="inProgress" class="card">
          <span class="card-tag tag-green">Live now</span>
          <h3>We're in session!</h3>
          <p>Come along to IT1 if you're not already here.</p>
        </div>

        <div v-else class="grid">
          <div class="card"><h3>{{ pad(countdown.days) }}</h3><p>Days</p></div>
          <div class="card"><h3>{{ pad(countdown.hours) }}</h3><p>Hours</p></div>
          <div class="card"><h3>{{ pad(countdown.minutes) }}</h3><p>Minutes</p></div>
          <div class="card"><h3>{{ pad(countdown.seconds) }}</h3><p>Seconds</p></div>
        </div>
      </div>
    </section>

    <section class="section">
      <div class="container">
        <h2 class="section-title">What we do</h2>
        <p class="section-subtitle">Build things, learn together, and ship projects in a relaxed lunchtime setting.</p>
        <div class="grid">
          <div class="card">
            <span class="card-tag tag-red">Build</span>
            <h3>Make projects</h3>
            <p>Work on websites, games, tools, or whatever you want to ship.</p>
          </div>
          <div class="card">
            <span class="card-tag tag-green">Learn</span>
            <h3>Help each other</h3>
            <p>Pick up new languages and tools with people who are happy to help.</p>
          </div>
          <div class="card">
            <span class="card-tag tag-blue">Share</span>
            <h3>Show off</h3>
            <p>Demo what you've made and get feedback from the group.</p>
          </div>
        </div>
      </div>
    </section>

    <footer>
      <div class="container">
        Huish Episcopi Academy Hack Club · <a href="https://hackclub.com" target="_blank">hackclub.com</a>
      </div>
    </footer>
  </div>
</template>
