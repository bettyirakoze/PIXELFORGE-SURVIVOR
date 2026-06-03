```vue
<template>
  <div class="hud">
    <div class="hud-item">
      <div class="hud-label">SCORE</div>
      <div class="hud-value">{{ String(score).padStart(4, '0') }}</div>
    </div>

    <div class="hud-item hud-hp">
      <div class="hud-label">HP</div>
      <div class="hp-bar-bg">
        <div class="hp-bar" :style="{ width: health + '%' }"></div>
      </div>
      <div class="hp-text">{{ health }}/100</div>
    </div>

    <div class="hud-item hud-xp">
      <div class="hud-label">XP</div>
      <div class="xp-bar-bg">
        <div class="xp-bar" :style="{ width: (xp / xpNext * 100) + '%' }"></div>
      </div>
      <div class="xp-text">{{ xp }}/{{ xpNext }}</div>
    </div>

    <div class="hud-item">
      <div class="hud-label">TIME</div>
      <div class="hud-value">{{ formattedTime }}</div>
    </div>

    <div class="hud-item">
      <div class="hud-label">LEVEL</div>
      <div class="hud-value hud-level">LV.{{ difficulty }}</div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
const props = defineProps({ score: Number, health: Number, timer: Number, difficulty: Number, xp: Number, xpNext: Number })
const formattedTime = computed(() => {
  const m = String(Math.floor(props.timer / 60)).padStart(2, '0')
  const s = String(props.timer % 60).padStart(2, '0')
  return `${m}:${s}`
})
</script>

<style scoped lang="scss">
.hud {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 20px;
  background: rgba(0,0,0,0.9);
  border: 2px solid #8b0000;
  border-bottom: none;
  padding: 10px 20px;
  width: 800px;
  margin: 0 auto;
  font-family: 'Press Start 2P', monospace;
}
.hud-item { display: flex; flex-direction: column; align-items: center; gap: 4px; min-width: 70px; }
.hud-label { font-size: 6px; color: #666; letter-spacing: 1px; }
.hud-value { font-size: 10px; color: #e8d5a3; }
.hud-level { color: #f5c518; text-shadow: 0 0 6px #f5c518; }
.hud-hp { min-width: 140px; }
.hp-bar-bg { width: 120px; height: 8px; background: #1a0000; border: 1px solid #600; }
.hp-bar { height: 100%; background: linear-gradient(90deg, #cc0000, #ff3333); transition: width 0.2s; }
.hp-text { font-size: 6px; color: #ff6666; }
.hud-xp { min-width: 140px; }
.xp-bar-bg { width: 120px; height: 8px; background: #001a0a; border: 1px solid #0a6; }
.xp-bar { height: 100%; background: linear-gradient(90deg, #00aa44, #00ff88); transition: width 0.2s; }
.xp-text { font-size: 6px; color: #00ff88; }
</style>
