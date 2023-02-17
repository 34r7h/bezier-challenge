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
        const dragOffset = ref({ x: 0, y: 0 })
        const dragEnabled = ref<Array<boolean>>([false, false])
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
        var truecoords: any = computed({
            get: () => {
                console.log(props.coords)
                return props.coords
            },
            set: (val) => {
                console.log('setting', { val })
                ctx.emit('newCoords', [...val, ...c.value.c2])
            }
        })
        const coordinates = { // c1,c2 == circles, bh,bw == box w/h
            c1: computed(() => [truecoords.value[0], truecoords.value[1]]),
            c2: computed(() => [truecoords.value[2], truecoords.value[3]]),
            c: computed(() => [...truecoords.value]),
            b: computed(() => ({
                w: box?.value?.getBoundingClientRect().width,
                h: box?.value?.getBoundingClientRect().height,
            }))
        }
        const c = ref(coordinates)
        console.log({ c });


        const path = computed(() => "M1,99 C1,99 1,99 50,50 C50,50 99,1 99,1")

        // Required functions

        function drag1(e: MouseEvent) {
            console.log('drag1', { e });
            const id = (e.target as SVGGraphicsElement)?.id;
            const circleElement: SVGGraphicsElement[] | null =
                id === 'circle1' ? circle1.value : id === 'circle2' ? circle2.value : null;
            if (circleElement) {
                const rect = circleElement[0].getBBox();
                dragOffset.value.x = e.clientX - (rect.left + rect.width / 2);
                dragOffset.value.y = e.clientY - (rect.top + rect.height / 2);
            }
            circleElement?.[0].addEventListener('mousemove', move);
        }

        function move(e: MouseEvent) {
            console.log('move', { e });
            const svgElement = box?.value;
            const id = (e.target as SVGGraphicsElement)?.id;

            const circleElement: SVGGraphicsElement[] | null = id === 'circle1' ? circle1.value : id === 'circle2' ? circle2.value : null;
            console.log({svgElement, circleElement});
            
            if (svgElement && circleElement?.[0]) {
                const bbox = circleElement[0].getBBox();
                const x = bbox.x + bbox.width / 2;
                const y = bbox.y + bbox.height / 2;
                console.log({x,y});
                
                // circleElement.setAttribute('cx', String(x - dragOffset.value.x));
                // circleElement.setAttribute('cy', String(y - dragOffset.value.y));
            }
        }


        function drop(e: MouseEvent) {
            console.log('drop', { e });
            const id = (e.target as SVGGraphicsElement)?.id;
            const circleElement: SVGGraphicsElement[] | null = id === 'circle1' ? circle1.value : id === 'circle2' ? circle2.value : null;
            dragOffset.value.x = dragOffset.value.y = 0;
            box?.value?.removeEventListener('mousemove', move)
        }





        function drawline() { }
        function drag(e: any) {
            if (!dragEnabled.value[0] && !dragEnabled.value[1]) {
                return
            }
            console.log({ e }, dragEnabled.value[0])
            const deltaX = e.movementX / 100
            const deltaY = e.movementY / 100
            var [dc1, dc2] = [
                [c.value.c[0], c.value.c[1]],
                [c.value.c[2], c.value.c[3]]
            ]
            if (e.target.id == 'circle1') {
                dc1 = [
                    dc1[0] + deltaX,
                    dc1[1] + deltaY
                ]
            }
            if (e.target.id == 'circle2') {
                dc2 = [
                    dc2[0] + deltaX,
                    dc2[1] + deltaY
                ]
            }
            console.log({ dc1, dc2 })
            var newCoords = [...dc1, ...dc2]
            console.log({ newCoords });
            return ctx.emit('newCoords', newCoords.map(x => +(x.toFixed(2))))

        }
        function enableDrag(c: number) {
            return dragEnabled.value[c] = !dragEnabled.value[c]
        }
        function handleDragStart(e: any) {
            e.preventDefault();
        }

        // State set and watch
        onMounted(() => {
            console.log({ box: box.value }, box?.value?.getBoundingClientRect());
            nextTick(() => {
                console.log(circle1?.value);
                console.log(circle2?.value);
            })

        })
        watch(() => props, () => drawline(), {
            immediate: true, deep: true
        })
        return { c, path, testcoords, drag, dragEnabled, enableDrag, truecoords, handleDragStart, box, drag1, drop, circle1, circle2 }
    }
})
</script>

<template>
    <div class="container" :id="`canvas-${name}`">
        <div v-if="interact" style="position: absolute; z-index:-1;">
            coords: {{ coords }}
            draggable:{{ dragEnabled }}
            path: {{ path }}
            truecoords: {{ truecoords }}
            c: {{ c }}
            {{ dragEnabled[0] }}
        </div>

        <svg ref="box" viewBox="0 0 100 100" version="1.1" xmlns="http://www.w3.org/2000/svg">
            <g>
                <circle v-for="i in 2" :id="`circle${i}`" :ref="`circle${i}`" draggable @mousedown="drag1" @mouseup="drop"
                    :cx="+(c[i % 2 == 0 ? 'c2' : 'c1'][0] as Number) * 100"
                    :cy="+(c[i % 2 == 0 ? 'c2' : 'c1'][1] as Number) * 100" r="5" fill="red" />
            </g>
            <g id="Sine">
                <path :d="path" fill="none" stroke="#000000" stroke-width="1" stroke-linecap="round" />
            </g>
        </svg>
</div>
</template>

<style scoped></style>
