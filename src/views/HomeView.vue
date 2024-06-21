<script setup >
import { ref } from "vue"
import TheWelcome from '../components/TheWelcome.vue'
import GridPlan from '@/components/GridPlan.vue';

const items = ref([

])

function recordChange(entity) {
  console.log('RECORD CHANGE')
  console.log(entity)
}

const entity = ref({
})

const step = ref(0);

function selectItem(item) {
  entity.value = item;
  step.value += 1;
}

function triggerAction(rect) {
  // could be a popup menu
  items.value.push({
    ...rect,
    h: 1,
    w: 1,
    id: `${Date.now()}-${Math.random()}`,
    ...activeType.value
  });
  selectItem(items.value.at(-1))
}

const availableTypes = ref([
  {
    color: '#BBCCCC',
    description: 'door',
    icon: 'D'
  },
  {
    color: '#8A8A8A',
    description: 'wall',
    icon: 'W'
  },
  {
    color: '#3366DD',
    description: 'server',
    icon: 'C'
  },
  {
    color: "#DD6633",
    description: "power",
    icon: 'P'
  }
])

const activeType = ref(availableTypes.value[0])

function setActiveType(t) {
  activeType.value = t;
}

function deleteItem(item) {
  items.value = items.value.filter(i => i.id !== item.id)
  entity.value = {}
  step.value += 1
}

function unselect() {
  entity.value = {}
  step.value += 1
}

</script>

<template>
  <main>
    <div class="flex flex-row gap-6 p-8">
      <div v-for="t in availableTypes" :style="`background:${t.color}; outline:${activeType.description === t.description ? '2px solid white' : 'none'}`" class="py-2 px-6 text-black cursor-pointer" @click="setActiveType(t)">
        {{ t.description }}
      </div>
    </div>
    <GridPlan 
      :key="step" 
      :items="items" 
      :active-entity="entity" 
      @change="recordChange" 
      @selectItem="selectItem"
      @triggerAction="triggerAction"
      @delete="deleteItem"
      @unselect="unselect"
    >

    <template #componentIcon="{ placedItem, maxSize }">

      <svg xmlns="http://www.w3.org/2000/svg" v-if="placedItem.description === 'server'" viewBox="0 0 24 24" stroke-width="1.5" stroke="#FFFFFF" fill="none" stroke-linecap="round" stroke-linejoin="round" :style="`width:${maxSize}px; position: absolute; top: 50%; left:50%; transform: translate(-50%,-50%) scale(0.8, 0.8)`">
        <path stroke="none" d="M0 0h24v24H0z" fill="none"/>
        <path d="M3 4m0 3a3 3 0 0 1 3 -3h12a3 3 0 0 1 3 3v2a3 3 0 0 1 -3 3h-12a3 3 0 0 1 -3 -3z" />
        <path d="M3 12m0 3a3 3 0 0 1 3 -3h12a3 3 0 0 1 3 3v2a3 3 0 0 1 -3 3h-12a3 3 0 0 1 -3 -3z" />
        <path d="M7 8l0 .01" />
        <path d="M7 16l0 .01" />
        <path d="M11 8h6" />
        <path d="M11 16h6" />
    </svg>

      <svg xmlns="http://www.w3.org/2000/svg" v-if="placedItem.description === 'wall'" viewBox="0 0 24 24" stroke-width="1.5" stroke="#FFFFFF" fill="none" stroke-linecap="round" stroke-linejoin="round" :style="`width:${maxSize}px; position: absolute; top: 50%; left:50%; transform: translate(-50%,-50%) scale(0.8, 0.8)`">
        <path stroke="none" d="M0 0h24v24H0z" fill="none"/>
        <path d="M4 4m0 2a2 2 0 0 1 2 -2h12a2 2 0 0 1 2 2v12a2 2 0 0 1 -2 2h-12a2 2 0 0 1 -2 -2z" />
        <path d="M4 8h16" />
        <path d="M20 12h-16" />
        <path d="M4 16h16" />
        <path d="M9 4v4" />
        <path d="M14 8v4" />
        <path d="M8 12v4" />
        <path d="M16 12v4" />
        <path d="M11 16v4" />
    </svg>

      <svg xmlns="http://www.w3.org/2000/svg" v-if="placedItem.description === 'door'" viewBox="0 0 24 24" stroke-width="1.5" stroke="#FFFFFF" fill="none" stroke-linecap="round" stroke-linejoin="round" :style="`width:${maxSize}px; position: absolute; top: 50%; left:50%; transform: translate(-50%,-50%) scale(0.8, 0.8)`">
        <path stroke="none" d="M0 0h24v24H0z" fill="none"/>
        <path d="M14 10m-2 0a2 2 0 1 0 4 0a2 2 0 1 0 -4 0" />
        <path d="M21 12a9 9 0 1 1 -18 0a9 9 0 0 1 18 0z" />
        <path d="M12.5 11.5l-4 4l1.5 1.5" />
        <path d="M12 15l-1.5 -1.5" />
    </svg>

      <svg xmlns="http://www.w3.org/2000/svg" v-if="placedItem.description === 'power'" viewBox="0 0 24 24" stroke-width="1.5" stroke="#FFFFFF" fill="none" stroke-linecap="round" stroke-linejoin="round" :style="`width:${maxSize}px; position: absolute; top: 50%; left:50%; transform: translate(-50%,-50%) scale(0.8, 0.8)`">
        <path stroke="none" d="M0 0h24v24H0z" fill="none"/>
        <path d="M13 3l0 7l6 0l-8 11l0 -7l-6 0l8 -11" />
    </svg>
    </template>
  </GridPlan>
  </main>
</template>