<!-- src/App.vue -->
<template>
  <div id="app">
    <ThreeEditor />
  </div>
</template>

<script>
import ThreeEditor from './components/ThreeEditor.vue';

export default {
  name: 'App',
  components: {
    ThreeEditor
  }
};
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
