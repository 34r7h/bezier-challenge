<script lang="ts">
/**
 *  An animation speed map
 *  Purpose
 *      Primary: displays visual information about animation speed
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
        // Creates a bezier string from coords array to pass to svg element's :style animation
        const bezierCss = computed(() => String(props.coords.reduce((a, b) => {
            const newBezierCoords = `${a} ${Number(b).toFixed(2)},`
            return newBezierCoords
        }, '')).slice(1, -1))

        onMounted(() => {
            const curve = document.querySelector('path');
            const length = curve?.getTotalLength();
            for (let i = 0; i < length!; i += (length! / 10)) {
                const point = curve?.getPointAtLength(i);
                // console.log(`i: ${i}, x: ${point?.x}, y:  ${100 - point?.y!}, difference from previous x: ${i > 10 ? Number(point?.x!) - Number( curve?.getPointAtLength(i-10).x) : point?.x!}, difference from previous y: ${i > 10 ? Number(point?.y!) - Number( curve?.getPointAtLength(i-10).y) : point?.y!}`);
                
                // The relationship is x is linear, y is the amount of circles within the linear spaces
            }
        })

        return {
            coords: props.coords, bezierCss, visualizerCircle
        }
    }
})
</script>

<template>
    <svg v-if="bezierCss" style="z-index: 1;" ref="box" viewBox="0 0 100 12" version="1.1"
        xmlns="http://www.w3.org/2000/svg">
        <g>
            <circle id="visualizer_circle" ref="visualizerCircle" :cx="6" :cy="6" fill="#c680d1" r="5"
                :style="'animation-timing-function: cubic-bezier(' + bezierCss + ');'" />
        </g>
    </svg>
</template>

<style scoped>
#visualizer_circle {
    transform: translateX(0vw);
    animation: animateWithBezier 1.5s backwards;
    animation-timing-function: cubic-bezier(0, 0, 0, 0);
    opacity: 0;
}

@keyframes animateWithBezier {
    0% {
        opacity: 1;
        transform: translateX(0px);
    }

    100% {
        opacity: 1;
        transform: translateX(89%);
    }
}
</style>
