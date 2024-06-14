<template>
  <div class="wrapper">
    <div class="score">Score: {{ props.score }}</div>
    <div class="score">Level: {{ props.level.toFixed() }}</div>
    <div class="score">Life: {{ props.life }}</div>
  </div>
</template>

<script setup lang="ts">
const props = defineProps({ score: Number, level: Number, life: Number })
</script>

<style>
.wrapper {
  display: flex;
  gap: 10px;
}
.score {
  font-size: 30px;
  color: white;
  font-weight: 600;
}
</style>
