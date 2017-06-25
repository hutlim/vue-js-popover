<template>
  <div v-if="visible"
       :class="className"
       :style="style"
       :data-popover="name"
       ref="dropdown"
       @click.stop>
    <slot/>
  </div>
</template>

<script>
import { events } from './bus'

const pointerSize = 10
const directions = {
  left:   [-1, 0],
  right:  [1, 0],
  top:    [0, 1],
  bottom: [0, -1]
}

export default {
  name: 'Popover',
  props: {
    name: {
      type: String,
      required: true
    },
    width: {
      type: Number,
      default: 180
    },
    pointer: {
      type: Boolean,
      default: true
    },
    event: {
      type: String,
      default: 'click'
    }
  },
  data () {
    return {
      visible: false,
      positionClass: '',
      position: {
        left: 0,
        top: 0
      }
    }
  },
  mounted () {
    events.$on(this.showEventName, this.showEventListener)
    events.$on(this.hideEventName, this.hideEventListener)
  },
  computed: {
    showEventName () {
      return `show:${this.event}`
    },
    hideEventName () {
      return `hide:${this.event}`
    },
    className () {
      return [
        'vue-popover',
        this.pointer && this.positionClass
      ]
    },
    style () {
      return {
        width: `${this.width}px`,
        ...this.position
      }
    }
  },
  methods: {
    showEventListener (event) {
      if (this.visible) {
        events.$emit(this.hideEventName)
        return
      }

      this.$nextTick(() => {
        let { target, name, position } = event

        if (name === this.name) {
          let direction = directions[position]

          this.positionClass = `dropdown-position-${position}`
          this.visible = true

          this.$nextTick(() => {
            let position = this
              .getDrodownPosition(target, this.$refs.dropdown, direction)

            this.position = {
              left: `${position.left}px`,
              top: `${position.top}px`
            }

            this.$emit('show', { ...event, position })
          })
        }
      })
    },

    hideEventListener (event) {
      if (this.visible) {
        this.visible = false
        this.$emit('hide', event)
      }
    },

    getDrodownPosition (target, dropdown, direction) {
      let trRect = target.getBoundingClientRect()
      let ddRect = dropdown.getBoundingClientRect()

      let shiftX = ddRect.width - trRect.width
      let shiftY = 0.5 * (ddRect.height + trRect.height)

      let centerX = trRect.left - 0.5 * (ddRect.width - trRect.width) + window.scrollX
      let centerY = trRect.bottom - shiftY + window.scrollY

      let positionX = 0.5 * direction[0] * (ddRect.width + trRect.width)
      let positionY = direction[1] * shiftY

      if (this.pointer) {
        positionX += direction[0] * pointerSize
        positionY += direction[1] * pointerSize
      }

      return {
        left: centerX + positionX,
        top: centerY - positionY
      }
    }
  }
}
</script>

<style lang="scss">
$pointer-size: 10px;

.vue-popover {
  display: block;
  position: absolute;
  background: #fff;

  box-shadow: 0px 4px 20px 0px rgba(52, 73, 94, 0.2);

  padding: 5px;
  border-radius: 5px;

  z-index: 998;

  &:before {
    display: block;
    position: absolute;
    width: 0;
    height: 0;
    content: '';
  }

  &.dropdown-position-bottom:before {
    border-left: $pointer-size solid transparent;
    border-right: $pointer-size solid transparent;
    border-bottom: $pointer-size solid #fff;
    top: -$pointer-size;
    left: calc(50% - #{$pointer-size});
    filter: drop-shadow(0px -2px 2px rgba(52, 73, 94, 0.1));
  }

  &.dropdown-position-top:before {
    border-left: $pointer-size solid transparent;
    border-right: $pointer-size solid transparent;
    border-top: $pointer-size solid #fff;
    bottom: -$pointer-size;
    left: calc(50% - #{$pointer-size});
    filter: drop-shadow(0px 2px 2px rgba(52, 73, 94, 0.1));
  }

  &.dropdown-position-left:before {
    border-top: $pointer-size solid transparent;
    border-bottom: $pointer-size solid transparent;
    border-left: $pointer-size solid #fff;
    right: -$pointer-size;
    top: calc(50% - #{$pointer-size});
    filter: drop-shadow(2px 0px 2px rgba(52, 73, 94, 0.1));
  }

  &.dropdown-position-right:before {
    border-top: $pointer-size solid transparent;
    border-bottom: $pointer-size solid transparent;
    border-right: $pointer-size solid #fff;
    left: -$pointer-size;
    top: calc(50% - #{$pointer-size});
    filter: drop-shadow(-2px 0px 2px rgba(52, 73, 94, 0.1));
  }
}
</style>