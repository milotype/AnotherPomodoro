<template>
  <div class="grid grid-flow-row select-option-group gap-3" :class="[Object.keys(values).length > 3 ? 'md:grid-flow-row md:grid-cols-3' : 'md:grid-flow-col md:auto-cols-auto']">
    <OptionControl
      v-for="(item, key) in values"
      :key="key"
      class="min-w-0"
      :active="key === selected"
      :translation-key="translationKey"
      :translation-subkey="key"
      :custom-title="overrideText.title == null ? '' : (overrideText.title[key] ? overrideText.title[key] : null)"
      :custom-description="overrideText.description == null ? '' : (overrideText.description[key] ? overrideText.description[key] : null)"
      @click="select(key)"
    />
  </div>
</template>

<script>
export default {
  components: {
    OptionControl: () => import(/* webpackMode: "eager" */ '@/components/base/option.vue')
  },

  props: {
    values: {
      type: Object,
      required: true,
      default: () => {}
    },

    translationKey: {
      type: String,
      default: ''
    },

    selected: {
      type: String,
      default: ''
    },

    overrideText: {
      type: Object,
      default: () => {
        return {
          title: {},
          description: {}
        }
      }
    }
  },

  // data () {
  //   return {
  //     selected: null
  //   }
  // },

  methods: {
    select (key) {
      // this.selected = key
      this.$emit('input', key)
    }
  }
}
</script>
