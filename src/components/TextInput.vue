<template>
  <div class="sidebar-control">
    <div class="control-header">
      <div>
        <svgicon
          v-if="leftIcon"
          :name="leftIcon.icon"
          :width="leftIcon.width"
          :height="leftIcon.height"
          color="#FFFFFF"
          :fill="false"></svgicon>
      </div>
      <div class="control-label">
        {{ label }}
      </div>
      <div>
        <svgicon
          v-if="rightIcon"
          :name="rightIcon.icon"
          :width="rightIcon.width"
          :height="rightIcon.height"
          color="#FFFFFF"
          :fill="false"></svgicon>
      </div>
    </div>
    <div class="control-header">
      <input
        class="control-text-input"
        type="text"
        :value="value"
        @change="onInput">
      <a
        v-if="showReset"
        class="btn"
        @click="$emit('reset')">reset</a>
    </div>
  </div>
</template>

<script>
import '@/icons'

export default {
  name: 'TextInput',
  props: {
    leftIcon: {
      type: Object,
      default: () => {}
    },
    rightIcon: {
      type: Object,
      default: () => {}
    },
    label: {
      type: String,
      default: 'Label'
    },
    value: {
      type: String,
      default: ''
    },
    showReset: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    onInput ($event) {
      this.$emit('input', $event.target.value)
    }
  }
}
</script>
