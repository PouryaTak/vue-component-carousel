<template>
  <div id="app">
    <div style="display:flex; margin-bottom: 20px; justify-content:center; background:#eee">
        <div style="margin: 0 10px">
          <label for="grab">Grab</label>
        <input type="checkbox" name="grab" id="grab" @change="grab = !grab">
        </div>
        <div style="margin: 0 10px">
          <label for="rewind">Rewind</label>
        <input type="checkbox" name="rewind" id="rewind" @change="rewind = !rewind">
        </div>
        <div style="margin: 0 10px">
          <label for="rtl">RTL</label>
        <input type="checkbox" name="rtl" id="rtl" @change="rtl = !rtl">
        </div>
        <div style="margin: 0 10px">
          <label for="loop">Loop</label>
        <input type="checkbox" name="loop" id="loop" @change="loop = !loop">
        </div>
      </div>
        <button @click="$refs.pvcarousel.movePrv()">Prev</button>
        <button @click="$refs.pvcarousel.moveNxt()">next</button>
    <PvCarrousel ref="pvcarousel" :grab="grab" :rewind="rewind" :rtl="rtl" :loop="loop">
      <div class="pv_card" v-for="i in 10" :key="i" >{{i}}</div>
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
  data(){
    return {
      grab: false,
      rewind: false,
      rtl: false,
      loop: false,
    }
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
