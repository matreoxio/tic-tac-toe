<script setup lang="ts">
import GameBoard from './components/GameBoard.vue';

</script>


<template>
  <div id="app">
    <h1>Tic-Tac-Toe</h1>
    <GameBoard />
  </div>
</template>

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