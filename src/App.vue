<template>
  <div id="app">
    <MosaicChart />
  </div>
</template>

<script>
import MosaicChart from './components/MosaicChart.vue'
export default {
  name: 'App',
  components: {
    MosaicChart
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  background-color: #323236 ;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
