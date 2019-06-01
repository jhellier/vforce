// This is a comment

<template>
  <div id="app">
    <div id="force-panel">
      <VForce/>
  </div>
  </div>
</template>

<script>
import VForce from './components/VForce.vue'

export default {
  name: 'app',
  components: {
    VForce
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

#force-panel {
  height: 50vmax;
  width: 95vmax;
  background-color: rgb(88, 86, 86);
  
}


</style>
