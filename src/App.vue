<template>
  <div id="app">
    <!-- <div id = "box">

    </div> -->
    <Map />
  </div>
</template>

<script>
import Map from "./components/Map.vue";

export default {
  name: "app",
  components: {
    Map
  }
};
</script>

<style>
body { margin: 0; }
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
}
#box{
  height: 300px;
  background: red;
}
</style>
