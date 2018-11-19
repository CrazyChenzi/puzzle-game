<template>
  <div id="app">
    <puzzleGame />
  </div>
</template>

<script>
import puzzleGame from './components/puzzleGame.vue'

export default {
  name: 'app',
  components: {
    puzzleGame
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
