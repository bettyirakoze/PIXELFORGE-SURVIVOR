<template>
  <div class="app">
    <h1 class="game-title">☠ Pixel Survivor ☠</h1>

    <GameHUD
      :score="score"
      :health="health"
      :timer="timer"
      :difficulty="difficulty"
      :xp="xp"
      :xpNext="xpNext"
    />

    <GameBoard
      :player="player"
      :hazards="hazards"
    />

    <LevelUp
      v-if="isLevelUp"
      :difficulty="difficulty"
      @upgrade="applyUpgrade"
    />

    <GameOver
      v-if="isGameOver"
      :score="score"
      @restart="restartGame"
    />
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import GameHUD from './components/GameHUD.vue'
import GameBoard from './components/GameBoard.vue'
import GameOver from './components/GameOver.vue'
import LevelUp from './components/LevelUp.vue'

// --- State ---
const score = ref(0)
const health = ref(100)
const timer = ref(0)
const difficulty = ref(1)
const isGameOver = ref(false)
const isLevelUp = ref(false)
const xp = ref(0)
const xpNext = ref(10)
const hazards = ref([])
const player = ref({ x: 380, y: 230 })
const playerSpeed = ref(3)
const playerDefense = ref(1)

// --- Internals ---
let animFrame = null
let lastSpawn = 0
let lastTimer = 0
let lastDamage = 0
let hazardId = 0
const keys = {}

// --- Spawn enemies from edges ---
function spawnHazard() {
  const side = Math.floor(Math.random() * 4)
  let x, y
  if (side === 0) { x = Math.random() * 800; y = -20 }
  else if (side === 1) { x = Math.random() * 800; y = 520 }
  else if (side === 2) { x = -20; y = Math.random() * 500 }
  else { x = 820; y = Math.random() * 500 }

  hazards.value.push({
    id: hazardId++,
    x,
    y,
    speed: 0.8 + difficulty.value * 0.15
  })
}

// --- Move enemies toward player ---
function moveEnemies() {
  hazards.value.forEach((hazard) => {
    const dx = player.value.x - hazard.x
    const dy = player.value.y - hazard.y
    const dist = Math.sqrt(dx * dx + dy * dy)
    if (dist > 0) {
      hazard.x += (dx / dist) * hazard.speed
      hazard.y += (dy / dist) * hazard.speed
    }
  })
}

// --- Check enemy touching player ---
function checkCollision(now) {
  if (now - lastDamage < 800) return
  hazards.value.forEach((hazard) => {
    const dx = player.value.x - hazard.x
    const dy = player.value.y - hazard.y
    if (Math.sqrt(dx * dx + dy * dy) < 36) {
      const dmg = Math.max(1, 8 - playerDefense.value)
      health.value = Math.max(0, health.value - dmg)
      lastDamage = now
    }
  })
}

// --- Move player from held keys ---
function movePlayer() {
  const speed = playerSpeed.value
  if (keys['ArrowUp']    || keys['w']) player.value.y -= speed
  if (keys['ArrowDown']  || keys['s']) player.value.y += speed
  if (keys['ArrowLeft']  || keys['a']) player.value.x -= speed
  if (keys['ArrowRight'] || keys['d']) player.value.x += speed
  player.value.x = Math.max(0, Math.min(player.value.x, 780))
  player.value.y = Math.max(0, Math.min(player.value.y, 480))
}

// --- Gain XP and trigger level up ---
function gainXp(amount) {
  xp.value += amount
  if (xp.value >= xpNext.value) {
    xp.value = 0
    xpNext.value = Math.floor(xpNext.value * 1.4)
    isLevelUp.value = true
  }
}

// --- Main game loop (60fps) ---
function gameLoop(now) {
  if (isGameOver.value) return

  if (isLevelUp.value) {
    animFrame = requestAnimationFrame(gameLoop)
    return
  }

  // Tick timer every second
  if (now - lastTimer > 1000) {
    timer.value++
    score.value++
    gainXp(1)
    lastTimer = now
  }

  // Spawn rate gets faster with difficulty
  const spawnInterval = Math.max(500, 1800 - difficulty.value * 150)
  if (now - lastSpawn > spawnInterval) {
    spawnHazard()
    lastSpawn = now
  }

  movePlayer()
  moveEnemies()
  checkCollision(now)

  // Increase difficulty every 15 seconds
  difficulty.value = 1 + Math.floor(timer.value / 15)

  if (health.value <= 0) {
    isGameOver.value = true
    return
  }

  animFrame = requestAnimationFrame(gameLoop)
}

// --- Apply chosen upgrade ---
function applyUpgrade(type) {
  if (type === 'speed') playerSpeed.value += 1
  if (type === 'defense') playerDefense.value += 2
  isLevelUp.value = false
  animFrame = requestAnimationFrame(gameLoop)
}

// --- Restart everything ---
function restartGame() {
  cancelAnimationFrame(animFrame)

  score.value = 0
  timer.value = 0
  health.value = 100
  difficulty.value = 1
  xp.value = 0
  xpNext.value = 10
  playerSpeed.value = 3
  playerDefense.value = 1
  hazards.value = []
  player.value = { x: 380, y: 230 }
  isGameOver.value = false
  isLevelUp.value = false
  lastSpawn = 0
  lastTimer = 0
  lastDamage = 0

  animFrame = requestAnimationFrame(gameLoop)
}

// --- Keyboard input ---
function onKeyDown(e) {
  keys[e.key] = true
  if (e.key === 'x') gainXp(3) // test XP gain
}
function onKeyUp(e) {
  keys[e.key] = false
}

onMounted(() => {
  window.addEventListener('keydown', onKeyDown)
  window.addEventListener('keyup', onKeyUp)
  animFrame = requestAnimationFrame(gameLoop)
})

onUnmounted(() => {
  cancelAnimationFrame(animFrame)
  window.removeEventListener('keydown', onKeyDown)
  window.removeEventListener('keyup', onKeyUp)
})
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  background: #0a0a0f;
  font-family: 'Press Start 2P', monospace;
}

.app {
  min-height: 100vh;
  background: #0a0a0f;
  color: #e8d5a3;
  padding: 20px;
  text-align: center;
}

.game-title {
  font-size: 18px;
  color: #e8d5a3;
  text-shadow: 0 0 10px #ff4444, 3px 3px 0 #8b0000;
  margin-bottom: 16px;
  letter-spacing: 3px;
}
</style>