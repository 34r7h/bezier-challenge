<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import BezLine from './BezLine.vue'
import Visualizer from './BezierVisualizer.vue'

// 2-way binding setup
const props = defineProps({
    modelValue: String
})
const emit = defineEmits({
    'update:modelValue': (value: string) => true
})

// defs
var coords = ref([0, 0, 0, 0])
const cubicBezierString = ref('')
const presetCoords = ref([
    [.4, 0, .6, 1],
    [.4, 0, 1, 1],
    [0, 0, .6, 1],
])

// functions
function updateCoords(newCoords: [number, number, number, number]) {
    cubicBezierString.value = `cubic-bezier(${newCoords.map(c=>c.toFixed(2)).join(', ')})`
    emit("update:modelValue", cubicBezierString.value)
    return coords.value = newCoords
}
const parsedBezierString = computed(() => { // expecting a string, must parse
    if (!props.modelValue) return [0, 0, 0, 0]
    const [...values] = props.modelValue.split('(').pop()!.split(')')[0].split(',')!
    return values.map(Number)
})

onMounted(() => {
    coords.value = parsedBezierString.value
})

</script>

<template @scroll.prevent @touchmove.prevent>
    <article id="cubic-bezier" class="flexcol">
        <section class="visual flexcol">
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
        <section class="show-bezier-string">
            <!-- display the calculated cubic-bezier values -->
            {{ modelValue }}
        </section>
    </article>
</template>

<style scoped>
#cubic-bezier {
    overflow:hidden;
    max-height: 100%;
    max-width: 100%;
}
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
    flex: 2;
    max-height:20%;
}

.interaction {
    flex: 3;
    display: flex;
    width: 100%;
    max-height: 60%;
    align-items: center;
}

.show-bezier-string {
    display: flex;
    width: 100%;
    flex: 1;
    font-size: calc((100vh + 100vw) / 40);
    align-items: center;
    justify-content: space-around;
    max-height:20%;
    font-size: 2vh;
}

.adjust {
    width: 75%;
    padding: 5%;
    flex: 4;
}

.presets {
    width: 25%;
    flex: 1;
}
.presets button {margin: 10% 0; border: 0; border-radius: 5%; padding: 5%; height: 25%;}

.flex {
    display: flex;
    align-items: center;
    justify-content: center;
}

.flexcol {
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: space-around;
}
</style>
