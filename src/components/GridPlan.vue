<script setup>
import { computed, ref } from 'vue';

const props = defineProps({
    /**
     * Already placed elements on the grid (walls, other entities...)
     * @example
     * [
     *  { x: 1, y: 1, w: 2, h: 3, color: "#FF0000" },
     *  { x: 1, y: 4, w: 2, h: 3, color: "#FF0000" },
     * ]
     */
    items: {
        type: Array,
        default() {
            return []
        }
    },
    /**
     * The active entity that can be moved and resized on available space
     * If none provided, the item will be positioned on the first available square
     * @example
     * { x: 1, y: 1, w: 1, h: 1, "#FF0000" }
     */
    activeEntity: {
        type: Object,
        default() {
            return {}
        }
    },
    gridSize: {
        type: Object,
        default() {
            return {
                w: 20,
                h: 20,
                hType: 'numeric',
                wType: 'alpha'
            }
        }
    }
})

const config = ref({
    gridFill: '#3A3A3A',
    gridStroke: '#000000',
    gridStrokeWidth: 0.02,
    wallFill: '#8A8A8A',
    entityFill: props.activeEntity.color,
    handleFill: '#FFFFFF',
    handleSize: 0.3,
    sizeRatio: 10
})

const emit = defineEmits(['change', 'selectItem', 'triggerAction', 'delete'])

function selectItem(item) {
    emit('selectItem', item)
}

function triggerAction(rect) {
    emit('triggerAction', rect)
}

const width = ref(props.gridSize.w);
const height = ref(props.gridSize.h);
const SVG = ref(null);

const gridRects = computed(() => {
    const rects = [];
    for (let i = 0; i < height.value; i += 1) {
        for (let j = 0; j < width.value; j += 1) {
            rects.push({
                x: j,
                y: i
            });
        }
    }
    return rects;
});

function convertElementsToIndividualWalls(element) {
    const walls = [];
    element.forEach(coordinates => {
        for (let i = 0; i < coordinates.h; i += 1) {
            for (let j = 0; j < coordinates.w; j += 1) {
                walls.push({
                    x: coordinates.x + j,
                    y: coordinates.y + i
                });
            }
        }
    });
    return walls;
}

function numberToLetters(num) {
    let letters = '';
    while (num > 0) {
        let remainder = (num - 1) % 26;
        letters = String.fromCharCode(65 + remainder) + letters;
        num = Math.floor((num - 1) / 26);
    }
    return letters;
}

const gridCoordinates = computed(() => {
    const abs = [];
    const ord = [];

    for (let i = 1; i <= props.gridSize.w; i += 1) {
        if (props.gridSize.wType === 'alpha') {
            abs.push(numberToLetters(i));
        } else {
            abs.push(i);
        }
    }
    for (let i = 1; i <= props.gridSize.h; i += 1) {
        if (props.gridSize.hType === 'alpha') {
            ord.push(numberToLetters(i));  
        } else {
            ord.push(i);
        }
    }

    return {
        abs,
        ord
    };
});

function findFirstAvailableSquare(gridWidth, gridHeight, occupiedSquares) {
    const occupiedSet = new Set(
        occupiedSquares.map(square => `${square.x},${square.y}`)
    );

    for (let y = 0; y < gridHeight; y++) {
        for (let x = 0; x < gridWidth; x++) {
            if (!occupiedSet.has(`${x},${y}`)) {
                return { x, y };
            }
        }
    }

    return null; // No available square found
}

const walls = ref(convertElementsToIndividualWalls(props.items.filter(el => el.id !== props.activeEntity.id)))

function computeEntityCoordinates() {
    if(!props.activeEntity.x || !props.activeEntity.y) {
        const firstAvailableSquare = findFirstAvailableSquare(width.value, height.value, walls.value);

        if (!firstAvailableSquare) {
            console.error('There is no available space.');
        }

        return {
            ...firstAvailableSquare,
            h: 1,
            w: 1
        }
    } else {
        return props.activeEntity;
    }
}

const entity = ref(props.activeEntity || {})

const isDown = ref(false);
const resizingHandle = ref(null);
const initialDimensions = ref({ x: 0, y: 0, w: 0, h: 0 });

function getSVGCoords(evt, svg) {
    const pt = svg.createSVGPoint();
    if (evt.touches && evt.touches.length) {
        pt.x = evt.touches[0].clientX;
        pt.y = evt.touches[0].clientY;
    } else {
        pt.x = evt.clientX;
        pt.y = evt.clientY;
    }

    const svgMatrix = svg.getScreenCTM();
    const transformedPt = pt.matrixTransform(svgMatrix.inverse());

    return {
        x: transformedPt.x,
        y: transformedPt.y
    };
}

function move(e) {
    if (!isDown.value) return;

    e.preventDefault();

    const coordinates = getSVGCoords(e, SVG.value);
    const rounded = {
        x: Math.round(coordinates.x),
        y: Math.round(coordinates.y)
    };

    if (resizingHandle.value) {
        resizeEntity(rounded);
    } else {
        const newX = Math.round(coordinates.x - entity.value.w / 2);
        const newY = Math.round(coordinates.y - entity.value.h / 2);

        if (isColliding(newX, newY, entity.value.w, entity.value.h)) return;
        if (isOverflowing(newX, newY, entity.value.w, entity.value.h)) return;

        entity.value.x = newX;
        entity.value.y = newY;
    }
}

function isColliding(x, y, w, h) {
    for (let i = 0; i < w; i += 1) {
        for (let j = 0; j < h; j += 1) {
            if (walls.value.some(wall => wall.x === x + i && wall.y === y + j)) {
                return true;
            }
        }
    }
    return false;
}

function isOverflowing(x, y, w, h) {
    return x + w > width.value || y + h > height.value || x < 0 || y < 0;
}

function drop(e) {
    isDown.value = false;
    resizingHandle.value = null;
    emit('change', entity.value);
}

function startResize(handle) {
    isDown.value = true;
    resizingHandle.value = handle;
    initialDimensions.value = { ...entity.value };
}

function resizeEntity(coordinates) {
    const deltaX = coordinates.x - initialDimensions.value.x;
    const deltaY = coordinates.y - initialDimensions.value.y;

    let newX = initialDimensions.value.x;
    let newY = initialDimensions.value.y;
    let newW = initialDimensions.value.w;
    let newH = initialDimensions.value.h;

    switch (resizingHandle.value) {
        case 'top-left':
            newX = Math.min(coordinates.x, initialDimensions.value.x + initialDimensions.value.w - 1);
            newY = Math.min(coordinates.y, initialDimensions.value.y + initialDimensions.value.h - 1);
            newW = Math.max(initialDimensions.value.w - deltaX, 1);
            newH = Math.max(initialDimensions.value.h - deltaY, 1);
            break;
        case 'top-right':
            newY = Math.min(coordinates.y, initialDimensions.value.y + initialDimensions.value.h - 1);
            newW = Math.max(deltaX, 1);
            newH = Math.max(initialDimensions.value.h - deltaY, 1);
            break;
        case 'bottom-left':
            newX = Math.min(coordinates.x, initialDimensions.value.x + initialDimensions.value.w - 1);
            newW = Math.max(initialDimensions.value.w - deltaX, 1);
            newH = Math.max(deltaY, 1);
            break;
        case 'bottom-right':
            newW = Math.max(deltaX, 1);
            newH = Math.max(deltaY, 1);
            break;
    }

    if (!isColliding(newX, newY, newW, newH)) {
        entity.value.x = newX;
        entity.value.y = newY;
        entity.value.w = newW;
        entity.value.h = newH;
    }
}

function deleteEntity() {
    emit('delete', entity.value)
}

</script>

<template>
    <svg :viewBox="`-1 -1 ${width+2} ${height+2}`" width="100%" ref="SVG" @mousemove="move" @mouseup="drop" @touchmove="move" 
        @touchend="drop">
        <!-- GRID COORDINATES -->
        <text 
            v-for="(absCord, i) in gridCoordinates.abs"
            :x="i + 0.5"
            :y="-0.3"
            text-anchor="middle"
            fill="white"
            font-size="0.5"
        >
            {{ absCord }}
        </text>
        <text 
            v-for="(absCord, i) in gridCoordinates.abs"
            :x="i + 0.5"
            :y="gridSize.h+0.7"
            text-anchor="middle"
            fill="white"
            font-size="0.5"
        >
            {{ absCord }}
        </text>
        <text 
            v-for="(ordCord, i) in gridCoordinates.ord"
            :x="-0.1"
            :y="i + 0.7"
            text-anchor="end"
            fill="white"
            font-size="0.5"
        >
            {{ ordCord }}
        </text>
        <text 
            v-for="(ordCord, i) in gridCoordinates.ord"
            :x="gridSize.w + 0.1"
            :y="i + 0.7"
            text-anchor="start"
            fill="white"
            font-size="0.5"
        >
            {{ ordCord }}
        </text>

        <!-- GRID RECTS -->
        <rect 
            v-for="rect in gridRects" 
            :x="rect.x"
            :y="rect.y"
            :height="1"
            :width="1"
            :stroke="config.gridStroke"
            :stroke-width="config.gridStrokeWidth"
            :fill="config.gridFill"
            @click="triggerAction(rect)"
        />

        <!-- WALLS -->
        <rect 
            v-for="wall in walls"
            :x="wall.x"
            :y="wall.y"
            :height="1"
            :width="1"
            fill="transparent"
        />

        <rect 
            v-for="placedItem in items"
            :x="placedItem.x"
            :y="placedItem.y"
            :height="placedItem.h"
            :width="placedItem.w"
            :fill="placedItem.color"
            @click="selectItem(placedItem)"
        />

        <!-- ACTIVE ENTITY -->
        <rect
            v-if="activeEntity && activeEntity.x !== undefined"
            :x="entity.x" 
            :y="entity.y" 
            :width="entity.w" 
            :height="entity.h" 
            :fill="config.entityFill" 
            @mousedown="isDown = true" 
            @touchstart="isDown = true"
            :class="{ 'selected' : isDown }"
        />

        <text 
            v-for="placedItem in [...items, entity]"
            :x="placedItem.x + placedItem.w / 2"
            :y="placedItem.y + placedItem.h / 2 + 0.2"
            :font-size="0.6"
            :fill="'black'"
            style="pointer-events: none;"
            text-anchor="middle"
        >
            <slot name="componentItem" v-bind="{placedItem}"/>
        </text>

        <!-- HANDLES -->
        <g v-if="activeEntity && activeEntity.x !== undefined">
            <rect 
                :x="entity.x - config.handleSize / 2" 
                :y="entity.y - config.handleSize / 2" 
                fill="white" 
                :height="config.handleSize" 
                :width="config.handleSize" 
                @mousedown.prevent="startResize('top-left')"
                @touchstart.prevent="startResize('top-left')"
            />
            <rect 
                :x="entity.x + entity.w - config.handleSize / 2" 
                :y="entity.y - config.handleSize / 2" 
                fill="white" 
                :height="config.handleSize" 
                :width="config.handleSize" 
                @mousedown.prevent="startResize('top-right')"
                @touchstart.prevent="startResize('top-right')"
            />
            <rect 
                :x="entity.x - config.handleSize / 2" 
                :y="entity.y + entity.h - config.handleSize / 2" 
                fill="white" 
                :height="config.handleSize" 
                :width="config.handleSize" 
                @mousedown.prevent="startResize('bottom-left')"
                @touchstart.prevent="startResize('bottom-left')"
            />
            <rect 
                :x="entity.x + entity.w - config.handleSize / 2" 
                :y="entity.y + entity.h - config.handleSize / 2" 
                fill="white" 
                :height="config.handleSize" 
                :width="config.handleSize" 
                @mousedown.prevent="startResize('bottom-right')"
                @touchstart.prevent="startResize('bottom-right')"
            />
        </g>

        <!-- BUTTONS -->
        <circle 
            v-if="activeEntity && activeEntity.x"
            :cx="entity.x + entity.w - 0.5"
            :cy="entity.y - 1"
            :r="0.5"
            :fill="'#CCCCCC66'"
            @click="deleteEntity"
            style="cursor: pointer"
        />
            <line
                v-if="activeEntity && activeEntity.x"
                :x1="entity.x + entity.w - 0.75"
                :x2="entity.x + entity.w - 0.25"
                :y1="entity.y - 0.75"
                :y2="entity.y - 1.25"
                :stroke="'white'"
                :stroke-width="0.09"
                stroke-linecap="round"
                style="pointer-events: none;"
            />
            <line
                v-if="activeEntity && activeEntity.x"
                :x2="entity.x + entity.w - 0.75"
                :x1="entity.x + entity.w - 0.25"
                :y1="entity.y - 0.75"
                :y2="entity.y - 1.25"
                :stroke="'white'"
                :stroke-width="0.09"
                stroke-linecap="round"
                style="pointer-events: none;"
            />
    </svg>
</template>

<style scoped>
svg {
    border: 1px solid grey;
}

rect {
    transition: all 0.05s ease-in-out;
}
</style>