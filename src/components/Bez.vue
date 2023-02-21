<script setup lang="ts">
import { ref, computed, onMounted, defineProps } from 'vue'
import BezLine from './BezLine.vue'
import Visualizer from './BezierVisualizer.vue'


const props = defineProps({
  modelValue: String
})

var coords = ref([0, 0, 0, 0])
const cubicBezierString = ref('')
const presetCoords = ref([
    [.4, 0, .6, 1],
    [.4, 0, 1, 1],
    [0, 0, .6, 1],
])



function updateCoords(newCoords: [number, number, number, number]) {
    cubicBezierString.value = `cubic-bezier(${newCoords.join(', ')})`
    return coords.value = newCoords
}
console.log(props.modelValue);

// function updateValue(newValue: string) {
//   value.value = newValue
// }

const parsedBezierString = computed(() => {
    if (!props.modelValue) return [0, 0, 0, 0]
    const [...values] = props.modelValue.split('(').pop()!.split(')')[0].split(',')!
    console.log(values)
    return values.map(Number)
})

onMounted(() => {
    // console.log({ parsedBezierString, value });
    coords.value = parsedBezierString.value
})



</script>

<template class="test">
    <article id="cubic-bezier" class="flexcol">
        <section class="visual">
            <Visualizer :key="coords.join(',')" :coords="coords" /><!-- Show visualizer -->
        </section>
        <section class="interaction">
            <div class="presets flexcol">
                <!-- three vertical buttons, click to assign preset coords -->
                <button class="" v-for="presetCoord in presetCoords" @click="coords = presetCoord">
                    <BezLine @click.prevent="() => null" :coords="presetCoord" />
                </button>
            </div>
            <div class="adjust">
                <BezLine interact @newCoords="updateCoords" :coords="coords" /><!-- primary interactive zone -->
            </div>
        </section>
        <section class="input">
            <!-- display the calculated cubic-bezier values -->
            <span style="font-size: 2vh; position: fixed;">cubic-bezier(<span v-for="coord in coords">{{ coord.toFixed(2)
            }},
                </span>)</span>
            <!-- todo remove trailing comma -->
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

#box>* {
    background: rgba(0, 0, 0, 0.2);
}

.visual {
    flex: 0 0 16vh;
}

.interaction {
    flex: 1;
    display: flex;
    width: 100%;
}

.input {
    display: flex;
    width: 100%;
    flex: 0 0 16vh;
    font-size: calc((100vh + 100vw) / 40);
    align-items: center;
    justify-content: space-around;
}

.adjust {
    width: 75%;
}

.presets {
    width: 25%;
}

.flex {
    display: flex;
    align-items: center;
    justify-content: center;
}

.flexcol {
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: stretch;
}
</style>
