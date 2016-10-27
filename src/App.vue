<template>
  <div id="app">
    <div>
      Points are <span>{{ JSON.stringify(points) }}</span>.
    </div>
    <test-canvas :points="points" />
    <!--<img src="./assets/logo.png">-->
    <!--<hello></hello>-->
  </div>
</template>

<script>
import Hello from './components/Hello'
import TestCanvas from './TestCanvas'

export default {
  name: 'app',
  data: function () {
    return {
      points: [
        { x: 382, y: 344 },
        { x: 133, y: 342 },
        { x: 212, y: 76 }
      ]
    }
  },
  components: {
    Hello, TestCanvas
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
