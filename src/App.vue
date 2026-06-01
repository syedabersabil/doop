<script setup>
import { ref, watch, computed, onBeforeUnmount } from 'vue'

// Theme config
const theme = ref(localStorage.getItem('theme') || 'dark')

watch(theme, (newTheme) => {
  document.documentElement.setAttribute('data-theme', newTheme)
  localStorage.setItem('theme', newTheme)
}, { immediate: true })

const toggleTheme = () => {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
}

// ----------------------------------------------------
// 1. BREATHING CYCLE CONTROLLER
// ----------------------------------------------------
const isBreathing = ref(false)
const breathPhase = ref('exhale') // inhale, hold, exhale
const breathMessage = ref('Ready to begin')
let breathInterval = null

const startBreathing = () => {
  isBreathing.value = true
  runBreathingCycle()
}

const runBreathingCycle = () => {
  breathPhase.value = 'inhale'
  breathMessage.value = 'Breathe In...'
  
  let cycle = 0
  breathInterval = setInterval(() => {
    cycle = (cycle + 1) % 3
    if (cycle === 0) {
      breathPhase.value = 'inhale'
      breathMessage.value = 'Breathe In...'
    } else if (cycle === 1) {
      breathPhase.value = 'hold'
      breathMessage.value = 'Hold and Feel Calm...'
    } else if (cycle === 2) {
      breathPhase.value = 'exhale'
      breathMessage.value = 'Breathe Out...'
    }
  }, 4000)
}

const stopBreathing = () => {
  isBreathing.value = false
  breathPhase.value = 'exhale'
  breathMessage.value = 'Ready to begin'
  if (breathInterval) {
    clearInterval(breathInterval)
    breathInterval = null
  }
}

// ----------------------------------------------------
// 2. HABIT TRACKER LIST
// ----------------------------------------------------
const habits = ref([
  { id: 1, name: 'Drink 2.5 Liters of Water', completed: false, streak: 5 },
  { id: 2, name: '10-Minute Morning Meditation', completed: false, streak: 12 },
  { id: 3, name: 'Write 3 Things in Gratitude Journal', completed: false, streak: 3 },
  { id: 4, name: 'Unplug from Screens (1 hour before bed)', completed: false, streak: 8 },
  { id: 5, name: '30-Minute Physical Movement', completed: false, streak: 2 }
])

const toggleHabit = (id) => {
  const habit = habits.value.find(h => h.id === id)
  if (habit) {
    habit.completed = !habit.completed
    if (habit.completed) {
      habit.streak++
    } else {
      habit.streak = Math.max(0, habit.streak - 1)
    }
  }
}

const completionPercentage = computed(() => {
  const completed = habits.value.filter(h => h.completed).length
  return Math.round((completed / habits.value.length) * 100)
})

// ----------------------------------------------------
// 3. GRATITUDE JOURNAL
// ----------------------------------------------------
const journalText = ref('')
const journalEntries = ref([
  { id: 1, date: 'June 01, 2026', text: 'Grateful for the peace and quiet of the early morning hours.' },
  { id: 2, date: 'May 31, 2026', text: 'Appreciating a warm cup of herbal tea and the supportive friends in my life.' }
])

const addJournalEntry = () => {
  if (!journalText.value.trim()) return
  const now = new Date()
  const options = { month: 'long', day: '2-digit', year: 'numeric' }
  
  journalEntries.value.unshift({
    id: Date.now(),
    date: now.toLocaleDateString('en-US', options),
    text: journalText.value
  })
  journalText.value = ''
}

// ----------------------------------------------------
// 4. RANDOM AFFIRMATION / QUOTE GENERATOR
// ----------------------------------------------------
const quotes = [
  { text: "Breathe in deeply to bring your mind home to your body.", author: "Thich Nhat Hanh" },
  { text: "You don't have to control your thoughts. You just have to stop letting them control you.", author: "Dan Millman" },
  { text: "Within you, there is a stillness and a sanctuary to which you can retreat at any time.", author: "Hermann Hesse" },
  { text: "The present moment is filled with joy and happiness. If you are attentive, you will see it.", author: "Thich Nhat Hanh" },
  { text: "Quiet the mind and the soul will speak.", author: "Ma Jaya Sati Bhagavati" }
]
const currentQuoteIndex = ref(0)
const rotateQuote = () => {
  currentQuoteIndex.value = (currentQuoteIndex.value + 1) % quotes.length
}

// ----------------------------------------------------
// 5. CALMING BINAURAL / AMBIENT SYNTHESIZER
// ----------------------------------------------------
let audioCtx = null
let oscillators = []
let noiseNode = null
const currentPlayingSound = ref('') // 'bowl', 'waves', ''

const initAudioContext = () => {
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)()
  }
}

const toggleSound = (soundType) => {
  initAudioContext()
  
  if (currentPlayingSound.value === soundType) {
    stopAllAudio()
    return
  }
  
  stopAllAudio()
  currentPlayingSound.value = soundType
  
  if (soundType === 'bowl') {
    playSingingBowl()
  } else if (soundType === 'waves') {
    playOceanWaves()
  }
}

const playSingingBowl = () => {
  if (!audioCtx) return
  
  // Create master volume node
  const gainNode = audioCtx.createGain()
  gainNode.gain.setValueAtTime(0.001, audioCtx.currentTime)
  
  // Low resonant frequency with slight detune for binaural warmth
  const osc1 = audioCtx.createOscillator()
  osc1.type = 'sine'
  osc1.frequency.setValueAtTime(144, audioCtx.currentTime)
  
  const osc2 = audioCtx.createOscillator()
  osc2.type = 'sine'
  osc2.frequency.setValueAtTime(144.5, audioCtx.currentTime)
  
  osc1.connect(gainNode)
  osc2.connect(gainNode)
  gainNode.connect(audioCtx.destination)
  
  osc1.start()
  osc2.start()
  
  // Fade in
  gainNode.gain.exponentialRampToValueAtTime(0.2, audioCtx.currentTime + 3)
  
  oscillators.push(osc1, osc2)
}

const playOceanWaves = () => {
  if (!audioCtx) return

  const bufferSize = audioCtx.sampleRate * 2
  const noiseBuffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate)
  const output = noiseBuffer.getChannelData(0)
  
  // Generate Pink noise
  let b0, b1, b2, b3, b4, b5, b6
  b0 = b1 = b2 = b3 = b4 = b5 = b6 = 0.0
  for (let i = 0; i < bufferSize; i++) {
    let white = Math.random() * 2 - 1
    b0 = 0.99886 * b0 + white * 0.0555179
    b1 = 0.99332 * b1 + white * 0.0750759
    b2 = 0.96900 * b2 + white * 0.1538520
    b3 = 0.86650 * b3 + white * 0.3104856
    b4 = 0.55000 * b4 + white * 0.5329522
    b5 = -0.7616 * b5 - white * 0.0168980
    output[i] = b0 + b1 + b2 + b3 + b4 + b5 + b6 + white * 0.5362
    output[i] *= 0.11 // rough volume compensation
    b6 = white * 0.115926
  }

  // Create noise source
  const noise = audioCtx.createBufferSource()
  noise.buffer = noiseBuffer
  noise.loop = true

  // Filter to create wave dynamics
  const filter = audioCtx.createBiquadFilter()
  filter.type = 'lowpass'
  filter.Q.setValueAtTime(1, audioCtx.currentTime)

  // Modulate filter cutoff (LFO) to simulate ebb and flow of waves
  const lfo = audioCtx.createOscillator()
  lfo.type = 'sine'
  lfo.frequency.setValueAtTime(0.12, audioCtx.currentTime) // 8 second cycle

  const lfoGain = audioCtx.createGain()
  lfoGain.gain.setValueAtTime(400, audioCtx.currentTime) // modulation depth

  // Wave Master Volume
  const volume = audioCtx.createGain()
  volume.gain.setValueAtTime(0.001, audioCtx.currentTime)

  // Connections
  lfo.connect(lfoGain)
  lfoGain.connect(filter.frequency)
  noise.connect(filter)
  filter.connect(volume)
  volume.connect(audioCtx.destination)

  // Start nodes
  lfo.start()
  noise.start()
  
  // Cutoff default
  filter.frequency.setValueAtTime(600, audioCtx.currentTime)
  
  // Fade in
  volume.gain.exponentialRampToValueAtTime(0.15, audioCtx.currentTime + 3)

  oscillators.push(lfo, noise)
}

const stopAllAudio = () => {
  oscillators.forEach(osc => {
    try {
      osc.stop()
    } catch (e) {}
  })
  oscillators = []
  currentPlayingSound.value = ''
}

onBeforeUnmount(() => {
  stopAllAudio()
  stopBreathing()
  if (audioCtx) {
    audioCtx.close()
  }
})
</script>

<template>
  <div class="app-container">
    <div class="zen-gradient-bg"></div>
    
    <!-- Navbar Navigation -->
    <nav class="navbar">
      <div class="nav-logo">
        <span>ZenSpace</span>
      </div>
      <ul class="nav-links">
        <li><a href="#breathing" class="nav-link">Breathing</a></li>
        <li><a href="#habits" class="nav-link">Habits</a></li>
        <li><a href="#journal" class="nav-link">Journal</a></li>
        <li><a href="#ambient" class="nav-link">Ambient</a></li>
      </ul>
      <div class="nav-actions">
        <button 
          @click="toggleTheme" 
          class="theme-toggle-btn"
          aria-label="Toggle zen layout theme"
          title="Toggle Zen Layout Theme"
        >
          <template v-if="theme === 'light'">
            <svg width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M21.752 15.002A9.718 9.718 0 0118 15.75c-5.385 0-9.75-4.365-9.75-9.75 0-1.33.266-2.597.748-3.752A9.753 9.753 0 003 11.25C3 16.635 7.365 21 12.75 21a9.753 9.753 0 009.002-5.998z"></path>
            </svg>
          </template>
          <template v-else>
            <svg width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M12 3v2.25m0 13.5V21M4.978 4.978l1.591 1.591m10.862 10.862l1.591 1.591M3 12h2.25m13.5 0H21M4.978 19.022l1.591-1.591m10.862-10.862l1.591-1.591M12 7.5a4.5 4.5 0 110 9 4.5 4.5 0 010-9z"></path>
            </svg>
          </template>
        </button>
      </div>
    </nav>

    <!-- Main Container Content -->
    <main class="main-content">
      
      <!-- Calming Header Intro -->
      <section class="zen-hero">
        <h1 class="zen-hero-title">Your sanctuary for <span>mental clarity</span></h1>
        <p class="zen-hero-subtitle">
          Pause, center yourself, and build intentional wellness habits. Quiet your mind with our local ambient loops and mindfulness tools.
        </p>
      </section>

      <!-- Dashboard Grid Layout -->
      <div class="zen-grid">
        
        <!-- Breathing Sphere (Col 6) -->
        <section class="zen-card col-6" id="breathing">
          <h2 class="card-title">💨 Guided Breathing</h2>
          <p class="card-subtitle">Pace your breath with the expanding circles to lower stress levels.</p>
          
          <div class="breath-widget">
            <div class="breath-circle-container">
              <div :class="['breath-circle-glow', breathPhase]"></div>
              <div :class="['breath-circle', breathPhase]">
                <span>{{ breathPhase.toUpperCase() }}</span>
              </div>
            </div>
            
            <p class="breath-instruction">{{ breathMessage }}</p>
            
            <div class="breath-controls">
              <button v-if="!isBreathing" @click="startBreathing" class="btn-zen">Begin Practice</button>
              <button v-else @click="stopBreathing" class="btn-zen-outline">Stop Practice</button>
            </div>
          </div>
        </section>

        <!-- Daily Habit Tracker (Col 6) -->
        <section class="zen-card col-6" id="habits">
          <h2 class="card-title">🌿 Daily Intentions</h2>
          <p class="card-subtitle">Mark off your positive habits and watch your streaks grow.</p>
          
          <div class="habit-list">
            <article 
              v-for="habit in habits" 
              :key="habit.id" 
              :class="['habit-item', { completed: habit.completed }]"
              @click="toggleHabit(habit.id)"
            >
              <div class="habit-left">
                <div class="habit-checkbox">
                  <svg v-if="habit.completed" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3">
                    <polyline points="20 6 9 17 4 12"></polyline>
                  </svg>
                </div>
                <span class="habit-name">{{ habit.name }}</span>
              </div>
              <span class="habit-streak">🔥 {{ habit.streak }}d</span>
            </article>
          </div>

          <div class="progress-header">
            <span>Overall Completion</span>
            <span>{{ completionPercentage }}%</span>
          </div>
          <div class="progress-bar-container">
            <div class="progress-bar-fill" :style="{ width: `${completionPercentage}%` }"></div>
          </div>
        </section>

        <!-- Gratitude Journal (Col 8) -->
        <section class="zen-card col-8" id="journal">
          <h2 class="card-title">📝 Gratitude Canvas</h2>
          <p class="card-subtitle">Writing down things you are grateful for shifts focus onto positive emotions.</p>
          
          <form @submit.prevent="addJournalEntry" class="journal-form">
            <textarea 
              v-model="journalText" 
              class="journal-textarea" 
              placeholder="Today, I am grateful for..."
              required
            ></textarea>
            <button type="submit" class="btn-zen" style="align-self: flex-start;">Save Entry</button>
          </form>

          <div class="journal-entries-grid">
            <div v-for="entry in journalEntries" :key="entry.id" class="journal-card">
              <p class="journal-date">{{ entry.date }}</p>
              <p class="journal-text">{{ entry.text }}</p>
            </div>
          </div>
        </section>

        <!-- Quotes & Sounds (Col 4) -->
        <div class="col-4" style="display: flex; flex-direction: column; gap: 2rem;">
          
          <!-- Affirmation Quotes -->
          <section class="zen-card quotes-widget" style="flex: 1;">
            <h2 class="card-title">✨ Affirmation</h2>
            <p class="quote-text">"{{ quotes[currentQuoteIndex].text }}"</p>
            <div style="display: flex; justify-content: space-between; align-items: center; margin-top: 1rem;">
              <span class="quote-author">— {{ quotes[currentQuoteIndex].author }}</span>
              <button @click="rotateQuote" class="btn-zen-outline" style="padding: 0.4rem 0.8rem; font-size: 0.8rem;">Next</button>
            </div>
          </section>

          <!-- Ambient Sounds Synth -->
          <section class="zen-card sound-widget" id="ambient" style="flex: 1;">
            <h2 class="card-title">🎵 Calming Ambience</h2>
            <p class="card-subtitle">Synthesize local nature patterns using browser audio nodes.</p>
            
            <div class="sound-grid">
              <button 
                :class="['sound-card', { playing: currentPlayingSound === 'bowl' }]"
                @click="toggleSound('bowl')"
                style="background: transparent; cursor: pointer; border-radius: var(--border-radius-md); text-align: center; border: 1px solid var(--border-color);"
              >
                <span class="sound-emoji">🥣</span>
                <span class="sound-name">Singing Bowl</span>
              </button>
              
              <button 
                :class="['sound-card', { playing: currentPlayingSound === 'waves' }]"
                @click="toggleSound('waves')"
                style="background: transparent; cursor: pointer; border-radius: var(--border-radius-md); text-align: center; border: 1px solid var(--border-color);"
              >
                <span class="sound-emoji">🌊</span>
                <span class="sound-name">Ocean Waves</span>
              </button>
            </div>
          </section>

        </div>

      </div>
    </main>

    <!-- Zen Footer -->
    <footer class="zen-footer">
      <p class="footer-text">
        &copy; {{ new Date().getFullYear() }} ZenSpace Sanctuary. Built for calm mind & mindful habits.
      </p>
    </footer>
  </div>
</template>
