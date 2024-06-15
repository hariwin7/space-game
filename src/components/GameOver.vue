<template>
  <div v-if="props.gameOver" class="game-over">
    <div class="game-over-text">
      <h1>Game Over</h1>
      <h1>Press Enter/Return key to restart</h1>
    </div>
  </div>
</template>

<script setup lang="ts">
const props = defineProps({ gameOver: Boolean })
</script>

<style>
.game-over {
  position: absolute;
  top: 50%;
  right: 30%;
  color: white;
  animation: animate 1.5s linear infinite;
}
.game-over-text {
  display: flex;
  flex-direction: column;
  align-items: center;
}

@keyframes animate {
  0% {
    opacity: 0;
  }
  50% {
    opacity: 0.7;
  }
  100% {
    opacity: 0;
  }
}
</style>
