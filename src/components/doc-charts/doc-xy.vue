<script setup>
import { onMounted, ref } from "vue";
import { RouterLink, RouterView } from "vue-router";
import { XY } from "turbo-spark";

const props = defineProps({
    config: {
        type: Object,
        default() {
            return {}
        }
    }
})

let ds1 = [578, 999, 437, 504, 922, 443, 862, 763, 557, 676, 134, 644];
let ds2 = [82, 52, 79, 33, 41, 33, 2, 24, 1, 82, 94, 71] 
let ds3 = [1, 0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0] 
let ds4 = [1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 0, 0]
// for (let i = 0; i < 12; i += 1) {
//     ds1.push(Math.round(Math.random() * 1000));
//     ds2.push(Math.round(Math.random() * 100));
//     ds3.push(Math.round(Math.random() * 1));
//     ds4.push(Math.round(Math.random() * 1));
// }

const chart = ref(null);

const dataset = ref([
    {
        type: "bar",
        name: "Serie 0",
        values: ds1,
        // datapoint_height_ratio: 0.4,
        datapoint_scale_ticks: 3,
        datapoint_line_show_area: true,
        datapoint_line_smooth: false,
        // datapoint_datalabel_show: false,
    },
    {
        type: "line",
        name: "Serie 1",
        values: ds2,
        datapoint_scale_ticks: 3,
        // datapoint_height_ratio: 0.4,
        datapoint_line_show_area: true,
        datapoint_line_smooth: false,
        // datapoint_datalabel_show: false,
    },
    {
        type: "line",
        name: "Serie 2",
        values: ds3,
        datapoint_scale_ticks: 2,
        datapoint_line_smooth: false,
        // datapoint_datalabel_show: false,
        datapoint_line_show_area: true,
    },
    {
        type: "line",
        name: "Serie 3",
        values: ds4,
        datapoint_scale_ticks: 2,
        datapoint_line_smooth: false,
        // datapoint_datalabel_show: false,
        datapoint_line_show_area: true,
    },
]);

const xy = ref(null);

onMounted(() => {
    xy.value = XY({
        container: chart.value,
        dataset: dataset.value,
        config: props.config,
    });
});
</script>

<template>
    <div ref="chart"></div>
</template>
