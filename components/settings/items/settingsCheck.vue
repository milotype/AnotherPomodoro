<template>
  <BaseSettingsItem :settings-key="settingsKey" :disabled="disabled" :custom-set-function="customSetFunction">
    <template #content-action="{ settingsValue, update }">
      <Toggle :value="settingsValue" :disabled="disabled" @input="newValue => update(newValue)" />
    </template>
  </BaseSettingsItem>
</template>

<script>
import settingsInputMixin from '@/assets/mixins/settings/settingsItemBase'
import Toggle from '@/components/base/toggle.vue'

export default {
  components: {
    Toggle,
    BaseSettingsItem: () => import(/* webpackMode: "eager" */ '~/components/settings/resolvedSettingsItem.vue')
  },

  mixins: [settingsInputMixin],

  props: {
    settingsKey: {
      type: Array,
      default: () => []
    },

    customSetFunction: {
      type: Function,
      default: undefined
    }
  }
}
</script>
