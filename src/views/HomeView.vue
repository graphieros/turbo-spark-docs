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
    description: 'component',
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
    <template #componentItem="{ placedItem }">
      {{ placedItem.icon }}
    </template>
  </GridPlan>
  </main>
</template>
