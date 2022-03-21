<template>
  <Minesweeper :rows="10" :bombs="30"/>
</template>

<script>
  import 'animate.css';
  import Minesweeper from './components/Minesweeper.vue';

  export default {
    name: 'App',
    
    components: {
      Minesweeper
    }
  }
</script>

<style lang="scss">
  *, *:before, *:after {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  body {
    width: 100vw;
    height: 100vh;
    padding: 1rem;
    display: flex;
    align-items: center;
    justify-content: center;
  }
</style>
