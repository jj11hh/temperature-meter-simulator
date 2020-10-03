<template>
  <div id="app">
    <DigitalThermometerPanel style="float: left"/>
    <img src="./assets/pt100-bridge.svg" alt="pt100">
  </div>
</template>

<script>
import DigitalThermometerPanel from './components/DigitalThermometerPanel.vue'

export default {
  name: 'App',
  components: {
    DigitalThermometerPanel,
  },
  methods: {
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
