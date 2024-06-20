<script setup>
</script>

<template>
    <header>
        
      ICON
      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/docs">Docs</RouterLink>
      </nav>
  </header>
</template>

<style scoped>
header {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  padding: 12px;
  display: flex;
  flex-direction: row;
  gap: 12px;
  justify-content: space-between;
}

</style>