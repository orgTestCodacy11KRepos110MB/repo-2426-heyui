<template>
  <div :class="taginputCls">
    <span v-for="(v, index) of values" :key="v + index">
      <span>{{ v }}</span
      ><i v-if="!readonly" class="h-icon-close-min" @click.stop="remove(index)"></i>
    </span>
    <input
      v-if="!readonly"
      v-model="tagvalue"
      type="text"
      class="h-taginput-input h-input"
      :placeholder="placeholder"
      @focus="focusing = true"
      @blur="blur"
      @keyup.enter="add"
      @keydown.delete="removeLast"
    />
  </div>
</template>
<script>
import utils from 'heyui/utils/utils';
import Locale from 'heyui/mixins/locale';
import Message from 'heyui/plugins/message';

const prefix = 'h-taginput';

export default {
  name: 'HTagInput',
  mixins: [Locale],
  emits: ['input', 'change', 'update:modelValue'],
  props: {
    readonly: {
      type: Boolean,
      default: false
    },
    noBorder: {
      type: Boolean,
      default: false
    },
    placeholder: {
      type: String
    },
    type: {
      type: String,
      default: 'Array' // String
    },
    split: {
      type: String,
      default: ','
    },
    wordlimit: {
      type: Number,
      default: 50
    },
    limit: {
      type: Number,
      default: 10000
    },
    modelValue: [Array, String, Number]
  },
  data() {
    return {
      focusing: false,
      tagvalue: ''
    };
  },
  computed: {
    taginputCls() {
      return {
        [`${prefix}`]: true,
        [`${prefix}-input-border`]: !this.noBorder && !this.readonly,
        [`${prefix}-readonly`]: this.readonly,
        focusing: this.focusing
      };
    },
    values() {
      if (this.type == 'Array') {
        return this.modelValue || [];
      } else {
        if (utils.isNull(this.modelValue) || this.modelValue === '') {
          return [];
        }
        return String(this.modelValue).split(this.split);
      }
    }
  },
  methods: {
    removeLast(event) {
      if (event.target.value === '' && this.values.length) {
        this.remove(this.values.length - 1);
      }
    },
    remove(index) {
      if (this.readonly) return;
      let value = utils.copy(this.values);
      value.splice(index, 1);
      this.setvalue(value);
    },
    add() {
      if (this.wordlimit < this.tagvalue.length) {
        Message.error(this.hlang('h.wordlimit.warn', [this.wordlimit]));
        return false;
      }
      if (this.limit <= this.values.length) {
        Message.error(this.hlang('h.taginput.limitWords'));
        return false;
      }
      if (this.readonly) return false;
      if (this.tagvalue === '') return false;
      let value = utils.copy(this.values);
      value.push(this.tagvalue);
      this.setvalue(value);
    },
    setvalue(value) {
      if (this.type == 'string') {
        if (value.length == 0) {
          value = null;
        } else {
          value = value.join(this.split);
        }
      }
      this.$emit('input', value);
      this.$emit('update:modelValue', value);
      this.$emit('change', value);
      let event = document.createEvent('CustomEvent');
      event.initCustomEvent('setvalue', true, true, value);
      this.$el.dispatchEvent(event);
      this.tagvalue = '';
    },
    blur() {
      this.add();
      this.focusing = false;
    }
  }
};
</script>
