<template>
  <div class="gameover-overlay">
    <div class="gameover-box">
      <div class="gameover-skull">☠</div>
      <h2 class="gameover-title">YOU DIED</h2>
      <p class="gameover-score">SCORE: {{ String(score).padStart(4, '0') }}</p>
      <button class="gameover-btn" @click="$emit('restart')">▶ PLAY AGAIN</button>
    </div>
  </div>
</template>

<script setup>
defineProps({ score: Number })
defineEmits(['restart'])
</script>

<style scoped lang="scss">
.gameover-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.85);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  font-family: 'Press Start 2P', monospace;
}

.gameover-box {
  background: #0a0a0f;
  border: 3px solid #8b0000;
  box-shadow: 0 0 30px rgba(139, 0, 0, 0.6);
  padding: 40px 60px;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.gameover-skull {
  font-size: 48px;
  animation: skullPulse 1s infinite alternate;
}

@keyframes skullPulse {
  from { text-shadow: 0 0 10px #ff0000; }
  to   { text-shadow: 0 0 30px #ff4444, 0 0 60px #ff0000; }
}

.gameover-title {
  font-size: 28px;
  color: #cc0000;
  text-shadow: 3px 3px 0 #4a0000;
  letter-spacing: 4px;
}

.gameover-score {
  font-size: 12px;
  color: #e8d5a3;
}

.gameover-btn {
  font-family: 'Press Start 2P', monospace;
  font-size: 11px;
  color: #e8d5a3;
  background: #1a0a0a;
  border: 2px solid #8b0000;
  padding: 12px 24px;
  cursor: pointer;
  letter-spacing: 2px;
  transition: all 0.1s;

  &:hover {
    background: #8b0000;
    color: #fff;
    box-shadow: 0 0 10px rgba(139, 0, 0, 0.8);
  }

  &:active {
    transform: scale(0.97);
  }
}
</style>