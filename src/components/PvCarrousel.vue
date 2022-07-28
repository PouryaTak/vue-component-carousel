<template lang="">
  <div :id="containerId" draggable="false" :style="rtl ? 'direction:rtl' : ''">
    <div ref="pv_caro" draggable="false" class="pv_caro" :class="{ pv_caro_vertical: vertical }" :style="preventTouchScorll ? 'overflow:hidden' : ''">
      <div
        ref="pv_container"
        draggable="false"
        class="pv_container"
        :style="{ transform: transform_data, gap: gap + 'px', transition: `transform ${transition_speed}s ${transition_timming_function}` }"
        :class="{ pv_grab: grab, pv_container_vertical: vertical }"
      >
        <slot />
      </div>
    </div>
    <div ref="pv_dots" class="pv_dots" v-if="dots">
      <div ref="pv_dot" class="pv_dot" v-for="i in pages" :key="i" @click="dotFunc(i)" :class="activeDot == i ? 'pv_opacity' : ''"></div>
    </div>
  </div>
</template>
<script>
//! -------------------------------------------------------------------- there is an issue with gaps larger than card's width

// ? ToDo fix vertical movement function

export default {
  name: 'PvCarrousel',
  data () {
    return {
      pv_card: undefined,
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
      // activeDot: 1,
      allDuplicatedCards: null,
      transition_speed: 0.3,
      transition_timming_function: 'ease',
      offset: 0,
      touchstartX: null,
      touchstartY: null,
      touchendX: null,
      touchendY: null,
      swipeThershold: 50,
      movement_list: [],
      grab_control: {
        init_move: 0,
        move: 0,
        move_store: 0,
        snap: true
      },
      autoPlay_id: null,
      debouncer_id: null
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
    },
    preventTouchScorll: {
      type: Boolean,
      default: true
    },
    highLightItem: {
      type: Number,
      default: 2
    },
    hitglightClass: {
      type: String,
      default: 'pv_highlight'
    },
    autoPlay: {
      type: Number,
      default: 0
    }
  },
  mounted () {
    this.transformation_value = this.offset
    this.initialSetup()
    this.transformation_value += (this.width_of_card + this.gap) * (this.startFrom - 1)
    this.directingTheMovment()
    this.addTouchEventListener()
    this.addDragEventListner()
    if (this.autoPlay) {
      this.autoPlayProcess()
    }
  },
  computed: {
    containerId () {
      return 'pv_' + Math.random().toString(36).substring(2, 5) + Math.random().toString(36).substring(2, 5)
    },
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
      return extra === this.width_of_card ? extra + this.gap : extra

      // return ((this.width_of_cards_container/this.width_of_viewport_container) % 1) * this.width_of_viewport_container
    },
    lastIndex () {
      return this.number_of_all_cards - Math.floor(this.number_of_cards_in_chunk)
    },
    activeDot () {
      let activeDot
      for (let i = 0; i <= this.pages; i++) {
        if (this.initIndex >= i * this.number_of_cards_in_chunk) {
          activeDot = i
        } else {
          break
        }
      }
      return activeDot === this.pages ? activeDot : activeDot + 1
      // indicats which dot is active
    }
  },
  methods: {
    initialSetup () {
      if (process.browser) {
        this.pv_card = document.querySelector(`#${this.containerId} .pv_card`)
        if (this.pv_card) {
          this.width_of_card = this.pv_card.offsetWidth // changed from clientWidth to offsetWidth to include boarders
          this.cards_container = this.$refs.pv_container
          this.width_of_cards_container = this.cards_container.clientWidth
          this.width_of_viewport_container = this.$refs.pv_caro.clientWidth
          this.all_cards = document.querySelectorAll(`#${this.containerId} .pv_card`)
          this.number_of_all_cards = this.all_cards.length
          // this.pages = Math.round(this.number_of_chunks)
          this.pages = Math.ceil(this.number_of_chunks)
          if (this.loop) {
            this.all_cards.forEach((card) => {
              this.cards_container.appendChild(card.cloneNode(true))
            })
          }
          if (this.highLightItem && this.loop) {
            this.allDuplicatedCards = document.querySelectorAll(`#${this.containerId} .pv_card`)
            this.allDuplicatedCards[this.highLightItem - 1 + (this.initIndex - 1)].classList.add(this.hitglightClass)
          }
          if (this.loop) {
            for (let i = 1; i <= this.number_of_all_cards - 1; i++) {
              this.movement_list[0] = 0
              this.movement_list[i] = this.gap + this.width_of_card + this.movement_list[i - 1]
            }
          } else {
            for (let i = 1; i <= this.lastIndex - 1; i++) {
              this.movement_list[0] = 0
              this.movement_list[i] = this.gap + this.width_of_card + this.movement_list[i - 1]
            }
            this.movement_list[this.lastIndex] = this.movement_list[this.lastIndex - 1] + this.extra_width
          }
        }
      }
    },
    autoPlayInit () {
      this.autoPlay_id = setInterval(() => {
        this.moveNxt('autoPlay')
      }, this.autoPlay * 1000)
    },
    autoPlayProcess () {
      this.autoPlayInit()
      this.cards_container.addEventListener('mouseenter', () => {
        clearInterval(this.autoPlay_id)
        this.autoPlay_id = null
      })
      this.cards_container.addEventListener('mouseleave', () => {
        this.autoPlayInit()
      })
    },
    directingTheMovment () {
      let actionDirection
      // setting the action of the movement (eg. next, prev)
      if (this.transformation_value < this.offset) {
        actionDirection = this.offset
      } else if (this.rtl) {
        actionDirection = this.transformation_value
      } else {
        actionDirection = this.transformation_value * -1
      }
      // setting the direction of the movement in Horizontal or Vertical
      if (this.vertical) {
        this.transform_data = `translateY(${actionDirection}px)`
      } else {
        this.transform_data = `translateX(${actionDirection}px)`
      }
    },

    moveNxt (orgin = '') {
      if (this.autoPlay && orgin !== 'autoPlay') {
        clearInterval(this.autoPlay_id)
        clearTimeout(this.debouncer_id)
        this.debouncer_id = setTimeout(() => {
          clearInterval(this.autoPlay_id)
          this.autoPlay_id = null
          this.autoPlayInit()
        }, 2000)
      }
      if (this.chunk) {
        this.dotFunc(this.activeDot === this.pages ? (this.rewind ? 1 : this.pages) : ++this.activeDot)
        return
      }
      if (!this.loop && !this.rewind && this.initIndex > this.lastIndex) {
        return
      }
      if (this.rewind && this.initIndex > this.lastIndex) {
        this.initIndex = 1
        this.transformation_value = this.offset
        this.directingTheMovment()
        return
      }
      if (!this.loop && this.initIndex === this.lastIndex) {
        // when there is a little of last card lefts, (adding an extra_width push)!
        ++this.initIndex
        this.transformation_value += this.extra_width
        this.directingTheMovment()
        return
      }

      // ?------------------------------------- loop --------------------------------------------------

      if (this.loop && this.initIndex === this.number_of_all_cards) {
        ++this.initIndex
        this.transformation_value += this.width_of_card + this.gap
        this.directingTheMovment()
        return
      }

      // ?------------------------------------- loop end --------------------------------------------------
      this.transition_speed = 0.3
      ++this.initIndex
      this.transformation_value += this.width_of_card + this.gap
      this.directingTheMovment()
    },

    movePrv (orgin = '') {
      if (this.autoPlay && orgin !== 'autoPlay') {
        clearInterval(this.autoPlay_id)
        clearTimeout(this.debouncer_id)
        this.debouncer_id = setTimeout(() => {
          this.autoPlayInit()
        }, 2000)
        console.log('deb')
      }
      if (this.chunk) {
        this.dotFunc(this.activeDot === 1 ? (this.rewind ? this.pages : 1) : --this.activeDot)
        return
      }
      if (this.loop && this.initIndex === 1) {
        this.transition_speed = 0
        this.initIndex = this.number_of_all_cards + 1
        this.transformation_value = (this.width_of_card + this.gap) * (this.number_of_all_cards + 1)
        this.directingTheMovment()
      }

      if (!this.rewind && this.transformation_value <= this.offset) {
        return
      }

      if ((this.rewind && this.transformation_value <= this.offset) || this.initIndex === 1) {
        this.initIndex = this.lastIndex + 1
        this.transformation_value = (this.lastIndex - 1) * (this.width_of_card + this.gap) + this.extra_width
        this.directingTheMovment()
        return
      }

      if (!this.loop && this.initIndex > this.lastIndex) {
        --this.initIndex
        this.transformation_value -= this.extra_width
        this.directingTheMovment()
        return
      }
      --this.initIndex
      this.transformation_value -= this.width_of_card + this.gap
      this.directingTheMovment()
    },
    dotFunc (num) {
      this.activeDot = num
      if (num === this.pages) {
        this.initIndex = this.lastIndex + 1
        this.transformation_value = (this.width_of_card + this.gap) * (this.lastIndex - 1) + this.extra_width
        this.directingTheMovment()
        return
      }
      this.initIndex = (num - 1) * Math.floor(this.number_of_cards_in_chunk) + 1
      this.transformation_value = (this.width_of_card + this.gap) * (this.initIndex - 1)
      this.directingTheMovment()
    },
    calcSwipeDirection () {
      if (this.touchendX < this.touchstartX && Math.abs(this.touchstartX - this.touchendX) >= this.swipeThershold) {
        console.log('Left')
        // adding functionality for swipe left
        if (this.rtl) {
          this.movePrv()
        } else {
          this.moveNxt()
        }
      }

      if (this.touchendX > this.touchstartX && Math.abs(this.touchstartX - this.touchendX) >= this.swipeThershold) {
        console.log('Right')
        if (this.rtl) {
          this.moveNxt()
        } else {
          this.movePrv()
        }
      }

      if (this.touchendY < this.touchstartY && Math.abs(this.touchstartY - this.touchendY) >= this.swipeThershold) {
        console.log('Up')
      }

      if (this.touchendY > this.touchstartY && Math.abs(this.touchstartY - this.touchendY) >= this.swipeThershold) {
        console.log('Down')
      }

      if (this.touchendY === this.touchstartY) {
        console.log('Tap')
      }
    },
    calcMoveX (event) {
      this.grab_control.move = event.pageX - this.grab_control.init_move + this.grab_control.move_store
      console.log(this.grab_control.move)
      this.transformation_value = this.grab_control.move
    },
    returnNearstNumber (num, arr, limiter) {
      if (limiter === undefined) {
        limiter = (arr[1] - arr[0]) / 2
      }
      let closest = arr[0]
      for (const i of arr) {
        if (num + limiter < i) {
          break
        }
        closest = i
      }
      return closest
    },
    addTouchEventListener () {
      if (!this.grab) {
        return
      }
      this.cards_container.addEventListener(
        'touchstart',
        (event) => {
          this.touchstartX = event.changedTouches[0].screenX
          this.touchstartY = event.changedTouches[0].screenY
        },
        false
      )

      this.cards_container.addEventListener(
        'touchend',
        (event) => {
          this.touchendX = event.changedTouches[0].screenX
          this.touchendY = event.changedTouches[0].screenY
          // console.log(this.touchendX, this.touchstartX, this.swipeThershold, this.touchendY, this.touchstartY)
          this.calcSwipeDirection()
        },
        false
      )
    },
    addDragEventListner () {
      if (!this.grab) {
        return
      }
      this.cards_container.addEventListener('mousedown', (event) => {
        this.cards_container.style.cursor = 'grabbing'
        this.grab_control.init_move = event.pageX
        document.addEventListener('mousemove', this.calcMoveX)
        this.directingTheMovment()
      })

      this.cards_container.addEventListener('mouseup', (event) => {
        this.cards_container.style.cursor = 'grab'
        document.removeEventListener('mousemove', this.calcMoveX)
        if (this.grab_control.snap) {
          this.transformation_value = this.returnNearstNumber(this.transformation_value, this.movement_list)
        }
        this.grab_control.move_store = this.transformation_value
        this.directingTheMovment()
      })
      this.cards_container.addEventListener('touchend', (event) => {})

      this.cards_container.addEventListener('mouseleave', (event) => {
        this.cards_container.style.cursor = 'grab'
        // document.removeEventListener('mousemove', this.calcMoveX)
        // if (this.grab_control.snap) {
        //   this.transformation_value = this.returnNearstNumber(this.transformation_value, this.movement_list)
        // }
        // this.grab_control.move_store = this.transformation_value
        // this.directingTheMovment()
      })

      // this.cards_container.addEventListener('dragstart', event => {
      //   // replace draging shadow with trasparent div
      //   const elem = document.createElement('div')
      //   elem.id = 'drag-ghost'
      //   this.cards_container.appendChild(elem)
      //   event.dataTransfer.setDragImage(elem, 0, 0)

      //   this.cards_container.style.cursor = 'progress'
      //   // this.cards_container.style.userSelect = 'none'
      //   // console.log(event)

      //   this.touchstartX = event.screenX
      //   this.touchstartY = event.screenY
      // })
      // this.cards_container.addEventListener('dragend', event => {
      //   const ghost = document.getElementById('drag-ghost')
      //   if (ghost.parentNode) {
      //     ghost.parentNode.removeChild(ghost)
      //   }
      //   this.cards_container.style.cursor = 'grab'
      //   this.cards_container.style.removeProperty('user-select')
      //   this.touchendX = event.screenX
      //   this.touchendY = event.screenY
      //   this.calcSwipeDirection()
      // })
    }
  },
  watch: {
    mouseMove (newval, oldval) {
      if (newval < 0) {
        console.log('moue moved!')
      }
    },
    number_of_chunks (to, from) {
      this.pages = Math.ceil(to)
    },
    initIndex (to, from) {
      if (from !== 1 && to === this.number_of_all_cards + 1) {
        setTimeout(() => {
          this.transition_speed = 0
          this.initIndex = 1
          this.transformation_value = this.offset
          this.directingTheMovment()
          // this.transition_speed = 0.3
        }, 300)
      }
      if (from === 1 && to === this.number_of_all_cards) {
        setTimeout(() => {
          this.transition_speed = 0.3
          this.transformation_value -= this.width_of_card + this.gap
          this.directingTheMovment()
        }, 5)
      }
      if (this.highLightItem && this.loop) {
        this.allDuplicatedCards[this.highLightItem - 1 + (this.initIndex - 1)].classList.add(this.hitglightClass)
        this.allDuplicatedCards[this.highLightItem - 1 + (from - 1)].classList.remove(this.hitglightClass)
      }
    }
  }
}
</script>
<style>
#drag-ghost {
  width: 0;
  height: 0;
  position: absolute;
  background: #0000;
}
.pv_caro {
  /* overflow: scroll; */
  overflow: hidden;
  width: 100%;
}
.pv_caro_vertical {
  height: 100%;
}
.pv_caro::-webkit-scrollbar {
  display: none;
}
.pv_card {
  user-select: none !important;
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
