<template>
  <EShopDashboard />
</template>

<script lang="ts">
import { defineComponent } from "vue";
import EShopDashboard from "./components/EShopDashboard.vue";

export default defineComponent({
  name: "App",
  components: {
    EShopDashboard,
  },
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background: linear-gradient(to right, #a8e063, #ffffff);
  height: 100vh;
  width: 100vw;
  margin: 0;
  padding: 0;
  overflow: hidden;
}
</style>
