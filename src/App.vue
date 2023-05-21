<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <AlphaCompent></AlphaCompent>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import AlphaCompent from './components/color/IndexComponent.vue';

export default {
  name: 'App',
  components: {
    // HelloWorld,
    AlphaCompent
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
