<template>
  <section v-show="processedValue" class="fixed z-40 w-full h-full p-0 md:p-4 md:max-w-screen-sm">
    <div class="flex flex-col h-full overflow-hidden bg-white rounded-none shadow-lg md:rounded-lg dark:bg-gray-900 dark:text-gray-50">
      <h1 class="px-4 mt-4 mb-2 text-xl font-bold uppercase">
        <span>{{ $i18n.t('settings.heading') }}</span>
        <Button
          :aria-label="$i18n.t('settings.buttons.close')"
          default
          circle
          :importance="3"
          class="float-right -mt-2 -mr-2"
          tabindex="0"
          @click="processedValue = false"
        >
          <CloseIcon :aria-label="$i18n.t('settings.buttons.close')" />
        </Button>
      </h1>
      <div class="flex-grow px-4 py-2 overflow-y-auto">
        <div class="w-full">
          <Transition tag="div" name="tab-transition" mode="out-in" class="relative w-full overflow-hidden">
            <div v-if="activeTab === 1" :key="1" class="settings-tab">
              <OptionGroup
                :values="$languages"
                :selected="settingsStore.lang"
                :override-text="{ title: $languages, description: null }"
                @input="(event) => { settingsStore.lang = event }"
              />
              <Divider />
              <SettingsCheck :settings-key="['adaptiveTicking', 'enabled']" />
              <SettingsCheck :settings-key="['timerControls', 'enableKeyboardShortcuts']" />
              <Divider />
              <SettingsCheck :settings-key="['permissions', 'audio']" />
              <SettingsCheck
                :settings-key="['permissions', 'notifications']"
                :set-value-on-change="false"
                :disabled="notificationsEnabled === false"
                :custom-set-function="changeNotificationSettings"
              />

              <Divider />

              <SettingsCheck :settings-key="['tasks', 'enabled']" />
              <SettingsText
                :settings-key="['tasks', 'maxActiveTasks']"
                :min="1"
                numeric
                :disabled="!settingsStore.tasks.enabled"
              />
              <SettingsCheck :settings-key="['tasks', 'removeCompletedTasks']" :disabled="!settingsStore.tasks.enabled" />

              <Divider />

              <div v-text="$i18n.t('settings.manage.heading')" />
              <div class="-mt-2 text-sm leading-snug text-black text-opacity-75 dark:text-gray-50 dark:text-opacity-75" v-text="$i18n.t('settings.manage.description')" />
              <div class="grid grid-flow-col grid-cols-2 gap-2 mt-1">
                <ExportButton />
                <ImportButton />
              </div>

              <Divider />

              <SettingsCheck :settings-key="['reset']" />
            </div>

            <div v-else-if="activeTab === 2" :key="2" class="settings-tab">
              <SettingsText :settings-key="['schedule', 'longPauseInterval']" :min="1" numeric />
              <Divider />

              <SettingsOptions
                :settings-key="['schedule', 'lengths']"
                :custom-value="settingsStore.getActiveSchedulePreset"
                override-translation-key="timerpreset"
                :values="timerPresets"
                :set-value-on-change="false"
                :custom-set-function="(v) => { settingsStore.applyPreset(v) }"
              />
              <SettingsTime :settings-key="['schedule', 'lengths', 'work']" :min-ms="5000" />
              <SettingsTime :settings-key="['schedule', 'lengths', 'shortpause']" :min-ms="5000" />
              <SettingsTime :settings-key="['schedule', 'lengths', 'longpause']" :min-ms="5000" />
              <div class="flex flex-row items-center px-3 py-4 space-x-2 rounded-lg ring-inset ring ring-primary bg-primary/20 dark:bg-gray-700 dark:text-gray-100">
                <InfoIcon />
                <span v-text="$i18n.t('settings.scheduleMinTime')" />
              </div>
            </div>

            <div v-else-if="activeTab === 3" :key="3" class="settings-tab">
              <SettingsCheck :settings-key="['visuals', 'darkMode']" />
              <Divider />
              <SettingsOptions :settings-key="['currentTimer']" :values="{traditional: 'traditional', approximate: 'approximate', percentage: 'percentage'}" />
              <Divider />
              <SettingsCheck :settings-key="['schedule', 'visibility', 'enabled']" />
              <SettingsCheck :settings-key="['schedule', 'visibility', 'showSectionType']" :disabled="!settingsStore.schedule.visibility.enabled" />
              <SettingsText
                :settings-key="['schedule', 'numScheduleEntries']"
                :min="3"
                :max="10"
                :disabled="!settingsStore.schedule.visibility.enabled"
                numeric
              />
              <Divider />
              <SettingsCheck :settings-key="['performance', 'showProgressBar']" />
              <SettingsCheck :settings-key="['pageTitle', 'useTickEmoji']" />
              <!-- TODO Audio volume control -->
            </div>

            <div v-else-if="activeTab === 4" :key="4" class="settings-tab">
              <div class="flex flex-col items-center">
                <img src="/favicon.svg" width="64" height="64" class="inline-block p-2 mb-1 bg-red-200 rounded-lg">
                <div>
                  <div class="inline-block text-2xl font-bold">
                    AnotherPomodoro
                  </div>
                  <sup class="text-base" v-text="version" />
                </div>
                <div v-text="$i18n.t('settings.about.madeby')" />
                <div class="flex flex-col items-center justify-center mt-8">
                  <!-- Support links -->
                  <div class="flex flex-row flex-wrap justify-center gap-2 mt-3 text-center">
                    <Button
                      :importance="1"
                      dark
                      link
                      href="https://www.github.com/Hanziness/AnotherPomodoro?utm_source=AnotherPomodoro&utm_medium=web&utm_content=settings"
                      inner-class="flex flex-row items-center gap-1 text-slate-50 text-gray-50"
                      bg-class="bg-slate-900 dark:bg-slate-700"
                    >
                      <AboutGithub />
                      <span v-text="$i18n.t('settings.about.source')" />
                    </Button>
                    <Button
                      :importance="1"
                      link
                      href="https://www.buymeacoffee.com/imreg?utm_source=AnotherPomodoro&utm_medium=web&utm_content=settings"
                      inner-class="flex flex-row items-center gap-1 text-black"
                      bg-class="bg-yellow-300"
                    >
                      <AboutSupport />
                      <span v-text="$i18n.t('settings.about.support')" />
                    </Button>
                  </div>
                  <!-- Share links -->
                  <div class="my-2" v-text="$i18n.t('settings.about.share')" />
                  <div class="flex flex-row items-center space-x-2 text-sm">
                    <Button
                      link
                      circle
                      dark
                      :importance="1"
                      href="https://twitter.com/AnotherPomodoro?utm_source=AnotherPomodoro&utm_medium=web&utm_content=settings"
                      bg-class="bg-[#1da1f2]"
                      inner-class="!p-4 text-slate-50"
                    >
                      <AboutTwitter :aria-label="$i18n.t('index.alt.links.share.twitter')" size="24" />
                    </Button>
                    <Button
                      link
                      circle
                      dark
                      :importance="1"
                      href="http://www.facebook.com/share.php?u=https://another-pomodoro.netlify.app"
                      bg-class="bg-[#1877f2]"
                      inner-class="!p-4 text-slate-50"
                    >
                      <AboutFacebook :aria-label="$i18n.t('index.alt.links.share.facebook')" size="24" class="translate-x-[-1px]" />
                    </Button>
                    <Button
                      link
                      circle
                      dark
                      :importance="1"
                      href="https://reddit.com/submit?url=https://another-pomodoro.netlify.app"
                      bg-class="bg-[#ff4500]"
                      inner-class="!p-4 text-slate-50"
                    >
                      <AboutReddit :aria-label="$i18n.t('index.alt.links.share.reddit')" size="24" />
                    </Button>
                  </div>
                </div>
              </div>
            </div>
          </Transition>
        </div>
      </div>

      <!-- Tab bar -->
      <div class="flex flex-row flex-none h-20 p-4">
        <TabHeader :active="activeTab === 1" :text="$i18n.t('settings.tabs.main')" @click="activeTab = 1">
          <template #icon>
            <TabIconGeneral role="presentation" />
          </template>
        </TabHeader>
        <TabHeader :active="activeTab === 2" :text="$i18n.t('settings.tabs.timer')" @click="activeTab = 2">
          <template #icon>
            <TabIconSchedule role="presentation" />
          </template>
        </TabHeader>
        <TabHeader :active="activeTab === 3" :text="$i18n.t('settings.tabs.display')" @click="activeTab = 3">
          <template #icon>
            <TabIconVisuals role="presentation" />
          </template>
        </TabHeader>
        <TabHeader :active="activeTab === 4" :text="$i18n.t('settings.tabs.about')" @click="activeTab = 4">
          <template #icon>
            <TabIconAbout role="presentation" />
          </template>
        </TabHeader>
      </div>
    </div>
  </section>
</template>

<script>
import { XIcon, AdjustmentsIcon, AlarmIcon, ArtboardIcon, InfoCircleIcon, BrandGithubIcon, CoffeeIcon, BrandTwitterIcon, BrandFacebookIcon, BrandRedditIcon } from 'vue-tabler-icons'

import { mapActions, mapState, mapStores } from 'pinia'
import OptionGroup from '@/components/base/optionGroup.vue'
import TabHeader from '@/components/settings/panel/tabHeader.vue'
import ExportButton from '@/components/settings/exportButton.vue'
import ImportButton from '@/components/settings/importButton.vue'

import presetTimers from '@/assets/settings/timerPresets'
import { useSettings } from '~/stores/settings'
import { useNotifications } from '~/stores/notifications'
import { useMain } from '~/stores'

export default {
  name: 'SettingsPanel',
  components: {
    // UiOverlay: () => import('@/components/base/overlay.vue'),
    Button: () => import(/* webpackChunkName: "uibase" */ '@/components/base/button.vue'),
    SettingsCheck: () => import(/* webpackMode: "eager" */ '@/components/settings/items/settingsCheck.vue'),
    SettingsText: () => import(/* webpackMode: "eager" */ '@/components/settings/items/settingsText.vue'),
    SettingsTime: () => import(/* webpackMode: "eager" */ '@/components/settings/items/settingsTime.vue'),
    SettingsOptions: () => import(/* webpackMode: "eager" */ '@/components/settings/items/settingsOptions.vue'),
    Divider: () => import(/* webpackMode: "eager" */ '@/components/base/divider.vue'),
    OptionGroup,
    TabHeader,
    ExportButton,
    ImportButton,
    CloseIcon: XIcon,
    // ResetIcon: RefreshAlertIcon,
    TabIconGeneral: AdjustmentsIcon,
    TabIconSchedule: AlarmIcon,
    TabIconVisuals: ArtboardIcon,
    TabIconAbout: InfoCircleIcon,
    InfoIcon: InfoCircleIcon,
    AboutGithub: BrandGithubIcon,
    AboutSupport: CoffeeIcon,
    AboutTwitter: BrandTwitterIcon,
    AboutFacebook: BrandFacebookIcon,
    AboutReddit: BrandRedditIcon
  },
  props: {
    value: {
      type: Boolean,
      default: false
    }
  },

  data () {
    return {
      activeTab: 1,
      resetConfirm: false,
      timerPresets: presetTimers
    }
  },

  computed: {
    ...mapStores(useSettings),
    ...mapState(useMain, ['version']),
    ...mapState(useNotifications, {
      notificationsEnabled: 'enabled'
    }),

    processedValue: {
      get () { return this.value },
      set (newValue) { this.$emit('input', newValue) }
    },

    notificationPermission: {
      get () {
        return Notification ? (Notification.permission === 'granted' && this.settingsStore.permissions.notifications) : false
      },
      set (newValue) {
        if (Notification.permission === 'default') {
          const thisRef = this
          this.$notification.requestPermission().then((newValue) => {
            if (newValue === 'granted') {
              thisRef.settingsStore.SET({ key: ['permissions', 'notifications'], value: true })
            } else {
              thisRef.notificationsBlocked = true
              thisRef.settingsStore.SET({ key: ['permissions', 'notifications'], value: false })
            }

            thisRef.updateNotificationsEnabled(newValue === 'granted')
          })
        } else if (Notification.permission === 'granted') {
          this.settingsStore.SET({ key: ['permissions', 'notifications'], value: newValue })
        }
      }
    }
  },

  methods: {
    ...mapActions(useNotifications, {
      updateNotificationsEnabled: 'updateEnabled'
    }),

    changeNotificationSettings (newValue) {
      this.notificationPermission = newValue
    }
  }
}
</script>

<style lang="scss" scoped>
div.settings-tab {
  @apply grid grid-cols-1 gap-2;
}

// ===== TAB TRANSITIONS =====
.tab-transition-enter-active,
.tab-transition-leave-active {
  transition: transform 0.2s ease-out, opacity 0.2s ease-out;
  // transition: opacity 0.5s ease-out;
  position: relative;
}

.tab-transition-enter {
  transform: translateY(10px);
  opacity: 0;
}

.tab-transition-leave-to {
  transform: translateY(-10px);
  opacity: 0;
}
</style>
