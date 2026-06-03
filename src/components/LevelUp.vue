<template>
  <div class="levelup-overlay">
    <div class="levelup-box">
      <div class="levelup-star">★</div>
      <h2 class="levelup-title">LEVEL UP!</h2>
      <p class="levelup-sub">CHOOSE AN UPGRADE</p>
      <div class="upgrade-options">
        <button
          v-for="upgrade in upgrades"
          :key="upgrade.id"
          class="upgrade-btn"
          @click="$emit('upgrade', upgrade.id)"
        >
          <span class="upgrade-icon">{{ upgrade.icon }}</span>
          <span class="upgrade-name">{{ upgrade.name }}</span>
          <span class="upgrade-desc">{{ upgrade.desc }}</span>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({ upgrades: Array })
defineEmits(['upgrade'])
</script>

<style scoped lang="scss">
.levelup-overlay {
  position: fixed; inset: 0;
  background: rgba(0,0,0,0.8);
  display: flex; align-items: center; justify-content: center;
  z-index: 200; font-family: 'Press Start 2P', monospace;
}
.levelup-box {
  background: #0a0a0f; border: 3px solid #f5c518;
  box-shadow: 0 0 30px rgba(245,197,24,0.4);
  padding: 32px 48px; text-align: center;
  display: flex; flex-direction: column; align-items: center; gap: 16px;
}
.levelup-star {
  font-size: 36px; color: #f5c518;
  text-shadow: 0 0 16px #f5c518;
  animation: starSpin 2s linear infinite;
}
@keyframes starSpin {
  from { transform: rotate(0deg); }
  to   { transform: rotate(360deg); }
}
.levelup-title { font-size: 22px; color: #f5c518; text-shadow: 2px 2px 0 #7a6000; letter-spacing: 4px; }
.levelup-sub { font-size: 8px; color: #888; letter-spacing: 2px; }
.upgrade-options { display: flex; gap: 16px; margin-top: 8px; }
.upgrade-btn {
  font-family: 'Press Start 2P', monospace;
  background: #0f0f1a; border: 2px solid #8b0000;
  color: #e8d5a3; padding: 16px; cursor: pointer;
  display: flex; flex-direction: column; align-items: center;
  gap: 8px; width: 140px; transition: all 0.1s;
  &:hover { border-color: #f5c518; background: #1a1500; box-shadow: 0 0 12px rgba(245,197,24,0.3); }
  &:active { transform: scale(0.97); }
}
.upgrade-icon { font-size: 28px; }
.upgrade-name { font-size: 8px; color: #f5c518; }
.upgrade-desc { font-size: 6px; color: #888; line-height: 1.6; }
</style>