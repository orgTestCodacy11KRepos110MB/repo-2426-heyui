<template>
  <div :class="classes" :style="styles">
    <slot></slot>
    <div v-if="type != 'flex'" class="h-row-clear"></div>
  </div>
</template>
<script>
const prefixCls = 'h-row';

const Props = {
  type: ['flex'],
  align: ['top', 'middle', 'bottom'],
  justify: ['start', 'end', 'center', 'space-around', 'space-between'],
  direction: ['row', 'row-reverse', 'column', 'column-reverse']
};

const getHalf = (width, hasRemainder) => {
  return Math.floor(width / -2) + (hasRemainder ? width % 2 : 0) + 'px';
};

export default {
  name: 'HRow',
  props: {
    type: {
      validator(value) {
        return Props.type.indexOf(value) != -1;
      }
    },
    align: {
      validator(value) {
        return Props.align.indexOf(value) != -1;
      }
    },
    justify: {
      validator(value) {
        return Props.justify.indexOf(value) != -1;
      }
    },
    direction: {
      validator(value) {
        return Props.direction.indexOf(value) != -1;
      }
    },
    space: {
      type: Number,
      default: 0
    },
    spaceX: {
      type: Number,
      default: 0
    },
    spaceY: {
      type: Number,
      default: 0
    }
  },
  computed: {
    classes() {
      return [
        {
          [`${prefixCls}`]: !this.type,
          [`${prefixCls}-${this.type}`]: !!this.type,
          [`${prefixCls}-${this.type}-${this.align}`]: !!this.align,
          [`${prefixCls}-${this.type}-${this.direction}`]: this.direction,
          [`${prefixCls}-${this.type}-${this.justify}`]: !!this.justify
        }
      ];
    },
    styles() {
      let style = {};
      if (this.space !== 0) {
        const leftTop = getHalf(this.space, true);
        const rightBottom = getHalf(this.space, false);
        style.marginLeft = leftTop;
        style.marginRight = rightBottom;
        style.marginTop = leftTop;
        style.marginBottom = rightBottom;
      }

      if (this.spaceX !== 0) {
        style.marginLeft = getHalf(this.spaceX, true);
        style.marginRight = getHalf(this.spaceX, false);
      }

      if (this.spaceY !== 0) {
        style.marginTop = getHalf(this.spaceY, true);
        style.marginBottom = getHalf(this.spaceY, false);
      }

      return style;
    }
  }
};
</script>
