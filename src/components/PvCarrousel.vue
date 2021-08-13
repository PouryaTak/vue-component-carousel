<template lang="">
    <div :style="rtl ? 'direction:rtl' : ''">
      <div style="display:flex; margin-bottom: 20px; justify-content:center; background:#eee">
        <div style="margin: 0 10px">
          <label for="grab">grab</label>
        <input type="checkbox" name="grab" id="grab" @change="grab = !grab">
        </div>
        <div style="margin: 0 10px">
          <label for="rewind">rewind</label>
        <input type="checkbox" name="rewind" id="rewind" @change="rewind = !rewind">
        </div>
        <div style="margin: 0 10px">
          <label for="rtl">rtl</label>
        <input type="checkbox" name="rtl" id="rtl" @change="rtl = !rtl">
        </div>
      </div>
        <button @click="movePrv">Prev</button>
        <button @click="moveNxt">next</button>
        <div class="pv_caro">
            <div class="pv_container" @mousedown="grabCursor" @mouseup="releasCursor" :style="{ transform: transformVal}" :class="grab ? 'grab' : ''">
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
      rewind: false,
      rtl: false,
      grab: false,
      grabbing: false,
      mousePos: {
        int: null,
        end: null
      }
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
      if (!this.rewind && this.initIndex > this.lastIndex) {
        return;
      }
      if (this.rewind && this.initIndex > this.lastIndex) {
        this.initIndex = 1;
        this.transformData = 0;
        this.transformVal = `translateX(${this.direction}px)`;
        return;
      }
      if (this.initIndex == this.lastIndex) {
        ++this.initIndex;
        this.transformData +=
          (1 - (this.slidesToShow % 1)) * this.card_width - this.gap;
        this.transformVal = `translateX(${this.direction}px)`;
        return;
      }
      ++this.initIndex;
      this.transformData += this.card_width + this.gap;
      this.transformVal = `translateX(${this.direction}px)`;
    },
    movePrv() {
      if (!this.rewind && this.transformData <= 0) {
        return;
      }

      if (this.rewind && this.transformData <= 0) {
        this.initIndex = this.lastIndex + 1;
        this.transformData =
          (this.lastIndex - 1) * (this.card_width + this.gap) +
          (1 - (this.slidesToShow % 1)) * (this.card_width - this.gap);
        this.transformVal = `translateX(${this.direction}px)`;
        return;
      }

      if (this.initIndex > this.lastIndex) {
        --this.initIndex;
        this.transformData -=
          (1 - (this.slidesToShow % 1)) * this.card_width - this.gap;
        this.transformVal = `translateX(${this.direction}px)`;
        return;
      }
      --this.initIndex;
      this.transformData -= this.card_width + this.gap;
      this.transformVal = `translateX(${this.direction}px)`;
    },
    grabCursor(e) {
      const container = document.querySelector(".pv_container");
      if (this.grab) {
        this.grabbing = !this.grabbing;
        console.log(this.grabbing);
        container.style.cursor = "grabbing";
        container.style.userSelect = "none";
        this.mousePos.int = e.clientX

      }
    },
    releasCursor(e) {
      const container = document.querySelector(".pv_container");
      if (this.grab) {
        this.grabbing = !this.grabbing;
        console.log(this.grabbing);
        container.style.cursor = "grab";
        container.style.removeProperty("user-select");
        this.mousePos.end = e.clientX
        console.log(this.mousePos)
      }
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
    direction() {
      if (!this.rtl) {
        return this.transformData * -1;
      } else {
        return this.transformData;
      }
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
.grab {
  cursor: grab;
}
.grabbing {
  cursor: grabbing;
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
