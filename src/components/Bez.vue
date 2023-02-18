<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import BezLine from './BezLine.vue'
import Visualizer from './BezierVisualizer.vue'
/**
 * Main component container (Interactive)
 * Purpose
 *  Primary: let a user drag two points around a diagonal line to fine-tune cubic-bezier values
 *  Secondary: display preset values, an animation infographic, and show the current values
 * Components: BezierLine, Visualizer
 */
var coords = ref([0,0,0,0])
const presetCoords = ref([
[.5,1,-0.5,0], // "S" - bottom of the curve is pulled right, top is pulled left
[1,1,1,1], // middle of the curve is towards bottom-right corner
[0,0,0,0], // middle of the curve is towards top-left corner
[.2,.3,.1,.5] // just for kicks
])
function updateCoords(newCoords: [number, number, number, number]){
  console.log('Bezier', {newCoords});
  return coords.value = newCoords
}
</script>

<template class="test">
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
        <section visual>
          <Visualizer :coords="coords"/><!-- Show visualizer -->
        </section>
        <section interaction>
            <div presets flexcol>
                <!-- three vertical buttons, click to assign preset coords -->
                <button v-for="presetCoord in presetCoords" @click="coords = presetCoord">
                <BezLine @click.prevent="()=>null" :coords="presetCoord"/></button>
            </div>
            <div adjust>
                <BezLine interact @newCoords="updateCoords" :coords="coords"/><!-- primary interactive zone -->
            </div>
        </section>
        <section input>
            <!-- display the calculated cubic-bezier values -->
            {{coords}}
        </section>
    </article>
</template>

<style scoped>
.test > * {background:rgba(0,0,0,.9); border: rgba(0,0,0,.4) solid}

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
