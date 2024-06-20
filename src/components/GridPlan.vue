<script setup>
import { computed, ref } from 'vue';

const props = defineProps({
    /**
     * Already placed elements on the grid (walls, other entities...)
     * @example
     * [
     *  { x: 1, y: 1, w: 2, h: 3 },
     *  { x: 1, y: 4, w: 2, h: 3 },
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
     * { x: 1, y: 1, w: 1, h: 1 }
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
                h: 20
            }
        }
    }
})

const emit = defineEmits(['change'])

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

const walls = ref(convertElementsToIndividualWalls(props.items))

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

const entity = ref(computeEntityCoordinates())

const isDown = ref(false);
const resizingHandle = ref(null);
const initialDimensions = ref({ x: 0, y: 0, w: 0, h: 0 });

function getSVGCoords(evt, svg) {
    const pt = svg.createSVGPoint();
    pt.x = evt.clientX;
    pt.y = evt.clientY;

    const svgMatrix = svg.getScreenCTM();
    const transformedPt = pt.matrixTransform(svgMatrix.inverse());

    return {
        x: transformedPt.x,
        y: transformedPt.y
    };
}

function move(e) {
    if (!isDown.value) return;

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

function drop() {
    isDown.value = false;
    resizingHandle.value = null;
    emit('change', entity.value)
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
</script>

<template>
    <svg :viewBox="`0 0 ${width} ${height}`" width="100%" ref="SVG" @mousemove="move" @mouseup="drop" @mouseleave="drop">
        <!-- GRID -->
        <rect 
            v-for="rect in gridRects" 
            :x="rect.x"
            :y="rect.y"
            :height="1"
            :width="1"
            :stroke="'white'"
            stroke-width="0.01"
        />

        <!-- WALLS -->
        <rect 
            v-for="wall in walls"
            :x="wall.x"
            :y="wall.y"
            :height="1"
            :width="1"
            fill="grey"
        />

        <!-- ACTIVE ENTITY -->
        <rect 
            :x="entity.x" 
            :y="entity.y" 
            :width="entity.w" 
            :height="entity.h" 
            fill="indigo" 
            @mousedown="isDown = true" 
        />

        <!-- HANDLES -->
        <rect 
            :x="entity.x - 0.15" 
            :y="entity.y - 0.15" 
            fill="white" 
            :height="0.3" 
            :width="0.3" 
            @mousedown.prevent="startResize('top-left')"
        />
        <rect 
            :x="entity.x + entity.w - 0.15" 
            :y="entity.y - 0.15" 
            fill="white" 
            :height="0.3" 
            :width="0.3" 
            @mousedown.prevent="startResize('top-right')"
        />
        <rect 
            :x="entity.x - 0.15" 
            :y="entity.y + entity.h - 0.15" 
            fill="white" 
            :height="0.3" 
            :width="0.3" 
            @mousedown.prevent="startResize('bottom-left')"
        />
        <rect 
            :x="entity.x + entity.w - 0.15" 
            :y="entity.y + entity.h - 0.15" 
            fill="white" 
            :height="0.3" 
            :width="0.3" 
            @mousedown.prevent="startResize('bottom-right')"
        />
    </svg>
</template>

<style>
svg {
    border: 1px solid grey;
}
</style>