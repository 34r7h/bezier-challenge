<script lang="ts">
/**
 * An animation speed map
 * Purpose
 *  Primary: displays visual information about animation speed
 * Components: none
 */
import { ref, defineComponent, onMounted, watch, computed } from 'vue';
export default defineComponent({
    props: {
        coords: {
            type: Array,
            default: [0, 0, 0, 0],
        },
    },
    setup(props) {
        const visualizerCircle = ref<SVGCircleElement>()
        const bezierCss = computed(() => String(props.coords.reduce((a, b) => {
            console.log({ a, b });
            const newBezierCoords = `${a} ${Number(b).toFixed(2)},`
            console.log({ newBezierCoords });
            return newBezierCoords
            // String(a).concat(' ' + +(b).toFixed(2)))
        }, '')).slice(1, -1))
        console.log({ bezierCss: bezierCss.value });
        onMounted(() => {
            console.log(visualizerCircle.value)
        })
        // watch(
        //     () => bezierCss.value,
        //     () => {
        //         const circle = visualizerCircle.value;
        //         if (circle) {
        //             let animationTiming = `cubic-bezier(${bezierCss.value}) forwards`
        //             console.log({ animationTiming });
        //             circle.style['animation-timing-function'] = `cubic-bezier(${bezierCss.value})`
        //         }
        //         console.log({ bezierCSS: bezierCss.value }, circle?.style);
        //     }
        // );
        return {
            coords: props.coords, bezierCss, visualizerCircle
        }
    }
})
</script>

<template>
    <div style="position: fixed; font-size: 2vh;">vizok {{ bezierCss }} {{ 'animation: animateWithBezier 3s cubic-bezier(' + bezierCss +
        ');' }}</div>
    <svg v-if="bezierCss" style="z-index: 1;" ref="box" viewBox="0 0 100 12" version="1.1"
        xmlns="http://www.w3.org/2000/svg">
        <g>
            <circle id="visualizer_circle" ref="visualizerCircle" :cx="6" :cy="6" fill="green" r="5"
                :style="'animation-timing-function: cubic-bezier(' + bezierCss + ');'" />
        </g>
    </svg>
</template>

<style scoped>
#visualizer_circle {
    transform: translateX(-10vw);
    animation: animateWithBezier 3s;
    animation-timing-function: cubic-bezier(0,0,0,0);
}

@keyframes animateWithBezier {
    0% {
        transform: translateX(0px);
    }

    100% {
        transform: translateX(90%);
    }
}
</style>
