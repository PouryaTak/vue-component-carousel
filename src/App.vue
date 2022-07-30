<template>
  <div id="app">
    <div style="display: flex; margin-bottom: 20px; justify-content: center; background: #eee">
      <div style="margin: 0 10px">
        <label for="grab">Grab</label>
        <input type="checkbox" name="grab" id="grab" @change=";(grab = !grab), refreshComponent()" :checked="grab" />
      </div>
      <div style="margin: 0 10px">
        <label for="rewind">Rewind</label>
        <input type="checkbox" name="rewind" id="rewind" @change="rewind = !rewind" :checked="rewind" />
      </div>
      <div style="margin: 0 10px">
        <label for="rtl">RTL</label>
        <input type="checkbox" name="rtl" id="rtl" @change="rtl = !rtl" :checked="rtl" />
      </div>
      <div style="margin: 0 10px">
        <label for="loop">Loop</label>
        <input type="checkbox" name="loop" id="loop" @change="loop = !loop" :checked="loop" />
      </div>
      <div style="margin: 0 10px">
        <label for="chunk">Chunk</label>
        <input type="checkbox" name="chunk" id="chunk" @change="chunk = !chunk" :checked="chunk" />
      </div>
      <!-- <div style="margin: 0 10px">
        <label for="chunk">Vertical</label>
        <input type="checkbox" name="vertical" id="vertical" @change="vertical = !vertical" :checked="vertical" />
      </div> -->
      <div style="margin: 0 10px">
        <label for="gap">Gap</label>
        <input type="number" name="gap" id="gap" v-model.number="gap" style="width: 50px; margin: 0 5px" />
      </div>
      <div style="margin: 0 10px">
        <label for="autoPlay">AutoPlay(sec)</label>
        <input type="number" name="autoPlay" id="autoPlay" v-model.number="autoPlay" style="width: 50px; margin: 0 5px" @change="refreshComponent"/>
      </div>
    </div>
    <div style="width:80%;margin:0 auto">
      <button @click="$refs.pvcarousel.movePrv()">Prev</button>
      <button @click="$refs.pvcarousel.moveNxt()">next</button>
      <div style="height:600px;">
        <PvCarrousel ref="pvcarousel" :grab="grab" :rewind="rewind" :rtl="rtl" :loop="loop" :gap="gap" :chunk="chunk" :key="key" :vertical="vertical" :autoPlay="autoPlay">
          <div class="pv_card" v-for="i in 10" :key="i" :style="`filter:hue-rotate(${i * 20}deg)`">
          <a :href="`#${i}`">{{i}}</a>
          </div>
        </PvCarrousel>
      </div>
    </div>
  </div>
</template>

<script>
import PvCarrousel from '@/components/PvCarrousel.vue'
export default {
  name: 'App',
  components: {
    PvCarrousel
  },
  data () {
    return {
      grab: true,
      rewind: false,
      rtl: false,
      loop: true,
      chunk: false,
      gap: 20,
      key: 0,
      vertical: false,
      autoPlay: 0
    }
  },
  watch: {
    loop (val) {
      this.refreshComponent()
    }
  },
  methods: {
    refreshComponent () {
      ++this.key
    }
  }
}
</script>

<style lang="scss">
body {
  margin: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: #2c3e50;
  padding-top: 60px;
  overflow: hidden;
  max-height: 100vh;
}

.pv_card {
  width: 300px;
  height: 150px;
  border-radius: 15px;
  background: #ff7474;
  display: grid;
  place-items: center;
}
.pv_highlight {
  box-shadow: 0 0 0px 10px rgb(192, 192, 192);
}
</style>
