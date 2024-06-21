<script setup >
import { ref } from "vue"
import TheWelcome from '../components/TheWelcome.vue'
import GridPlan from '@/components/GridPlan.vue';

const items = ref([

])

function recordChange(entity) {
  console.log(entity)
}

const entity = ref({
})

const step = ref(0);

function selectItem(item) {
  console.log(item)
  entity.value = item;
  step.value += 1;
}

function triggerAction(rect) {
  // could be a popup menu
  items.value.push({
    ...rect,
    h: 1,
    w: 1,
    id: items.value.length + 1,
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
  }
])

const activeType = ref(availableTypes.value[0])

function setActiveType(t) {
  activeType.value = t;
}

</script>

<template>
  <main>
    <div class="flex flex-row gap-6 p-8">
      <div v-for="t in availableTypes" :style="`background:${t.color}`" class="py-2 px-6 text-black cursor-pointer" @click="setActiveType(t)">
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
    >
    <template #componentItem="{ placedItem }">
      {{ placedItem.icon }}
    </template>
  </GridPlan>
  </main>
</template>
