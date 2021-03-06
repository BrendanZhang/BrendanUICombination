<template>
  <div
    class="z-slides"
    @mouseenter="onMouseEnter"
    @mouseleave="onMouseLeave"
    @touchstart="onTouchStart"
    @touchmove="onTouchMove"
    @touchend="onTouchEnd"
  >
    <div class="z-slides-window" ref="window">
      <div class="z-slides-wrapper">
        <slot></slot>
      </div>
    </div>
    <div class="z-slides-dots">
      <span @click="onClickPrev" class="prev">
        <z-icon name="left"></z-icon>
      </span>
      <span
        v-for="n in childrenLength"
        :key="n"
        :data-index="n-1"
        :class="{active: selectedIndex === n-1}"
        @click="select(n-1)"
      >{{n}}</span>
      <span @click="onClickNext" class="next">
        <z-icon name="right"></z-icon>
      </span>
    </div>
  </div>
</template>
<script>
import Icon from "../icon";
export default {
  name: "zealotSlides",
  components: {
    "z-icon": Icon
  },
  props: {
    selected: {
      type: String
    },
    autoPlay: {
      type: Boolean,
      default: true
    },
    autoPlayDelay: {
      type: Number,
      default: 3000
    }
  },
  data() {
    return {
      childrenLength: 0,
      lastSelectedIndex: undefined,
      timerId: undefined,
      startTouch: undefined
    };
  },
  mounted() {
    this.updateChildren();
    if (this.autoPlay) {
      this.toggleAutoPlay();
    }
    this.childrenLength = this.items.length;
  },
  updated() {
    this.updateChildren();
  },
  beforeDestroy() {
    this.pause();
  },
  computed: {
    selectedIndex() {
      let index = this.names.indexOf(this.selected);
      return index === -1 ? 0 : index;
    },
    names() {
      return this.items.map(vm => vm.name);
    },
    items() {
      return this.$children.filter(
        vm => vm.$options.name === "zealotSlides-items"
      );
    }
  },
  methods: {
    onClickPrev() {
      this.select(this.selectedIndex - 1);
    },
    onClickNext() {
      this.select(this.selectedIndex + 1);
    },
    onMouseEnter() {
      this.pause();
    },
    onMouseLeave() {
      this.toggleAutoPlay();
    },
    onTouchStart(e) {
      this.pause();
      if (e.touches.length > 1) {
        return;
      }
      this.startTouch = e.touches[0];
    },
    onTouchMove() {},
    onTouchEnd(e) {
      let endTouch = e.changedTouches[0];
      let { clientX: x1, clientY: y1 } = this.startTouch;
      let { clientX: x2, clientY: y2 } = endTouch;
      let horizon = Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));
      let deltaY = Math.abs(y2 - y1);
      let slope = horizon / deltaY;
      if (slope > 2) {
        if (x2 > x1) {
          this.select(this.selectedIndex - 1);
        } else {
          this.select(this.selectedIndex + 1);
        }
      }
      this.$nextTick(() => {
        this.toggleAutoPlay();
      });
    },
    getSelected() {
      let first = this.items[0];
      return this.selected || first.name;
    },
    pause() {
      window.clearTimeout(this.timerId);
      this.timerId = undefined;
    },
    updateChildren() {
      let selected = this.getSelected();
      this.items.forEach(vm => {
        let reverse =
          this.selectedIndex > this.lastSelectedIndex ? false : true;
        if (this.timerId) {
          if (
            this.lastSelectedIndex === this.items.length - 1 &&
            this.selectedIndex === 0
          ) {
            reverse = false;
          }
          if (
            this.lastSelectedIndex === 0 &&
            this.selectedIndex === this.items.length - 1
          ) {
            reverse = true;
          }
        }
        vm.reverse = reverse;
        this.$nextTick(() => {
          vm.selected = selected;
        });
      });
    },
    toggleAutoPlay() {
      if (this.timerId) {
        return;
      }
      let run = () => {
        let index = this.names.indexOf(this.getSelected());
        let newIndex = index + 1;
        this.select(newIndex);
        this.timerId = setTimeout(run, this.autoPlayDelay);
      };
      this.timerId = setTimeout(run, this.autoPlayDelay);
    },
    select(newIndex) {
      this.lastSelectedIndex = this.selectedIndex;
      if (newIndex === -1) {
        newIndex = this.names.length - 1;
      }
      if (newIndex === this.names.length) {
        newIndex = 0;
      }
      this.$emit("update:selected", this.names[newIndex]);
    }
  }
};
</script>
<style lang="scss" scoped>
.z-slides {
  border: 1px solid grey;
  &-window {
    overflow: hidden;
  }
  &-wrapper {
    position: relative;
  }
  &-dots {
    padding: 8px 0;
    display: flex;
    justify-content: center;
    align-items: center;
    > span {
      width: 20px;
      height: 20px;
      display: inline-flex;
      border-radius: 50%;
      justify-content: center;
      align-items: center;
      background: #ddd;
      margin: 0 8px;
      font-size: 12px;
      &:hover {
        cursor: pointer;
      }
      &.active {
        background: #1b1b1b;
        color: #ddd;
        &:hover {
          cursor: default;
        }
      }
    }
  }
}
</style>
