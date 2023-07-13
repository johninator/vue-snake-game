<template>
  <div class="score"> Score: {{ currentScore }}</div>
</template>

<script setup lang="ts">
// import { defineProps } from 'vue'

const props = defineProps({
  currentScore: Number
})

</script>

<style>
.score {
  position: relative;
  top: 0;
  left: 0;
  font-size: 24px;
  font-weight: bold;
  padding: 10px;
  background-color: #333;
  color: #fff;
}
</style>
