<script lang="ts">
/**
 * Line generator component
 * Purpose
 *  Primary: draws a line with curves representing cubic bezier values passed in
 *  Secondary: keeps view updated on value changes from parent component
 * Components: none
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
        interact: {
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
        const path = computed(() => {
            let [x1, y1, x2, y2] = [...c1.value, ...c2.value]
                .map((coordnum) => +(coordnum!) * 100)
            var p = `M0,100 C${x1},${100 - y1} ${x2},${100 - y2} 100,0`
            return p
        })

        // define functions [drag, undrag, update]
        function drag(e: any) {
            if (!props.interact) return
            // find closest circle, snap circle to coords, assign move watcher,
            let boxElement = box.value?.getBoundingClientRect()
            let pointClicked = [ // get relative points within svg box, scale by 100
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
            // console.log({ points, distances, pointClicked, selectedCircle });

            ctx.emit('newCoords', selectedCircle.value == '1' ?
                [...pointClicked.map(x => x / 100), ...c2.value] :
                [...c1.value, ...pointClicked.map(x => x / 100)])

            // console.log('drag', e, circleElement_1?.value?.[0], box.value);

            box.value?.addEventListener('mousemove', drag)
            box.value?.addEventListener('mouseup', (e) => {
                ['mousemove', 'mouseup'].forEach(watcher =>
                    box.value?.removeEventListener(watcher, drag))
                selectedCircle.value = ''
            })
        }

        // init and watch
        return {
            box, c1, c2, drag, circleElement_1, circleElement_2, path
        }
    }
})
</script>

<template>
    <div class="container" :id="`svg-${name}`">
        <svg @mousedown="drag" style="z-index: 1;" ref="box" viewBox="0 0 100 100" version="1.1"
            xmlns="http://www.w3.org/2000/svg">

            <g id="Sine">
                <path :d="path" fill="none" stroke="#000000" stroke-width="2" stroke-linecap="round" />
            </g>
            <g v-for="i in 2">
                <line :x1="i == 1 ? 0 : 100" :y1="i == 1 ? 100 : 0"
                    :x2="+(i == 1 ? c1[0] as Number : c2[0] as Number) * 100"
                    :y2="100 - +(i == 1 ? c1[1] as Number : c2[1] as Number) * 100" stroke="black" stroke-width="1" />
                <circle :id="`circle${i}`" :ref="`circleElement_${i}`" draggable="true" @drag="drag"
                    :cx="+(i == 1 ? c1[0] as Number : c2[0] as Number) * 100"
                    :cy="100 - +(i == 1 ? c1[1] as Number : c2[1] as Number) * 100" :fill="i == 1 ? 'red' : 'blue'" r="4" />
            </g>

        </svg>
</div>
</template>

<style scoped></style>
