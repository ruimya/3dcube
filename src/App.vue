<template>
  <div id="app">
    <Cube />
  </div>
</template>

<script>
import Cube from "./components/Cube.vue";
export default {
  name: "App",
  components: { Cube },
};
</script>

<style lang="css">
* {
  margin: 0;
  padding: 0;
}
#app {
  display: flex;
  width: 100vw;
  height: 100vh;
}
</style>
