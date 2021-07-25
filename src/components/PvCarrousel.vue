<template lang="">
    <div>
        <button @click="movePrv">Prev</button>
        <button @click="moveNxt">next</button>
        <div class="pv_caro">
            <div class="pv_container" :style="{ transform: transformVal }">
                <slot/>
            </div>
        </div>
    </div>
</template>
<script>
export default {
  name: "PvCarrousel",
  data() {
    return {
      container_width: null,
      card_width: null,
      transformData: null,
      transformVal: "translateX(0px)",
      gap: 10,
      cardCount: null,
      overalLenght: null,
    };
  },
  mounted() {
    this.card_width = document.querySelector(".pv_card").clientWidth;
    this.overalLenght = document.querySelector(".pv_container").clientWidth;
    this.container_width = document.querySelector(".pv_caro").clientWidth;
    this.cardCount = document.querySelectorAll(".pv_card").length;
  },
  methods: {
    moveNxt() {
      if ( this.transformData >= this.extra) {
        return;
      }
      this.transformData += this.card_width + this.gap;
      this.transformVal = `translateX(-${this.transformData}px)`;
    },
    movePrv() {
      if (this.transformData <= 0) {
        return;
      }
      else if (this.transformData == this.extra) {

      }
      this.transformData -= this.card_width + this.gap;
      this.transformVal = `translateX(-${this.transformData}px)`;
    },
  },
  computed: {
    extra() {
      return Math.round(((this.overalLenght / this.container_width)  % 1) * this.overalLenght);
    },
    slidePage() {
        return Math.floor(this.overalLenght / this.container_width)
    }

  },
};
</script>
<style lang="scss">
.pv_caro {
  overflow: scroll;
  * {
    transition: all 0.4s ease;
  }
  &::-webkit-scrollbar {
    display: none;
  }
}
.pv_container {
  width: max-content;
  height: auto;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin: 15px 0;
}


</style>
