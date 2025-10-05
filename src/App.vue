<template>
  <div id="app">
    <quark-resource msg="Welcome to Your Vue.js App"></quark-resource>
  </div>
</template>

<script>
import QuarkResource from './components/QuarkResource.vue'

export default {
  name: 'App',
  components: {
    QuarkResource
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  height: 100%;
}
</style>
