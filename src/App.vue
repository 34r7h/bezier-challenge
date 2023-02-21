<script setup lang="ts">
import { ref } from 'vue'
import EasingEditor from './components/Bez.vue';

// defs
var easing = ref('cubic-bezier(0.7, 0.1, 0.4, 0.8)') // initial coords
const updateCubicBezierString = (val: string) => { // 2-way binding to update
  easing.value = val;
};

</script>

<template>
  <div id="bezier-app">
    <easing-editor v-model="easing" @update:modelValue="updateCubicBezierString"  />
  </div>
</template>

<style scoped>

#bezier-app {
  height: 50vh;
  width: 50vh;
  border: 1px solid rgba(0,0,0,.2);
  padding: 16px;
  border-radius: 5vh;
}
</style>
