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
        const box = ref<HTMLElement | null>(null)
        const circle1 = ref<SVGGraphicsElement[]>([])
        const circle2 = ref<SVGGraphicsElement[]>([])
        const dragOffsetX = ref(0)
        const dragOffsetY = ref(0)
        const testcoords = ref([
            //Straight line
            "M1,99 C1,99 1,99 50,50 C50,50 99,1 99,1",
            //Straight line
            "M1,1 C1,1 99,99 50,50 C50,50 99,99 99,99",
            // Up, over, up hard
            "M1,99 C1,99 1,25 50,50 C99,75 99,1 99,1",
            // Up, over, up
            "M1,99 C1,99 1,50 50,50 C99,50 99,1 99,1",
            // over, up, over
            "M1,99 C50,99 50,50 50,50 C50,50 50,1 99,1",
            // over, up, over hard
            "M99,0 C99,0 75,0 50,50 C25,99 0,99 0,99",

            // over, up
            "M1,99 C99,99 99,50 99,1",
            // up, over
            "M1,99 C1,1 50,1 99,1",
            // over, up hard
            "M1,99 C99,99 99, 99 99,1",
            // up, over hard
            "M1,99 C1,1 1,1 99,1"
            /**
             * ANALYSIS
             * All curves start and stop at 1,99 and 99,1 respectively. 
             */
        ])
        const c1 = computed(() => [props.coords[0], props.coords[1]] as [number, number])
        const c2 = computed(() => [props.coords[2], props.coords[3]] as [number, number])
        const boxSize = computed(() => box?.value?.getBoundingClientRect())
        
        // const coordinates = { // c1,c2 == circles, bh,bw == box w/h
        //     c1: computed(() => [props.coords[0], props.coords[1]]),
        //     c2: computed(() => [props.coords[2], props.coords[3]]),
        //     c: computed(() => props.coords),
        //     b: computed(() => ({
        //         w: box?.value?.getBoundingClientRect().width,
        //         h: box?.value?.getBoundingClientRect().height,
        //     }))
        // }
        // const c = ref(coordinates)
        // console.log({ c });


        const path = computed(() => "M1,99 C1,99 10,99 50,50 C50,60 99,1 99,1")

        // Required functions

        function drag1(e: MouseEvent) {
            console.log('drag1', { e });
            const id = (e.target as SVGGraphicsElement)?.id;
            const circleElement: SVGGraphicsElement[] | null =
                id === 'circle1' ? circle1.value : id === 'circle2' ? circle2.value : null;
            console.log({circleElement})
            if (circleElement) {
                const cir = circleElement[0].getBBox();
                console.log({cir,  offset: [e.offsetX, e.offsetY] }, { c: [boxSize.value, c1.value, c2.value] }, { cir })
                dragOffsetX.value = (e.offsetX - (cir.width / 2)) / boxSize.value?.width!;
                dragOffsetY.value = (e.offsetY - (cir.height / 2)) / boxSize.value?.height!;
                console.log({ dragOffsetX, dragOffsetY }, [e.offsetX - (cir.width / 2),  e.offsetY - (cir.height / 2)])
            }
            box?.value?.addEventListener('mousemove', move);
        }

        function move(e: MouseEvent) {
            console.log('move', { e });
            const svgElement = box?.value;
            const id = (e.target as SVGGraphicsElement)?.id;

            const circleElement: SVGGraphicsElement[] | null = id === 'circle1' ? circle1.value : id === 'circle2' ? circle2.value : null;
            console.log({ svgElement, circleElement });

            if (svgElement && circleElement?.[0]) {
                // const bbox = circleElement[0].getBBox();
                // console.log({ bbox });
                console.log({ cb: box.value, offsets: {dragOffsetX, dragOffsetY} });
                const ratios = {
                    x: (dragOffsetX.value / (boxSize?.value?.width!)) * 100,
                    y: (dragOffsetY.value /  (boxSize?.value?.height!)) * 100,
                }
                console.log({ ratios });
                console.log('Circle1:', { x: [c1.value[0], ratios.x] }, { y: [c1.value[1], ratios.y] });
                let newCoords = (id === 'circle1' ?
                    [Math.ceil(c1.value[0] + ratios.x), Math.ceil(c1.value[1] + ratios.y), ...c2.value] :
                    [...c1.value, Math.ceil(c2.value[0] + ratios.x), Math.ceil(c2.value[1] + ratios.y)])
                    .map(x => +(x / 100)
                    .toFixed(2))

                console.log('move', { newCoords });
                circleElement[0].setAttribute('cx', newCoords[id === 'circle1' ? 0 : 2].toString());
                circleElement[0].setAttribute('cy', newCoords[id === 'circle1' ? 1 : 3].toString());
                ctx.emit('newCoords', newCoords)
            }
        }

        function drop(e: MouseEvent) {
            console.log('drop', { e });
            const id = (e.target as SVGGraphicsElement)?.id;
            const circleElement: SVGGraphicsElement[] | null = id === 'circle1' ? circle1.value : id === 'circle2' ? circle2.value : null;
            dragOffsetX.value = dragOffsetY.value = 0;
            box?.value?.removeEventListener('mousemove', move)
            ctx.emit('newCoords', [...c1.value, ...c2.value])
        }

        function drawline() { }

        // State set and watch
        watch(() => props, () => drawline(), {
            immediate: true, deep: true
        })
        return { c1, c2, path, {coords: props.coords}, testcoords, box, drag1, drop, circle1, circle2, dragOffsetX, dragOffsetY }
    }
})
</script>

<template>
    <div class="container" :id="`canvas-${name}`">
        <div v-if="interact" style="position: absolute; z-index:-1; bottom:0">
            coords: {{ coords }}
            <hr>
            c1, c2: {{ c1 }}, {{ c2 }}
            <hr>
            path: {{ path }}
            <hr>
            dragOffset: {{ dragOffsetX }}, {{ dragOffsetY }}
        </div>

        <svg style="z-index: 1;" ref="box" viewBox="0 0 100 100" version="1.1" xmlns="http://www.w3.org/2000/svg">
            <g>
                <circle v-for="i in 2" :id="`circle${i}`" :ref="`circle${i}`" draggable @mousedown="drag1" @mouseup="drop"
                    :cx="(i == 1 ? coords[0] : c2[0]) * 100"
                    :cy="(i == 1 ? c1[1] : c2[1]) * 100" r="5" :fill="i == 1 ? 'red' : 'blue'" />
            </g>
            <g id="Sine">
                <path :d="path" fill="none" stroke="#000000" stroke-width="1" stroke-linecap="round" />
            </g>
        </svg>
</div>
</template>

<style scoped></style>
