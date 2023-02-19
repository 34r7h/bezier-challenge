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
        const path = computed(() => {
            //  "M0,100 C100,100 0,0 100,0"
            let [x1, y1, x2, y2] = [...c1.value, ...c2.value]
                .map((coordnum) => +(coordnum!) * 100)
            var p = `M0,100 C${x1},${y1} ${x2},${y2} 100,0`
            return p
        })

        // what is the function to translate bez coords into path points?
        // C50,100 50,50 50,50 C50,50 50,0 100,0

        // given [1, 0, 0, 1] produce a hard "S" curve
        // multiply by 100 [100, 0, 0, 100]
        // subtract y from 100 to flip perspective [100, 100, 0, 0]



        // reflection along diag [0, 1, 1, 0]
        // M0,100  C100,100  0,0   100,0
        // m0,100 x1,y2 x2,y1 100,0
        // m0,100 100,100 0,0 100,0


        // 1, 0, 1, 0
        // 100,0 0,100
        //  0,100   50,100   50,50   50,50
        //  100,0   100,0    50,0    50,50


        // [1, 0, 0, 1]

        // find center
        // averge x and average y (* 100)
        // 1+0 /2 0+1 /2
        // 50, 50

        // middle is 50,50

        //     1,0   0,1

        // "M 0,100 
        // C 50,100 50,75 50,50 
        // C 50,25 50,0 100,0"


        // facts:
        // averages     x1y1:.5, x2y2:.5 / x1y2:1, x2y1:0 / x1y1:.5 x2y2:.5
        // normalized       50        50       100     0         50      50


        // total average coord 
        // x = 0+50+50+50+50+50+100 = 350 / 7 = 50
        // y = 100+100+75+50+25+0+0 = 350 / 7 = 50
        // separate c average coords
        // 1) x = 50+50+50 = 150 / 3 = 50, 
        //    y = 100+75+50 =  225 / 3 = 75
        // 2) x = 50 + 50 + 100 = 200 / 3 = 66.666
        //    y = 25 + 0 + 0 = 25 / 3 = 8.333 

        // [1,0,0,0] is a very ugly curve
        // [1,1,0,1] is it's ugly reflection on the diag

        // 50,100 75,75 50,50
        // 100,0 25,0 50,50




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
            "M1,99 C50,99 75,75 50,50 C50,50 25,1 99,1",
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

        // define functions [drag, undrag, update]
        function drag(e: any) {
            if (!props.interact) return
            // find closest circle, snap circle to coords, assign move watcher,
            let boxElement = box.value?.getBoundingClientRect()
            let pointClicked = [ // get relative points within svg box, scale by 100
                (e.clientX - boxElement?.left!) / boxElement?.width! * 100,
                (e.clientY - boxElement?.top!) / boxElement?.height! * 100,
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
                    Math.abs(pointClicked[1] - circleCenterPoint[1]),
                ]
                const combinedDistance = differenceFromClick.reduce((total, distance) => total + distance, 0)
                distances[circle[0]] = combinedDistance
            })
            const closerCircle = distances.circle1 < distances.circle2 ? '1' : '2'
            console.log({ points, distances, closerCircle, pointClicked });

            ctx.emit('newCoords', distances.circle1 < distances.circle2 ?
                [...pointClicked.map(x => x / 100), ...c2.value] :
                [...c1.value, ...pointClicked.map(x => x / 100)])

            console.log('drag', e, circleElement_1?.value?.[0], box.value);

            box.value?.addEventListener('mousemove', drag)
            box.value?.addEventListener('mouseup', (e) => {
                ['mousemove', 'mouseup'].forEach(watcher =>
                    box.value?.removeEventListener(watcher, drag))

            })
        }

        // init and watch
        return {
            box, c1, c2, drag, circleElement_1, circleElement_2, path, testcoords
        }
    }
})
</script>

<template>
    <div class="container" :id="`svg-${name}`">
        <svg @mousedown="drag" style="z-index: 1;" ref="box" viewBox="0 0 101 101" version="1.1"
            xmlns="http://www.w3.org/2000/svg">
            <g id="Sine">
                <path v-for="p in testcoords" :d="path" fill="none" stroke="#000000" stroke-width="2"
                    stroke-linecap="round" />
            </g>
            <g v-for="i in 2">
                <line :x1="i == 1 ? 0 : 100" :y1="i == 1 ? 100 : 0"
                    :x2="+(i == 1 ? c1[0] as Number : c2[0] as Number) * 100"
                    :y2="+(i == 1 ? c1[1] as Number : c2[1] as Number) * 100" stroke="black" stroke-width="1" />
                <circle :id="`circle${i}`" :ref="`circleElement_${i}`" draggable="true" @drag="drag"
                    :cx="+(i == 1 ? c1[0] as Number : c2[0] as Number) * 100"
                    :cy="+(i == 1 ? c1[1] as Number : c2[1] as Number) * 100" :fill="i == 1 ? 'red' : 'blue'" r="4" />
            </g>
        </svg>
        <div style="font-size: 2vh; position: fixed;">c1: {{ c1 }}, c2: {{ c2 }}, path: {{ path }}</div>
</div>
</template>

<style scoped></style>
