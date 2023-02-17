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
        const circleElement_1 = ref<SVGCircleElement[]>([])
        const circleElement_2 = ref<SVGCircleElement[]>([])
        const c1 = computed(()=>[props.coords[0], props.coords[1]])
        const c2 = computed(()=>[props.coords[2], props.coords[3]])

        // define functions [drag, undrag, update]
        function drag(e: any) {
            console.log('drag', e, circleElement_1?.value?.[0]);
            return circleElement_1.value[0].addEventListener('mousemove', move)
        }
        function move(e:any){
            console.log('move', e);
            let newCoords = [
            Number(c1.value[0]) + e.movementX / 100,
            Number(c1.value[1]) + e.movementY / 100,
            ...c2.value
            ]
            ctx.emit('newCoords', newCoords)
            
        }
        onMounted(()=>{
            console.log(circleElement_1.value);
            console.log(circleElement_2.value);
            
        })
        // init and watch
        return {
            c1, c2, drag, circleElement_1, circleElement_2
        }
    }
})
</script>

<template>
    <div class="container" :id="`canvas-${name}`">
        <svg style="z-index: 1;" ref="box" viewBox="0 0 100 100" version="1.1" xmlns="http://www.w3.org/2000/svg">
            <g>
                <circle v-for="i in 2" :id="`circle${i}`" :ref="`circleElement_${i}`" draggable="true" @mousedown="drag" @drag="drag"
                    :cx="+(i == 1 ? c1[0] as Number : c2[0] as Number) * 100"
                    :cy="+(i == 1 ? c1[1] as Number : c2[1] as Number) * 100"
                    :fill="i == 1 ? 'red' : 'blue'" r="5" />
            </g>
        </svg>
        <div>c1: {{ c1 }}, c2: {{ c2 }}</div>
</div>
</template>

<style scoped></style>
