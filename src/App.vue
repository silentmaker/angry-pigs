<template>
  <div id="app">
    <Game />
  </div>
</template>

<script>
import Game from './components/Game.vue'

export default {
  name: 'app',
  components: {
    Game
  }
}
</script>

<style>
body {
  padding: 0;
  margin: 0;
}
.hidden {
  display: none;
}
#app {
  font-family: Helvetica, Arial, sans-serif;
  color: #2c3e50;
}
</style>
