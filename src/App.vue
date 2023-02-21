<script setup lang="ts">
import Bez from './components/Bez.vue';
var cubicBezierString = 'cubic-bezier(0.7, 0.1, 0.4, 0.8)'
</script>

<template>
    <div id="bezier-app">
      <bez v-model="cubicBezierString" />
    </div>
</template>

<style scoped>
#bezier-app {
  height: 100%;
  width: 100%;
}
</style>
