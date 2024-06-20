<script setup >
import { ref } from "vue"
import TheWelcome from '../components/TheWelcome.vue'
import GridPlan from '@/components/GridPlan.vue';

const items = [
  { x: 0, y: 0, w: 20, h: 1},
  { x: 10, y: 1, w: 1, h: 10},
  { x: 2, y: 7, w: 5, h: 2}
]

function recordChange(entity) {
  console.log(entity)
}

const entity = ref({
  x: 2,
  y: 3,
  h: 2,
  w: 2,
})

</script>

<template>
  <main>
    <GridPlan :items="items" @change="recordChange" :active-entity="entity"/>
  </main>
</template>
