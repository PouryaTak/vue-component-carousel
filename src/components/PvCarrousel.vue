<template lang="">
    <div :style="rtl ? 'direction:rtl' : ''">
        <div ref="pv_caro" class="pv_caro" :class="{pv_caro_vertical: vertical}">
            <div ref="pv_container" class="pv_container" @mousedown="grabCursor" @mouseup="releasCursor" :style="{ transform: transform_data, gap: gap +'px',transition: `transform ${transition_speed}s ${transition_timming_function}`}" :class="{pv_grab:grab, pv_container_vertical:vertical }">
                <slot/>
            </div>
        </div>
        <div ref="pv_dots" class="pv_dots" v-if="dots">
          <div ref="pv_dot" class="pv_dot" v-for="i in pages" :key="i" @click="dotFunc(i)" :class="dot == i ? 'pv_opacity':''"></div>
        </div>
    </div>
</template>
<script>
// const debounce = (func, delay, { leading } = {}) => {
//   let timerId;

//   return (...args) => {
//     if (!timerId && leading) {
//       func(...args);
//     }
//     clearTimeout(timerId);

//     timerId = setTimeout(() => func(...args), delay);
//   };
// };

//! -------------------------------------------------------------------- there is an issue with gaps larger than card's width

// ? ToDo Loop function

// ? ToDo grab function

export default {
  name: 'PvCarrousel',
  data () {
    return {
      width_of_viewport_container: null,
      width_of_card: null,
      transformation_value: 0,
      transform_data: 'translateX(0px)',
      all_cards: null,
      number_of_all_cards: null,
      cards_container: null,
      width_of_cards_container: null,
      initIndex: this.startFrom,
      grabbing: false,
      arr: [],
      mouseMove: 0,
      pages: null,
      dot: 1,
      transition_speed: 0.3,
      transition_timming_function: 'ease'
    }
  },
  props: {
    rewind: {
      type: Boolean,
      default: false
    },
    rtl: {
      type: Boolean,
      default: false
    },
    grab: {
      type: Boolean,
      default: false
    },
    loop: {
      type: Boolean,
      default: false
    },
    gap: {
      type: Number,
      default: 10
    },
    dots: {
      type: Boolean,
      default: true
    },
    chunk: {
      type: Boolean,
      default: false
    },
    vertical: {
      type: Boolean,
      default: false
    },
    startFrom: {
      type: Number,
      default: 1
    }
  },
  mounted () {
    if (process.browser) {
      this.width_of_card = document.querySelector('.pv_card').clientWidth
      this.cards_container = this.$refs.pv_container
      this.width_of_cards_container = this.cards_container.clientWidth
      this.width_of_viewport_container = this.$refs.pv_caro.clientWidth
      this.all_cards = document.querySelectorAll('.pv_card')
      this.number_of_all_cards = this.all_cards.length
      this.pages = Math.round(this.number_of_chunks)
      if (this.loop) {
        this.all_cards.forEach((card) => {
          this.cards_container.appendChild(card.cloneNode(true))
        })
      }
    }
    this.transformation_value += (this.width_of_card + this.gap) * (this.startFrom - 1)
    this.transform_data = `translateX(${this.direction}px)`
  },
  computed: {
    number_of_cards_in_chunk () {
      // how many cards can be seen in a viewport
      const initial_value = this.width_of_viewport_container / (this.width_of_card + this.gap)
      const total_width_of_cards = this.width_of_viewport_container - Math.floor(initial_value) * this.gap
      return total_width_of_cards / this.width_of_card
    },
    number_of_chunks () {
      // think of it as dots in carousel
      return this.width_of_cards_container / (this.number_of_cards_in_chunk * this.width_of_card + (this.number_of_cards_in_chunk - 1) * this.gap)
    },
    extra_width () {
      const extra = (1 - (this.number_of_cards_in_chunk % 1)) * this.width_of_card
      return extra == this.width_of_card ? extra + this.gap : extra

      // return ((this.width_of_cards_container/this.width_of_viewport_container) % 1) * this.width_of_viewport_container
    },
    lastIndex () {
      return this.number_of_all_cards - Math.floor(this.number_of_cards_in_chunk)
    },
    direction () {
      if (this.transformation_value < 0) {
        return 0
      }
      if (this.rtl) {
        return this.transformation_value
      } else {
        return this.transformation_value * -1
      }
    }
  },
  methods: {
    moveNxt () {
      if (this.chunk) {
        this.dotFunc(this.dot == this.pages ? (this.rewind ? 1 : this.pages) : ++this.dot)
        return
      }
      if (!this.loop && !this.rewind && this.initIndex > this.lastIndex) {
        return
      }
      if (this.rewind && this.initIndex > this.lastIndex) {
        this.initIndex = 1
        this.transformation_value = 0
        this.transform_data = `translateX(${this.direction}px)`
        return
      }
      if (!this.loop && this.initIndex == this.lastIndex) {
        // when there is a little of last card lefts, (adding an extra_width push)!
        ++this.initIndex
        this.transformation_value += this.extra_width
        this.transform_data = `translateX(${this.direction}px)`
        return
      }

      // ?------------------------------------- loop --------------------------------------------------

      if (this.loop && this.initIndex == this.number_of_all_cards) {
        ++this.initIndex
        this.transformation_value += this.width_of_card + this.gap
        this.transform_data = `translateX(${this.direction}px)`
        return
      }

      // ?------------------------------------- loop --------------------------------------------------
      this.transition_speed = 0.3
      ++this.initIndex
      this.transformation_value += this.width_of_card + this.gap
      this.transform_data = `translateX(${this.direction}px)`
    },
    movePrv () {
      if (this.chunk) {
        this.dotFunc(this.dot === 1 ? (this.rewind ? this.pages : 1) : --this.dot)
        return
      }
      if (this.loop && this.initIndex === 1) {
        this.transition_speed = 0
        this.initIndex = this.number_of_all_cards + 1
        this.transformation_value = (this.width_of_card + this.gap) * (this.number_of_all_cards + 1)
        this.transform_data = `translateX(${this.direction}px)`
      }

      if (!this.rewind && this.transformation_value <= 0) {
        return
      }

      if ((this.rewind && this.transformation_value <= 0) || this.initIndex == 1) {
        this.initIndex = this.lastIndex + 1
        this.transformation_value = (this.lastIndex - 1) * (this.width_of_card + this.gap) + this.extra_width
        this.transform_data = `translateX(${this.direction}px)`
        return
      }

      if (!this.loop && this.initIndex > this.lastIndex) {
        --this.initIndex
        this.transformation_value -= this.extra_width
        this.transform_data = `translateX(${this.direction}px)`
        return
      }
      --this.initIndex
      this.transformation_value -= this.width_of_card + this.gap
      this.transform_data = `translateX(${this.direction}px)`
    },
    grabMove (e) {
      this.arr.push(e.clientX)
      console.log(this.arr[this.arr.length - 1] - this.arr[0])
      // console.log(this.arr)
      // ------------------------------------------------------------------------------------
      this.mouseMove = this.arr[this.arr.length - 1] - this.arr[0] // to check the direction of grabbing
      if (this.mouseMove > 0 && this.transformation_value !== 0) {
        this.transformation_value -= 10
        this.transform_data = `translateX(${this.direction}px)`
      } else if (this.mouseMove < 0 && this.transformation_value <= (this.width_of_card + this.gap) * (this.lastIndex - 1) + this.extra_width) {
        this.transformation_value += 10
        this.transform_data = `translateX(${this.direction}px)`
      }
      console.log(this.mouseMove)
      console.log(this.transformation_value)
    },
    grabCursor (e) {
      const container = document.querySelector('.pv_container')
      if (this.grab) {
        this.grabbing = !this.grabbing
        console.log(this.grabbing)
        container.style.cursor = 'grabbing'
        container.style.userSelect = 'none'
        this.grabMove.int = e.clientX
        container.addEventListener('mousemove', this.grabMove)
      }
    },
    releasCursor (e) {
      const container = document.querySelector('.pv_container')
      if (this.grab) {
        this.grabbing = !this.grabbing
        console.log(this.grabbing)
        container.style.cursor = 'grab'
        container.style.removeProperty('user-select')
        this.grabMove.end = e.clientX
        container.removeEventListener('mousemove', this.grabMove)
        this.arr.splice(0, this.arr.length)
        this.mouseMove = 0
      }
    },
    dotFunc (num) {
      this.dot = num
      if (num == this.pages) {
        this.initIndex = this.lastIndex + 1
        this.transformation_value = (this.width_of_card + this.gap) * (this.lastIndex - 1) + this.extra_width
        this.transform_data = `translateX(${this.direction}px)`
        return
      }
      this.initIndex = (num - 1) * Math.floor(this.number_of_cards_in_chunk) + 1
      this.transformation_value = (this.width_of_card + this.gap) * (this.initIndex - 1)
      this.transform_data = `translateX(${this.direction}px)`
    }
  },
  watch: {
    mouseMove (newval, oldval) {
      if (newval < 0) {
        console.log('moue moved!')
      }
    },
    number_of_chunks (to, from) {
      this.pages = Math.round(to)
    },
    initIndex (to, from) {
      if (from !== 1 && to == this.number_of_all_cards + 1) {
        setTimeout(() => {
          this.transition_speed = 0
          this.initIndex = 1
          this.transformation_value = 0
          this.transform_data = `translateX(${this.direction}px)`
          // this.transition_speed = 0.3
        }, 300)
      }
      if (from == 1 && to == this.number_of_all_cards) {
        setTimeout(() => {
          this.transition_speed = 0.3
          this.transformation_value -= this.width_of_card + this.gap
          this.transform_data = `translateX(${this.direction}px)`
        }, 5)
      }
    }
  }
}
</script>
<style>
.pv_caro {
  overflow: scroll;
}
.pv_caro_vertical {
  height: 100%;
}
.pv_caro::-webkit-scrollbar {
  display: none;
}
.pv_grab {
  cursor: grab;
}
.pv_grabbing {
  cursor: grabbing;
}
.pv_container {
  width: max-content;
  height: auto;
  display: flex;
  flex-wrap: wrap;
  margin: 15px 0;
}
.pv_container_vertical {
  width: auto;
  height: max-content;
  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
}
.pv_dots {
  display: flex;
  gap: 10px;
  width: max-content;
  margin: 0 auto;
}
.pv_dot {
  width: 20px;
  height: 20px;
  border-radius: 100px;
  background: rgb(196, 196, 196);
  cursor: pointer;
  opacity: 0.5;
}
.pv_opacity {
  opacity: 1;
}
</style>
