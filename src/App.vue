<template>
  <div id="app">
    <PvCarrousel>
      <div class="pv_card" v-for="i in 10" :key="i">{{i}}</div>
    </PvCarrousel>
  </div>
</template>

<script>
import PvCarrousel from './components/PvCarrousel.vue';
export default {
  name: 'App',
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
