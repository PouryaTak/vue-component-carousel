<template>
  <div id="app">
    <PvCarrousel>
      <div class="pv_card">1</div>
      <div class="pv_card">2</div>
      <div class="pv_card">3</div>
      <div class="pv_card">4</div>
      <div class="pv_card">5</div>
      <div class="pv_card">6</div>
      <div class="pv_card">7</div>
      <div class="pv_card">8</div>
      <div class="pv_card">9</div>
      <div class="pv_card">10</div>
    </PvCarrousel>
  </div>
</template>

<script>
import PvCarrousel from "./components/PvCarrousel.vue";
export default {
  name: "App",
  components: {
    PvCarrousel,
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: #2c3e50;
  margin-top: 60px;
  overflow: hidden;
}

.pv_card {
  width: 300px;
  height: 150px;
  border-radius: 15px;
  background: #999999;
  display: grid;
  place-items: center;
}
</style>
