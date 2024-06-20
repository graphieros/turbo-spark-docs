<script setup>
import docXy from '@/components/doc-charts/doc-xy.vue';
import { default_config_xy } from '@/components/doc-charts/defaultConfig';
import { computed, ref, watch } from 'vue';

const configItems = computed(() => {
    return Object.keys(default_config_xy).map(key => {
        return {
            key,
            value: default_config_xy[key],
            type: detectInputType(key)
        }
    })
})

const config = ref({
    // CHART SETTINGS
    chart_height: 309,
    chart_width: 500,
    chart_padding_top: 24,
    chart_padding_left: 48,
    chart_padding_right: 12,
    chart_padding_bottom: 48,
    chart_background: "#FFFFFF",
    chart_area_background_show: true,
    chart_area_background: "#1A1A1A",
    chart_area_background_opacity: 0.01,
    chart_custom_palette: [],

    // GRID SETTINGS
    grid_axis_x_name: '',
    grid_axis_y_name: '',
    grid_axis_y_name_offset_x: -30,
    grid_axis_x_name_offset_y: 40,
    grid_axis_names_font_size: 10,
    grid_axis_names_color: '#1A1A1A',
    grid_axis_stroke: "#CCCCCC",
    grid_axis_stroke_width: 0.5,
    grid_axis_x_show: true,
    grid_axis_y_show: true,
    grid_axis_y_scale_ticks: 5,
    grid_lines_y_show: true,
    grid_lines_y_stroke: "#CCCCCC",
    grid_lines_y_stroke_width: 0.5,
    grid_lines_y_stroke_dasharray: 0,
    grid_lines_x_show: true,
    grid_lines_x_stroke: "#CCCCCC",
    grid_lines_x_stroke_width: 0.5,
    grid_lines_x_stroke_dasharray: 2,
    grid_lines_x_stroke_opacity: 0.5,
    grid_lines_y_stroke_opacity: 0.3,

    // LABEL SETTINGS
    label_axis_y_bold: false,
    label_axis_y_color: '#1A1A1A',
    label_axis_y_font_size: 10,
    label_axis_y_rounding: 1,
    label_axis_y_show: true,
    label_axis_y_offset_x: 0,
    label_axis_y_offset_y: 0,
    label_prefix: '',
    label_suffix: '',
    label_axis_x_values: [],
    label_axis_x_bold: false,
    label_axis_x_color: '#1A1A1A',
    label_axis_x_font_size: 12,
    label_axis_x_rounding: 1,
    label_axis_x_show: true,
    label_axis_x_offset_x: 0,
    label_axis_x_offset_y: 0,
    label_axis_x_rotation: 0,

    // DATA LABEL SETTINGS
    datalabel_show: true,
    datalabel_use_serie_color: true,
    datalabel_default_color: '#1A1A1A',
    datalabel_font_size: 10,
    datalabel_rounding: 1,
    datalabel_offset_y: -6,

    // TOOLTIP SETTINGS
    tooltip_show: true,
    tooltip_value_rounding: 1,
    tooltip_background_color: "#FFFFFF",
    tooltip_font_size: 14,
    tooltip_color: "#1A1A1A",
    tooltip_padding: 12,
    tooltip_border_radius: 4,
    tooltip_border: "1px solid #e1e5e8",
    tooltip_box_shadow: '0 0 12px -6px rgba(0,0,0,0.3)',
    tooltip_max_width: 255,
    tooltip_custom: null,
    tooltip_marker_size: 14,

    // LINE SETTINGS
    line_smooth: false,
    line_smooth_force: 0.15, // between 0 and 0.2
    line_sheathed: true,
    line_stroke_width: 1,
    line_area_opacity: 0.2,

    // PLOT SETTINGS
    plot_radius: 3,
    plot_stroke: "#FFFFFF",
    plot_stroke_width: 1,
    plot_focus_radius: 5,

    // BAR SETTINGS
    bar_border_radius: 2,
    bar_group_gap_proportion: 0.3,
    bar_stroke: '#FFFFFF',
    bar_stroke_width: 0.75,

    // SELECTOR
    selector_show: true,
    selector_stroke: "#CCCCCC",
    selector_stroke_width: 1,
    selector_stroke_dasharray: 2,

    // SERIES SETTINGS
    series_stacked: true,
    series_stack_gap: 20,

    // LEGEND SETTINGS
    legend_show: true,
    legend_background: "#FFFFFF",
    legend_color: "#1A1A1A",
    legend_font_size: 14,
    legend_marker_size: 14,

    // TITLE SETTINGS
    title_show: true,
    title_background: '#FFFFFF',
    title_align: 'center',
    title_text: "Lorem ipsum",
    title_color: "#1A1A1A",
    title_font_size: 20,
    title_bold: true,
    subtitle_text: "Dolor sit amet",
    subtitle_color: "#8A8A8A",
    subtitle_font_size: 16,
    subtitle_bold: false,

    // TABLE SETTINGS
    table_details_title: "Data table",
    table_background: '#FFFFFF',
    table_color: '#1A1A1A',
    table_show: true,
    table_caption: "",
    table_font_size: 14,

    // ZOOM SETTINGS
    zoom_background: "#91edc2",
    zoom_stroke: "#40c98a",
    zoom_stroke_width: 1,
    zoom_opacity: 0.2,

    // PERIOD HIGHLIGHTER SETTINGS
    period_highlighter_show: true,
    period_highlighter_font_size: 10,
    period_highlighter_offset_y: 6,
    period_highlighter_background: "#e1e5e8",
    period_highlighter_color: "#1A1A1A",
    period_highlighter_width: 64,
    period_highlighter_height: 18,
    period_highlighter_box_shadow: '0 0 6px -3px rgba(0,0,0,0.3)'
})

function detectInputType(key) {
    const colorPatterns = ['color', 'background', 'stroke'];
    const numberPatterns = ['width', 'height', 'font_size', 'radius', 'padding', 'offset', 'size', 'opacity', 'rounding', 'dasharray', 'ticks', 'force'];
    const booleanPatterns = ['show', 'bold', 'stacked', 'custom', 'sheathed', 'smooth', 'use_serie_color'];

    // Check for boolean types first
    if (booleanPatterns.some(pattern => key.includes(pattern))) {
        return 'checkbox';
    }

    // Check for known exceptions for numbers
    if (key.includes('opacity') && !key.includes('background')) {
        return 'number';
    }
    if (key.includes('dasharray') || key.includes('width') && !key.includes('stroke_width')) {
        return 'number';
    }

    // Check for number types
    if (numberPatterns.some(pattern => key.includes(pattern))) {
        return 'number';
    }

    // Check for color types
    if (colorPatterns.some(pattern => key.includes(pattern))) {
        return 'color';
    }

    // Default to text for all other cases
    return 'text';
}
const step = ref(0);

</script>

<template>
    <div class="flex flex-row relative">
        <div class="config bg-gray-800 w-full max-h-[600px] overflow-auto p-6">
            <ul>
                <li v-for="item in configItems" class="flex flex-row gap-2 my-1">
                    <code>
                        <input class="w-[150px]" :type="item.type" v-model="config[item.key]" @change="step += 1"/>
                    </code>
                    <code>{{ item.key }}</code>
                </li>
            </ul>
        </div>
        <div class="w-full max-w-[800px] h-[600px] mx-auto bg-white relative">
            <div class="">
                <docXy :config="config" :key="step"/>
            </div>
        </div>
    </div>
</template>

<style>
input {
    color: black !important;
    padding: 0 6px;
}
</style>