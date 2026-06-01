<template>
  <div class="game-board">
    <div class="board-grid"></div>

    <!-- Player -->
    <div
      class="player"
      :style="{ left: player.x + 'px', top: player.y + 'px' }"
    >🧛</div>

    <!-- Hazards -->
    <div
      v-for="hazard in hazards"
      :key="hazard.id"
      class="hazard"
      :style="{ left: hazard.x + 'px', top: hazard.y + 'px' }"
    >💀</div>
  </div>
</template>

<script setup>
defineProps({
  player: Object,
  hazards: Array
})
</script>

<style scoped lang="scss">
.game-board {
  width: 800px;
  height: 500px;
  background: #0d1117;
  border: 2px solid #8b0000;
  box-shadow: 0 0 20px rgba(139, 0, 0, 0.4), inset 0 0 40px rgba(0,0,0,0.8);
  position: relative;
  overflow: hidden;
  margin: auto;
  image-rendering: pixelated;
}

.board-grid {
  position: absolute;
  inset: 0;
  background-image:
    linear-gradient(rgba(139, 0, 0, 0.07) 1px, transparent 1px),
    linear-gradient(90deg, rgba(139, 0, 0, 0.07) 1px, transparent 1px);
  background-size: 20px 20px;
  pointer-events: none;
}

.player {
  width: 32px;
  height: 32px;
  position: absolute;
  font-size: 24px;
  line-height: 32px;
  text-align: center;
  filter: drop-shadow(0 0 6px #ff4444);
  transform: translate(-50%, -50%);
  z-index: 10;
  transition: left 0.05s, top 0.05s;
}

.hazard {
  width: 28px;
  height: 28px;
  position: absolute;
  font-size: 20px;
  line-height: 28px;
  text-align: center;
  filter: drop-shadow(0 0 4px #aa0000);
  transform: translate(-50%, -50%);
  animation: float 2s ease-in-out infinite alternate;
}

@keyframes float {
  from { transform: translate(-50%, -50%) scale(1); }
  to   { transform: translate(-50%, -52%) scale(1.08); }
}
</style>