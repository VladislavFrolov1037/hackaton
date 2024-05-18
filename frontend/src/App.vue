<script setup>
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header>

    <div class="wrapper">
      <RouterLink to="/">Home</RouterLink>
    </div>
  </header>

  <RouterView />
</template>

<style scoped>



</style>
