<template>
  <div class="h-transfer">
    <div class="h-transfer-source">
      <slot name="sourceHeader">
        <div v-if="option && option.ltHeadText" class="h-transfer-header">{{ option.ltHeadText }}</div>
      </slot>
      <div v-if="option.filterable && (sources.length || ltSearchText)" class="h-transfer-filter">
        <Search v-model="ltSearchText" position="front" :placeholder="option.placeholder" />
      </div>
      <div class="h-transfer-list" :style="transferListStyle">
        <div v-for="op in sources" :key="op[key]" class="h-transfer-item">
          <Checkbox v-model="ltChecked" :value="op[key]" :checked="false">
            <slot name="item" :option="op" type="source">
              <template v-if="option && option.render">{{ option.render(op) }}</template>
              <template v-else>{{ op.text }}</template>
            </slot>
          </Checkbox>
        </div>
        <div v-if="sources.length === 0" class="h-transfer-item text-center">无数据</div>
      </div>
    </div>

    <div class="h-transfer-switch">
      <button class="h-btn h-btn-s" type="button" @click="move(-1)">
        <template v-if="option && option.ltText">{{ option.ltText }}</template>
        <i v-else-if="option && option.ltIcon" :class="option.ltIcon"></i>
        <i v-else class="h-icon-left"></i>
      </button>
      <button class="h-btn h-btn-s" type="button" @click="move(1)">
        <template v-if="option && option.rtText">{{ option.rtText }}</template>
        <i v-else-if="option && option.rtIcon" :class="option.rtIcon"></i>
        <i v-else class="h-icon-right"></i>
      </button>
    </div>

    <div class="h-transfer-target">
      <slot name="targetHeader">
        <div v-if="option && option.rtHeadText" class="h-transfer-header">{{ option.rtHeadText }}</div>
      </slot>
      <div v-if="option.filterable && (targets.length || rtSearchText)" class="h-transfer-filter">
        <Search v-model="rtSearchText" position="front" :placeholder="option.placeholder" />
      </div>
      <div class="h-transfer-list" :style="transferListStyle">
        <div v-for="op in targets" :key="op[key]" class="h-transfer-item">
          <label>
            <Checkbox v-model="rtChecked" :value="op[key]">
              <slot name="item" :option="op" type="target">
                <template v-if="option && option.render">{{ option.render(op) }}</template>
                <template v-else>{{ op.text }}</template>
              </slot>
            </Checkbox>
          </label>
        </div>
        <div v-if="targets.length === 0" class="h-transfer-item text-center">无数据</div>
      </div>
    </div>
  </div>
</template>

<script>
import Checkbox from 'heyui/components/checkbox';
import Search from 'heyui/components/search';

export default {
  name: 'HTransfer',
  components: { Checkbox, Search },
  emits: ['transfer', 'input', 'change', 'update:modelValue', 'init'],
  props: {
    modelValue: {
      type: Array,
      default: () => []
    },
    datas: {
      type: Array,
      default: () => []
    },
    keyName: {
      type: String,
      default: 'key'
    },
    height: Number,
    option: {
      type: Object,
      default: () => ({})
    }
  },
  data() {
    return {
      ltChecked: [],
      rtChecked: [],
      ltSearchText: '',
      rtSearchText: '',
      key: this.keyName || 'key'
    };
  },
  computed: {
    transferListStyle() {
      let param = {};
      if (this.height) {
        param['height'] = `${this.height}px`;
      }
      return param;
    },
    sources() {
      let value = this.modelValue || [];
      let key = this.keyName || 'key';
      let result = this.datas.filter(d => value.indexOf(d[key]) == -1);
      if (this.ltSearchText && this.ltSearchText.trim()) {
        return result.filter(d => d.text.indexOf(this.ltSearchText.trim()) != -1);
      }
      return result;
    },
    targets() {
      let value = this.modelValue || [];
      let key = this.keyName || 'key';
      let result = this.datas.filter(d => value.indexOf(d[key]) != -1);
      if (this.rtSearchText && this.rtSearchText.trim()) {
        return result.filter(d => d.text.indexOf(this.rtSearchText.trim()) != -1);
      }
      return result;
    }
  },
  mounted() {
    this.$emit('init', this.sources, this.targets);
  },
  methods: {
    move(direction) {
      this.$emit('transfer', direction, this.sources, this.targets);
      let value = this.modelValue ? [...this.modelValue] : [];
      if (direction === 1 && this.ltChecked.length > 0) {
        this.rtSearchText = null;
        value.push(...this.ltChecked);
        this.ltChecked.length = 0;
      } else if (direction === -1 && this.rtChecked.length > 0) {
        this.ltSearchText = null;
        this.rtChecked.forEach(d => {
          value.splice(value.indexOf(d), 1);
        });
        this.rtChecked.length = 0;
      }
      this.$emit('input', value);
      this.$emit('change', value);
      this.$emit('update:modelValue', value);
    }
  }
};
</script>
