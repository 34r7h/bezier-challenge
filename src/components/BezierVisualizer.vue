<script lang="ts">
/**
 * An animation speed map
 * Purpose
 *  Primary: displays visual information about animation speed
 * Components: none
 */
import { ref, defineComponent, onMounted, nextTick, watch, reactive, toRefs } from 'vue';
export default defineComponent({
    props:{
        coords: {
            type: Array,
            default: [0, 0, 0, 0],
        },
    },
    setup() {
        const state = reactive({
            count: 0,
        })

        return {
            ...toRefs(state),
        }
    }
})
</script>

<template>
    <div>viz{{coords}}</div>
</template>

<style scoped>

</style>
