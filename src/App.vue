<template>
  <div id="app">
    <Library />
  </div>
</template>

<script>
import Library from './components/Library.vue'

export default {
  name: 'App',
  components: {
    Library
  }
}
</script>

<style>
#app {
  font-family: Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 0;
}
</style>
