<template>
  <div id="app">
    <Borrar/>
    <HelloWorld msg=""/>
    
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import Borrar from './components/Borrar.vue'

export default {
  name: 'App',
  components: {
    HelloWorld,
    Borrar
  }
}
</script>


<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
