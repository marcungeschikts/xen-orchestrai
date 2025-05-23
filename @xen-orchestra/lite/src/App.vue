<template>
  <div v-if="!route.meta.hasStoryNav && !xenApiStore.isConnected">
    <AppLogin />
  </div>
  <div v-else class="layout">
    <AppHeader v-if="uiStore.hasUi" />
    <div class="container">
      <AppNavigation v-if="uiStore.hasUi" />
      <main class="main" :class="{ 'no-ui': !uiStore.hasUi }">
        <RouterView />
      </main>
    </div>
    <VtsTooltipList />
  </div>
  <ModalList />
</template>

<script lang="ts" setup>
import AppHeader from '@/components/AppHeader.vue'
import AppLogin from '@/components/AppLogin.vue'
import AppNavigation from '@/components/AppNavigation.vue'
import ModalList from '@/components/ui/modals/ModalList.vue'
import { useUnreachableHosts } from '@/composables/unreachable-hosts.composable'
import { usePoolStore } from '@/stores/xen-api/pool.store'
import { useXenApiStore } from '@/stores/xen-api.store'
import VtsTooltipList from '@core/components/tooltip-list/VtsTooltipList.vue'
import { useChartTheme } from '@core/composables/chart-theme.composable'
import { useUiStore } from '@core/stores/ui.store'
import { useActiveElement, useMagicKeys, whenever } from '@vueuse/core'
import { logicAnd } from '@vueuse/math'
import { getActivePinia } from 'pinia'
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRoute } from 'vue-router'

const route = useRoute()

const xenApiStore = useXenApiStore()

const { pool } = usePoolStore().subscribe()

// workaround
// since this commit https://github.com/vatesfr/xen-orchestra/commit/ac2f4e9f32beee27ce4d14ad0d4ce7d9c51a1d82
// useUiStore is unable to find the pinia instance itself.
const pinia = getActivePinia()
const uiStore = useUiStore(pinia)
useChartTheme(pinia)
// end workaround

if (import.meta.env.DEV) {
  const { locale } = useI18n()
  const activeElement = useActiveElement()
  const { D, L } = useMagicKeys()

  const canToggle = computed(() => {
    if (activeElement.value == null) {
      return true
    }

    return !['INPUT', 'TEXTAREA'].includes(activeElement.value.tagName)
  })

  whenever(logicAnd(D, canToggle), () => (uiStore.colorMode = uiStore.colorMode === 'dark' ? 'light' : 'dark'))

  whenever(logicAnd(L, canToggle), () => (locale.value = locale.value === 'en' ? 'fr' : 'en'))
}

whenever(
  () => pool.value?.$ref,
  poolRef => {
    xenApiStore.getXapi().startWatching(poolRef)
  }
)

useUnreachableHosts()
</script>

<style lang="postcss" scoped>
.layout {
  display: flex;
  flex-direction: column;
  height: 100dvh;
}

.container {
  display: flex;
}

.main {
  overflow: auto;
  flex: 1;
  height: calc(100vh - 5.5rem);
  background-color: var(--color-neutral-background-secondary);

  &.no-ui {
    height: 100vh;
  }
}
</style>
