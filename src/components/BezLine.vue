<script lang="ts">
/**
 * Line generator component
 * Purpose
 *  Primary: draws a line with curves representing cubic bezier values passed in
 *  Secondary: keeps view updated on value changes from parent component
 * Components: none
 */
import { ref, defineComponent, onMounted, computed, watch, nextTick } from 'vue';
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

        // define functions [drag, undrag, update]
        function drag(e: any) {
            if (!props.interact) return
            // find closest circle, snap circle to coords, assign move watcher,
            let boxElement = box.value?.getBoundingClientRect()
            let pointClicked = [ // get relative points within svg box, scale by 100
                (e.clientX - boxElement?.left!) / boxElement?.width! * 100,
                (e.clientY - boxElement?.top!) / boxElement?.height! * 100,
            ]
            let circleElements = {
                circle1: circleElement_1.value[0],
                circle2: circleElement_2.value[0]
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
                const distanceFromClick = [
                    Math.abs(pointClicked[0] - circleCenterPoint[0]),
                    Math.abs(pointClicked[1] - circleCenterPoint[1]),
                ]
                const combinedDistance = distanceFromClick.reduce((total, distance) => total + distance, 0)
                distances[circle[0]] = combinedDistance
            })
            const closerCircle = distances.circle1 < distances.circle2 ? '1' : '2'
            console.log({ points, distances, closerCircle, pointClicked });

            ctx.emit('newCoords', distances.circle1 < distances.circle2 ?
                [...pointClicked.map(x => x / 100), ...c2.value] :
                [...c1.value, ...pointClicked.map(x => x / 100)])

            console.log('drag', e, circleElement_1?.value?.[0], box.value);

            circleElements[`circle${closerCircle}`]
                .addEventListener('mousemove', move)
            box.value?.addEventListener('mouseup', (e) => {
                circleElements.circle1
                    .removeEventListener('mousemove', move)
                circleElements.circle2
                    .removeEventListener('mousemove', move)
            })
        }
        function move(e: any) {
            let boxElement = box.value?.getBoundingClientRect()
            let newX = (e.clientX - boxElement?.left!) / boxElement?.width!
            let newY = (e.clientY - boxElement?.top!) / boxElement?.height!
            
            ctx.emit('newCoords', e.target.id == 'circle1' ?
                [newX, newY, ...c2.value] :
                [...c1.value, newX, newY])
        }

        onMounted(() => {
            console.log(circleElement_1.value);
            console.log(circleElement_2.value);

        })
        // init and watch
        return {
            box, c1, c2, drag, circleElement_1, circleElement_2
        }
    }
})
</script>

<template>
    <div class="container" :id="`svg-${name}`">
        <svg @mousedown="drag" style="z-index: 1;" ref="box" viewBox="0 0 100 100" version="1.1"
            xmlns="http://www.w3.org/2000/svg">
            <g>
                <circle v-for="i in 2" :id="`circle${i}`" :ref="`circleElement_${i}`" draggable="true" @drag="drag"
                    :cx="+(i == 1 ? c1[0] as Number : c2[0] as Number) * 100"
                    :cy="+(i == 1 ? c1[1] as Number : c2[1] as Number) * 100" :fill="i == 1 ? 'red' : 'blue'" r="5" />
            </g>
        </svg>
        <div>c1: {{ c1 }}, c2: {{ c2 }}</div>
</div>
</template>

<style scoped></style>
