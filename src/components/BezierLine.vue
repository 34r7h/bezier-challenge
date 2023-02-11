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
        function drawline() { }
        onMounted(() => { })
        watch(() => props, () => drawline(), {
            immediate: true, deep: true
        })
        return {}
    }
})
</script>

<template>
    <div class="container" :id="`canvas-${name}`">
        {{ coords }}
    </div>
</template>

<style scoped>

</style>
