<template lang="">
    <div :style="rtl ? 'direction:rtl' : ''">
        <div class="pv_caro">
            <div class="pv_container" @mousedown="grabCursor" @mouseup="releasCursor" :style="{ transform: transformVal, gap: gap +'px'}" :class="grab ? 'grab' : ''">
                <slot/>
            </div>
        </div>
    </div>
</template>
<script>
const debounce = (func, delay, { leading } = {}) => {
  let timerId;

  return (...args) => {
    if (!timerId && leading) {
      func(...args);
    }
    clearTimeout(timerId);

    timerId = setTimeout(() => func(...args), delay);
  };
};
export default {
  name: "PvCarrousel",
  data() {
    return {
      container_width: null,
      card_width: null,
      transformData: 0,
      transformVal: "translateX(0px)",
      // gap: 10,
      cardCount: null,
      overalLenght: null,
      initIndex: 1,
      grabbing: false,
      arr: [],
      mouseMove: 0,
      // rewind: false,
      // rtl: false,
      // grab: false,
      // loop: false
    };
  },
  props: {
    rewind: {
      type: Boolean,
      default: false,
    },
    rtl: {
      type: Boolean,
      default: false,
    },
    grab: {
      type: Boolean,
      default: false,
    },
    loop: {
      type: Boolean,
      default: false,
    },
    gap: {
      type: Number,
      default: 10,
    },
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
      if (!this.loop && this.initIndex == this.lastIndex) {
        // when there is a little of last card left (adding an extra push)!
        ++this.initIndex;
        this.transformData += this.extra;
        this.transformVal = `translateX(${this.direction}px)`;
        return;
      }
      if (this.loop && this.initIndex == this.lastIndex) {
        alert("bingo");
      }
      ++this.initIndex;
      this.transformData += this.card_width + this.gap;
      this.transformVal = `translateX(${this.direction}px)`;
    },
    movePrv() {
      if (!this.rewind && this.transformData <= 0) {
        return;
      }

      if ((this.rewind && this.transformData <= 0) || this.initIndex == 1) {
        this.initIndex = this.lastIndex + 1;
        this.transformData =
          (this.lastIndex - 1) * (this.card_width + this.gap) + this.extra;
        this.transformVal = `translateX(${this.direction}px)`;
        return;
      }

      if (this.initIndex > this.lastIndex) {
        --this.initIndex;
        this.transformData -= this.extra;
        this.transformVal = `translateX(${this.direction}px)`;
        return;
      }
      --this.initIndex;
      this.transformData -= this.card_width + this.gap;
      this.transformVal = `translateX(${this.direction}px)`;
    },
    grabMove(e) {
      this.arr.push(e.clientX);
      console.log(this.arr[this.arr.length - 1] - this.arr[0]);
      // console.log(this.arr)
      // ------------------------------------------------------------------------------------
      this.mouseMove = this.arr[this.arr.length - 1] - this.arr[0]; // to check the direction of grabbing
      if (this.mouseMove > 0 && this.transformData !== 0) {
        if (this.initIndex > this.lastIndex) {
          --this.initIndex;
          this.transformData -= this.extra;
          this.transformVal = `translateX(${this.direction}px)`;
          return;
        }
        this.transformData -= this.card_width + this.gap;
        --this.initIndex;
      } else if (this.mouseMove < 0) {
        if (
          this.transformData <
          (this.lastIndex - 1) * (this.card_width + this.gap)
        ) {
          if (!this.loop && this.initIndex !== this.lastIndex) {
            this.transformData += this.card_width + this.gap;
            ++this.initIndex;
          }
          if (!this.loop && this.initIndex == this.lastIndex) {
            ++this.initIndex;
            this.transformData +=
              (1 - (this.slidesToShow % 1)) * this.card_width - this.gap;
            this.transformVal = `translateX(${this.direction}px)`;
            return;
          }
        } else {
          return;
        }
      }

      this.transformVal = `translateX(${this.direction}px)`;
      console.log(this.mouseMove);
      console.log(this.transformData);
    },
    grabCursor(e) {
      const container = document.querySelector(".pv_container");
      if (this.grab) {
        this.grabbing = !this.grabbing;
        console.log(this.grabbing);
        container.style.cursor = "grabbing";
        container.style.userSelect = "none";
        this.grabMove.int = e.clientX;
        container.addEventListener("mousemove", this.grabMove);
      }
    },
    releasCursor(e) {
      const container = document.querySelector(".pv_container");
      if (this.grab) {
        this.grabbing = !this.grabbing;
        console.log(this.grabbing);
        container.style.cursor = "grab";
        container.style.removeProperty("user-select");
        this.grabMove.end = e.clientX;
        container.removeEventListener("mousemove", this.grabMove);
        this.arr.splice(0, this.arr.length);
        this.mouseMove = 0;
      }
    },
  },
  computed: {
    slidePage() {
      // think of it as dots in carousel
      return (
        this.overalLenght / (this.slidesToShow * (this.card_width - this.gap))
      );
    },
    slidesToShow() {
      // how many slides can be seen in a viewport
      return this.container_width / (this.card_width + this.gap);
    },
    extra() {
      return (1 - (this.slidesToShow % 1)) * (this.card_width - this.gap);
    },
    lastIndex() {
      return this.cardCount - Math.floor(this.slidesToShow);
    },
    direction() {
      if (this.transformData < 0) {
        this.transformData = 0;
      }
      if (this.rtl) {
        return this.transformData;
      } else {
        return this.transformData * -1;
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
  // gap: 10px;
  margin: 15px 0;
}

.dot_container {
  display: flex;
  gap: 10px;
}
.dot {
  width: 20px;
  height: 20px;
  border-radius: 100px;
  background: #eee;
}
</style>
