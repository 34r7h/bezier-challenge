<script setup lang="ts">
import { ref } from 'vue'
import Bez from './components/Bez.vue';
var cubicBezierString = ref('cubic-bezier(0.7, 0.1, 0.4, 0.8)')
const updateCubicBezierString = (val: string) => {
  cubicBezierString.value = val;
};
setTimeout(() => {
  cubicBezierString.value = 'cubic-bezier(0.1, 0.7, 0.8, 0.4)';
}, 1000);
// return { cubicBezierString, updateCubicBezierString };
</script>

<template>
  <div id="bezier-app">
    <bez v-model="cubicBezierString" @update:modelValue="updateCubicBezierString"  />
  </div>
</template>

<style scoped>

#bezier-app {
  height: 50%;
  width: 50%;
  border: 1px solid rgba(0,0,0,.2);
  padding: 16px;
  border-radius: 5vh;
}
</style>
