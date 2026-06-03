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
      :dangerMsg="dangerMsg"
    />
    <GameBoard
      :player="player"
      :hazards="hazards"
      :bullets="bullets"
      :gems="gems"
    />
    <LevelUp
      v-if="isLevelUp"
      :difficulty="difficulty"
      :upgrades="currentUpgrades"
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

const score = ref(0)
const health = ref(100)
const timer = ref(0)
const difficulty = ref(1)
const isGameOver = ref(false)
const isLevelUp = ref(false)
const xp = ref(0)
const xpNext = ref(10)
const hazards = ref([])
const bullets = ref([])
const gems = ref([])
const player = ref({ x: 380, y: 230 })
const playerSpeed = ref(3)
const playerDefense = ref(1)
const dangerMsg = ref('')
const currentUpgrades = ref([])

let animFrame = null
let lastSpawn = 0
let lastTimer = 0
let lastDamage = 0
let lastAttack = 0
let hazardId = 0
let bulletId = 0
let gemId = 0
const keys = {}

let lastPlayerX = 380
let lastPlayerY = 230
let stillTime = 0
const STILL_THRESHOLD = 0.5
const FAST_SPEED = 2.8

// --- All possible upgrades ---
const ALL_UPGRADES = [
  { id: 'speed',   icon: '👟', name: 'SPEED +1',   desc: 'Move faster to dodge enemies' },
  { id: 'defense', icon: '🛡️', name: 'ARMOR +2',   desc: 'Take less damage per hit' },
  { id: 'health',  icon: '❤️', name: 'HEAL +20',   desc: 'Restore 20 HP right now' },
  { id: 'attack',  icon: '⚡', name: 'RAPID FIRE', desc: 'Shoot 30% faster' },
  { id: 'magnet',  icon: '🧲', name: 'MAGNET',     desc: 'Gems attract from further away' },
  { id: 'multishot', icon: '🔱', name: 'MULTISHOT', desc: 'Fire 2 extra bullets' },
]

let attackInterval = 900
let gemPickupRange = 40
let bulletCount = 1

function getRandomUpgrades() {
  const shuffled = [...ALL_UPGRADES].sort(() => Math.random() - 0.5)
  return shuffled.slice(0, 3)
}

function spawnHazard() {
  const side = Math.floor(Math.random() * 4)
  let x, y
  if (side === 0) { x = Math.random() * 800; y = -20 }
  else if (side === 1) { x = Math.random() * 800; y = 520 }
  else if (side === 2) { x = -20; y = Math.random() * 500 }
  else { x = 820; y = Math.random() * 500 }
  hazards.value.push({
    id: hazardId++, x, y,
    speed: 0.8 + difficulty.value * 0.15,
    fast: false
  })
}

function moveEnemies() {
  hazards.value.forEach((hazard) => {
    const dx = player.value.x - hazard.x
    const dy = player.value.y - hazard.y
    const dist = Math.sqrt(dx * dx + dy * dy)
    if (dist > 0) {
      const spd = hazard.fast ? FAST_SPEED : hazard.speed
      hazard.x += (dx / dist) * spd
      hazard.y += (dy / dist) * spd
    }
  })
}

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

function movePlayer() {
  const speed = playerSpeed.value
  if (keys['ArrowUp']    || keys['w']) player.value.y -= speed
  if (keys['ArrowDown']  || keys['s']) player.value.y += speed
  if (keys['ArrowLeft']  || keys['a']) player.value.x -= speed
  if (keys['ArrowRight'] || keys['d']) player.value.x += speed
  player.value.x = Math.max(0, Math.min(player.value.x, 780))
  player.value.y = Math.max(0, Math.min(player.value.y, 480))
}

function checkStill(dt) {
  const moved = Math.sqrt(
    Math.pow(player.value.x - lastPlayerX, 2) +
    Math.pow(player.value.y - lastPlayerY, 2)
  )
  if (moved < STILL_THRESHOLD) {
    stillTime += dt
    if (stillTime > 1200) {
      dangerMsg.value = '⚠ KEEP MOVING!'
      hazards.value.forEach(h => h.fast = true)
    } else if (stillTime > 600) {
      dangerMsg.value = "⚡ They're speeding up!"
    }
  } else {
    stillTime = 0
    dangerMsg.value = ''
    hazards.value.forEach(h => h.fast = false)
  }
  lastPlayerX = player.value.x
  lastPlayerY = player.value.y
}

function autoAttack(now) {
  if (now - lastAttack < attackInterval) return
  if (hazards.value.length === 0) return

  let nearest = null
  let nearestDist = Infinity
  hazards.value.forEach(h => {
    const dx = h.x - player.value.x
    const dy = h.y - player.value.y
    const d = Math.sqrt(dx * dx + dy * dy)
    if (d < nearestDist) { nearestDist = d; nearest = h }
  })
  if (!nearest) return

  const dx = nearest.x - player.value.x
  const dy = nearest.y - player.value.y
  const dist = Math.sqrt(dx * dx + dy * dy)

  // Fire main bullet + extras if multishot
  for (let i = 0; i < bulletCount; i++) {
    const angleOffset = (i - Math.floor(bulletCount / 2)) * 0.25
    const cos = Math.cos(angleOffset)
    const sin = Math.sin(angleOffset)
    const vx = ((dx / dist) * cos - (dy / dist) * sin) * 5
    const vy = ((dx / dist) * sin + (dy / dist) * cos) * 5
    bullets.value.push({ id: bulletId++, x: player.value.x, y: player.value.y, vx, vy, life: 90 })
  }
  lastAttack = now
}

function updateBullets() {
  bullets.value = bullets.value.filter(b => {
    b.x += b.vx
    b.y += b.vy
    b.life--
    if (b.x < 0 || b.x > 800 || b.y < 0 || b.y > 500 || b.life <= 0) return false
    for (let i = hazards.value.length - 1; i >= 0; i--) {
      const h = hazards.value[i]
      if (Math.sqrt((b.x - h.x) ** 2 + (b.y - h.y) ** 2) < 20) {
        // Drop XP gem where enemy died
        gems.value.push({ id: gemId++, x: h.x, y: h.y })
        hazards.value.splice(i, 1)
        score.value += 10
        return false
      }
    }
    return true
  })
}

function updateGems() {
  gems.value = gems.value.filter(gem => {
    const dx = player.value.x - gem.x
    const dy = player.value.y - gem.y
    const dist = Math.sqrt(dx * dx + dy * dy)

    // Move gem toward player when in range
    if (dist < gemPickupRange * 3) {
      gem.x += (dx / dist) * 3
      gem.y += (dy / dist) * 3
    }

    // Collect gem
    if (dist < gemPickupRange) {
      gainXp(1)
      return false
    }
    return true
  })
}

function gainXp(amount) {
  xp.value += amount
  if (xp.value >= xpNext.value) {
    xp.value = 0
    xpNext.value = Math.floor(xpNext.value * 1.4)
    currentUpgrades.value = getRandomUpgrades()
    isLevelUp.value = true
  }
}

let lastFrameTime = 0
function gameLoop(now) {
  if (isGameOver.value) return
  if (isLevelUp.value) {
    animFrame = requestAnimationFrame(gameLoop)
    return
  }

  const dt = Math.min(now - lastFrameTime, 50)
  lastFrameTime = now

  if (now - lastTimer > 1000) {
    timer.value++
    score.value++
    gainXp(1)
    lastTimer = now
  }

  const spawnInterval = Math.max(500, 1800 - difficulty.value * 150)
  if (now - lastSpawn > spawnInterval) {
    spawnHazard()
    lastSpawn = now
  }

  movePlayer()
  checkStill(dt)
  moveEnemies()
  checkCollision(now)
  autoAttack(now)
  updateBullets()
  updateGems()

  difficulty.value = 1 + Math.floor(timer.value / 15)

  if (health.value <= 0) {
    isGameOver.value = true
    return
  }

  animFrame = requestAnimationFrame(gameLoop)
}

function applyUpgrade(type) {
  if (type === 'speed')     playerSpeed.value += 1
  if (type === 'defense')   playerDefense.value += 2
  if (type === 'health')    health.value = Math.min(100, health.value + 20)
  if (type === 'attack')    attackInterval = Math.max(300, attackInterval - 270)
  if (type === 'magnet')    gemPickupRange += 30
  if (type === 'multishot') bulletCount += 2
  isLevelUp.value = false
  lastFrameTime = performance.now()
  animFrame = requestAnimationFrame(gameLoop)
}

function restartGame() {
  cancelAnimationFrame(animFrame)
  score.value = 0; timer.value = 0; health.value = 100
  difficulty.value = 1; xp.value = 0; xpNext.value = 10
  playerSpeed.value = 3; playerDefense.value = 1
  hazards.value = []; bullets.value = []; gems.value = []
  player.value = { x: 380, y: 230 }
  isGameOver.value = false; isLevelUp.value = false
  lastSpawn = 0; lastTimer = 0; lastDamage = 0
  lastAttack = 0; lastPlayerX = 380; lastPlayerY = 230
  stillTime = 0; dangerMsg.value = ''
  attackInterval = 900; gemPickupRange = 40; bulletCount = 1
  lastFrameTime = performance.now()
  animFrame = requestAnimationFrame(gameLoop)
}

function onKeyDown(e) { keys[e.key] = true }
function onKeyUp(e) { keys[e.key] = false }

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
* { box-sizing: border-box; margin: 0; padding: 0; }
body { background: #0a0a0f; font-family: 'Press Start 2P', monospace; }
.app { min-height: 100vh; background: #0a0a0f; color: #e8d5a3; padding: 20px; text-align: center; }
.game-title { font-size: 18px; color: #e8d5a3; text-shadow: 0 0 10px #ff4444, 3px 3px 0 #8b0000; margin-bottom: 16px; letter-spacing: 3px; }
</style>