<template>
  <button
    class="zealot-button"
    :class="{ [`icon-${iconposition}`]: true }"
    @click="$emit('click')"
  >
    <z-icon class="icon loading" name="loading" v-if="loading"></z-icon>
    <z-icon class="icon" v-if="icon && !loading" :name="icon"></z-icon>
    <div class="zealot-button-content">
      <slot></slot>
    </div>
  </button>
</template>

<script>
import Icon from "../icon";
export default {
  components: {
    "z-icon": Icon
  },
  name: "zealotButton",
  props: {
    icon: {},
    loading: {
      type: Boolean,
      default: false
    },
    iconposition: {
      type: String,
      default: "left",
      validator(value) {
        return value === "left" || value === "right";
      }
    }
  },
  methods: {}
};
</script>

<style lang="scss" scoped>
@import "var";
.zealot-button {
  font-size: $font-size;
  height: $button-height;
  padding: 0 1em;
  font: inherit;
  border-radius: $border-radius;
  border: 1px solid $border-color;
  background: $button-bg;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  vertical-align: middle;
  cursor: pointer;
  &:hover {
    border-color: $border-color-hover;
  }
  &:active {
    background-color: $button-active-bg;
  }
  &:focus {
    outline: none;
  }
  > .icon {
    order: 1;
    margin-right: 0.1em;
    margin-left: 0;
    fill: #636363;
  }
  > .zealot-button-content {
    order: 2;
    vertical-align: middle;
    display: inline-flex;
    justify-content: center;
    align-items: center;
    font-family: arial, sans-serif;
  }
  &.icon-right {
    > .icon {
      order: 2;
      margin-left: 0.1em;
      margin-right: 0;
    }
    > .zealot-button-content {
      order: 1;
    }
  }
  .loading {
    @include spin;
    fill: #acacac;
  }
  & + & {
    margin-left: 4px;
  }
}
</style>
