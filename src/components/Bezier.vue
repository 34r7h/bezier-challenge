<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import BezLine from './BezierLine.vue'
import Visualizer from './BezierVisualizer.vue'
/**
 * Main component container (Interactive)
 * Purpose
 *  Primary: let a user drag two points around a diagonal line to fine-tune cubic-bezier values
 *  Secondary: display preset values, an animation infographic, and show the current values
 * Components: BezierLine, Visualizer
 */

</script>

<template>
    <!-- Reference image -->
    <img style="
        object-fit: cover;
        position: fixed;
        z-index: -1;
        height: 100vh;
        width: 100vw;
        top: 0;
        left: 0;
      " src="https://i.imgur.com/7TXbj5h.png" />
    <!-- Start project -->
    <article id="box" flexcol>
        <section visual></section>
        <section interaction>
            <div presets flexcol>
            </div>
            <div adjust>
            </div>
        </section>
        <section input>
        </section>
    </article>
</template>

<style scoped>
#box {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
}
#box > * {
  background: rgba(0, 0, 0, 0.2);
}
[visual] {
  flex: 0 0 16vh;
}
[interaction] {
  flex: 1;
  display: flex;
  width: 100%;
}
[input] {
  display: flex;
  width: 100%;
  flex: 0 0 16vh;
  font-size: calc((100vh + 100vw) / 40);
  align-items: center;
  justify-content: space-around;
}
[adjust] {
  width: 75%;
}
[presets] {
  width: 25%;
}

[flex] {
  display: flex;
  align-items: center;
  justify-content: center;
}

[flexcol] {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}
</style>
