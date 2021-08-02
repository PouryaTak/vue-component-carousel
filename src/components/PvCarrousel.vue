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
      initIndex: 1,
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
      if (this.initIndex > this.lastIndex) {
        return;
      }
      if (this.initIndex == this.lastIndex) {
        ++this.initIndex;
        this.transformData +=
          (1 - (this.slidesToShow % 1)) * this.card_width - this.gap;
        this.transformVal = `translateX(-${this.transformData}px)`;
        return;
      }
      ++this.initIndex;
      this.transformData += this.card_width + this.gap;
      this.transformVal = `translateX(-${this.transformData}px)`;
    },
    movePrv() {
      if (this.transformData <= 0) {
        return;
      }
      if (this.initIndex > this.lastIndex) {
        --this.initIndex;
        this.transformData -=
          (1 - (this.slidesToShow % 1)) * this.card_width - this.gap;
        this.transformVal = `translateX(-${this.transformData}px)`;
        return;
      }
      --this.initIndex;
      this.transformData -= this.card_width + this.gap;
      this.transformVal = `translateX(-${this.transformData}px)`;
    },
  },
  computed: {
    slidePage() {
      return (
        this.overalLenght / (this.slidesToShow * (this.card_width - this.gap))
      );
    },
    slidesToShow() {
      return this.container_width / (this.card_width + this.gap);
    },
    extra() {
      return (1 - (this.slidesToShow % 1)) * this.card_width - this.gap;
    },
    lastIndex() {
      return this.cardCount - Math.floor(this.slidesToShow);
    },

  },
};
</script>
<style lang="scss">
body {
  margin: 0;
}
.pv_caro {
  overflow: scroll;
  * {
    transition: all 0.3s ease;
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
