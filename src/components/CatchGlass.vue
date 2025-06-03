<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
const basketOpen = '/mouth.png'
const basketClosed = '/mouth-closed.png'
const basketImg = ref(basketOpen)
const glassImages = [
  '/glass.png',
  '/glass2.png',
  '/glass3.png'
]
const glasses = ref<{ id: number; x: number; y: number; img: string }[]>([])
const score = ref(0)
const timer = ref(25)
const playerName = ref('')
const gameState = ref<'start' | 'playing' | 'end'>('start')
let glassId = 0
let gameInterval: number
let timerInterval: number
let mouthTimeout: number | undefined

const basketX = ref(180)
const basketWidth = 60
const basketY = 350

function spawnGlass() {
  const randomGlass = glassImages[Math.floor(Math.random() * glassImages.length)]
  glasses.value.push({
    id: glassId++,
    x: Math.random() * 320 + 20,
    y: 0,
    img: randomGlass,
  })
}

function moveGlasses() {
  glasses.value.forEach(g => (g.y += 5))
  let caught = false
  glasses.value.forEach(g => {
    if (
      g.y + 40 >= basketY &&
      g.x + 40 > basketX.value &&
      g.x < basketX.value + basketWidth
    ) {
      score.value++
      g.y = 1000
      caught = true
    }
  })
  if (caught) {
    basketImg.value = basketClosed
    clearTimeout(mouthTimeout)
    mouthTimeout = window.setTimeout(() => {
      basketImg.value = basketOpen
    }, 200)
  }
  glasses.value = glasses.value.filter(g => g.y < 400)
}

function handleKeydown(e: KeyboardEvent) {
  if (gameState.value !== 'playing') return
  if (e.key === 'ArrowLeft') basketX.value = Math.max(0, basketX.value - 20)
  if (e.key === 'ArrowRight') basketX.value = Math.min(400 - basketWidth, basketX.value + 20)
}

function startGame() {
  glasses.value = []
  score.value = 0
  timer.value = 25
  glassId = 0
  basketX.value = 180
  gameState.value = 'playing'
  basketImg.value = basketOpen

  gameInterval = setInterval(() => {
    if (Math.random() < 0.2) spawnGlass()
    moveGlasses()
  }, 50)
  timerInterval = setInterval(() => {
    timer.value--
    if (timer.value <= 0) {
      clearInterval(gameInterval)
      clearInterval(timerInterval)
      gameState.value = 'end'
    }
  }, 1000)
}

function tryAgain() {
  gameState.value = 'start'
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
  clearInterval(gameInterval)
  clearInterval(timerInterval)
  clearTimeout(mouthTimeout)
})
</script>

<template>
  <div class="game-area">
    <template v-if="gameState === 'start'">
      <div class="overlay">
        <input
          v-model="playerName"
          placeholder="Skriv ditt namn här"
          class="name-input"
          @keyup.enter="playerName && startGame()"
        />
        <button :disabled="!playerName" @click="startGame">Börja spela</button>
      </div>
    </template>
    <template v-else-if="gameState === 'playing'">
      <div class="timer">Time: {{ timer }}</div>
      <img
        v-for="glass in glasses"
        :key="glass.id"
        :src="glass.img"
        class="glass"
        :style="{ left: glass.x + 'px', top: glass.y + 'px' }"
        alt="glass"
      />
      <img
        :src="basketImg"
        class="basket"
        :style="{ left: basketX + 'px', bottom: '10px' }"
        alt="basket"
      />
      <div class="score">Score: {{ score }}</div>
    </template>
    <template v-else-if="gameState === 'end'">
      <div class="overlay">
        <h2>Tiden är slut!</h2>
        <p>Du åt: <strong>{{ score }}</strong> glassar</p>
        <p>Dela ditt resultat för en chans att vinna en glass.</p>
        <button @click="tryAgain">Försök igen</button>
      </div>
    </template>
  </div>
</template>
<style scoped>
.game-area {
  position: relative;
  width: 400px;
  height: 400px;
  background: white;
  overflow: hidden;
  border-radius: 16px;
  border: 2px solid #333;
  margin: 0 auto;
}
.glass {
  position: absolute;
  width: 40px;
  transition: top 0.05s linear;
  user-select: none;
  pointer-events: none;
  z-index: 3; /* higher than basket */
}
.basket {
  position: absolute;
  bottom: 10px;
  width: 80px;
  height: 80px;
  z-index: 2; /* lower than glass */
  transition: left 0.1s;
}
.score {
  position: absolute;
  top: 10px;
  left: 10px;
  font-size: 1.5em; 
  color: #333;
  font-weight: bold;
}
.timer {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 1.5em;
  color: #333;
  font-weight: bold;
}
.overlay {
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  background-image: url('/overlay.png');
  background-color: white;
  font-size: larger;
  background-size: cover;
  background-position: center;
  display: flex;
  color: Yellow;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 10;
  text-shadow: 2px 2px 8px #000, 0 0 2px #000; 
}
.name-input {
  font-size: 1.2em;
  padding: 0.5em;
  margin: 1em 0;
  border-radius: 8px;
  border: 1px solid #aaa;
}
button {
  font-size: 1.2em;
  padding: 0.5em 1.5em;
  border-radius: 8px;
  border: none;
  background: #2ccc44;
  color: #fff;
  cursor: pointer;
  margin-top: 1em;
  transition: background 0.2s;
}
button:disabled {
  background: #aaa;
  cursor: not-allowed;
}
button:not(:disabled):hover {
  background: yellowgreen;
}
</style>