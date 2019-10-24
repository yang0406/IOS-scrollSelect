<template>
  <div id="app">
    <scroll-select/>
  </div>
</template>

<script>
import scrollSelect from './components/scrollSelect'

export default {
  name: 'App',
  components: {
    scrollSelect
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
