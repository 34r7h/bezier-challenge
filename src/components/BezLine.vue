<script lang="ts">
/**
 *  Line generator with curve controls
 *  Purpose:
 *      Primary: draws a line with curves representing cubic bezier values passed in
 *      Secondary: keeps view updated on value changes from parent component
 */
import { ref, defineComponent, computed } from 'vue';
export default defineComponent({
    name: 'BezierLine',
    emits: ['newCoords'],
    props: {
        coords: {
            type: Array,
            default: [0, 0, 0, 0]
        },
        interact: { // same component for the presets
            type: Boolean,
            default: false,
        },
        name: {
            type: String,
            default: 'bezline-default',
        },
    },
    setup(props, ctx) {
        // define refs [box, c1, c2, path]
        const box = ref<SVGElement>()
        const circleElement_1 = ref<SVGCircleElement[]>([])
        const circleElement_2 = ref<SVGCircleElement[]>([])
        const c1 = computed(() => [props.coords[0], props.coords[1]])
        const c2 = computed(() => [props.coords[2], props.coords[3]])
        var selectedCircle = ref('')
        const path = computed(() => { // construct path from coordinate points
            let [x1, y1, x2, y2] = [...c1.value, ...c2.value]
                .map((coordnum) => +(coordnum!) * 100)
            var p = `M0,100 C${x1},${100 - y1} ${x2},${100 - y2} 100,0`
            return p
        })

        function drag(e: any) {// recursive function to start and during circle movement
            if (!props.interact) return // nothing to do on presets

            // defs
            let boxElement = box.value?.getBoundingClientRect()
            let pointClicked = [ // scale by 100
                (e.clientX - boxElement?.left!) / boxElement?.width! * 100,
                100 - (e.clientY - boxElement?.top!) / boxElement?.height! * 100,
            ]
            let circleElements: { [key: string]: SVGCircleElement } = {
                'circle1': circleElement_1.value[0],
                'circle2': circleElement_2.value[0]
            }
            let points: any = []
            let distances: any = {}
            Object.entries(circleElements).map(circle => {
                // find point of circle
                const circleBox = circle[1].getBBox()
                const circleCenterPoint = [
                    circleBox.x + (circleBox.width / 2),
                    circleBox.y + (circleBox.height / 2)
                ]
                points.push(circleCenterPoint)
                const differenceFromClick = [
                    Math.abs(pointClicked[0] - circleCenterPoint[0]),
                    100 - Math.abs(pointClicked[1] - circleCenterPoint[1]),
                ]
                const combinedDistance = differenceFromClick.reduce((total, distance) => total + distance, 0)
                distances[circle[0]] = combinedDistance
            })
            if (selectedCircle.value == '') {
                selectedCircle.value = distances.circle1 <= distances.circle2 ? '1' : '2'
            }

            // Emits new coords to parent
            ctx.emit('newCoords', selectedCircle.value == '1' ?
                [...pointClicked.map(x => x / 100), ...c2.value] :
                [...c1.value, ...pointClicked.map(x => x / 100)])

            // Set and remove listeners 
            box.value?.addEventListener('pointermove', drag)
            box.value?.addEventListener('pointerup', (e) => {
                ['pointermove', 'pointerup'].forEach(watcher =>
                    box.value?.removeEventListener(watcher, drag))
                selectedCircle.value = ''
            })
        }

        // init and watch
        return {
            box, c1, c2, circleElement_1, circleElement_2, drag, path
        }
    }
})
</script>

<template>
    <div class="container" :id="`svg-${name}`">
        <svg @pointerdown.prevent="drag" style="z-index: 1;" ref="box" viewBox="0 0 100 100" version="1.1"
            xmlns="http://www.w3.org/2000/svg">

            <g id="Sine">
                <path :d="path" fill="none" stroke="#000000" stroke-width="2" stroke-linecap="round" />
                <path v-if="interact" d="M0,100 L100,0" fill="none" stroke="rgba(0,0,0,.4)" stroke-width="1"
                    stroke-linecap="round" />
            </g>
            <g v-for="i in 2">
                <line :x1="i == 1 ? 0 : 100" :y1="i == 1 ? 100 : 0"
                    :x2="+(i == 1 ? c1[0] as Number : c2[0] as Number) * 100"
                    :y2="100 - +(i == 1 ? c1[1] as Number : c2[1] as Number) * 100" :stroke="interact ? '#c680d1' : '#444'"
                    stroke-width="1" />
                <circle :id="`circle${i}`" :ref="`circleElement_${i}`"
                    :cx="+(i == 1 ? c1[0] as Number : c2[0] as Number) * 100"
                    :cy="100 - +(i == 1 ? c1[1] as Number : c2[1] as Number) * 100" :fill="interact ? '#9c27af' : '#444'"
                    r="4" />
            </g>

        </svg>
    </div>
</template>

<style scoped>
.container {
    display: flex;
    flex-direction: column;
    justify-content: center;
}</style>
