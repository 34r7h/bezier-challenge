<script lang="ts">
/**
 * Line generator component
 * Purpose
 *  Primary: draws a line with curves representing cubic bezier values passed in
 *  Secondary: keeps view updated on value changes from parent component
 * Components: none
 */
import { ref, defineComponent, onMounted, nextTick, watch } from 'vue';
export default defineComponent({
    name: 'BezierLine',
    props: {
        coords: {
            type: Array,
            default: [0, 0, 0, 0],
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
    setup(props, refs) {

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
        function drawline() { }
        onMounted(() => { })
        watch(() => props, () => drawline(), {
            immediate: true, deep: true
        })
        return { testcoords }
    }
})
</script>

<template>
    <div class="container" :id="`canvas-${name}`">
        {{ coords }}
        <svg viewBox="0 0 100 100" version="1.1" xmlns="http://www.w3.org/2000/svg">
            <g id="Sine">

                <path v-for="(testcoord, i) in testcoords" :d="testcoord" fill="none" :stroke="`#0${i}${i}f${i}f`"
                    :stroke-width="1" stroke-linecap="round" />


            </g>
        </svg>
    </div>
</template>

<style scoped>

</style>
