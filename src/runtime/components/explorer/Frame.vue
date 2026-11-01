<script setup lang="ts">
import { useSlots, computed, useAttrs } from 'vue'
import Nav from './Nav.vue'

const props = defineProps({
  window: { type: Object, required: false },
  content: { type: null, required: false },
  chromePadding: { type: Boolean, required: false, default: true },
  frameClass: { type: String, required: false },
})

defineEmits([
  'drag:start',
  'drag:move',
  'drag:end',
  'resize:start',
  'resize:move',
  'resize:end',
])

defineOptions({ inheritAttrs: false })

const attrs = useAttrs()
const windowRootClass = computed(
  () => [attrs.class as string, props.frameClass].filter(Boolean),
)
const slots = useSlots()
const hasHeaderBelowNavSlot = computed(
  () => typeof slots['header-below-nav'] === 'function',
)
const hasNavSlot = computed(() => typeof slots.nav === 'function')
</script>

<template>
  <DesktopCoreWindow
    :class="windowRootClass"
    :window="props.window"
    :content="props.content"
    v-show="props.window?.state?.active ?? true"
    @drag:start="$emit('drag:start', $event)"
    @drag:move="$emit('drag:move', $event)"
    @drag:end="$emit('drag:end', $event)"
    @resize:start="$emit('resize:start', $event)"
    @resize:move="$emit('resize:move', $event)"
    @resize:end="$emit('resize:end', $event)"
  >
    <div :class="['flex flex-col h-full bg-white dark:bg-gray-950 border border-gray-200 dark:border-gray-800 rounded-lg shadow-lg overflow-hidden', props.chromePadding === false ? '' : 'p-2']">
      <div class="desktop-explorer-frame__header-stack">
        <slot v-if="hasNavSlot" name="nav" />
        <Nav v-else>
          <template #prepend>
            <slot name="nav-prepend" />
          </template>
          <template v-slot:title v-if="$slots['nav-title']">
            <slot name="nav-title" />
          </template>
          <template #append>
            <slot name="nav-append" />
          </template>
        </Nav>
        <div
          v-if="hasHeaderBelowNavSlot"
          class="desktop-explorer-frame__header-below-nav"
        >
          <slot name="header-below-nav" />
        </div>
      </div>
      <div class="flex-1 min-h-0 flex flex-col">
        <slot />
      </div>
    </div>
  </DesktopCoreWindow>
</template>

<style scoped>
.desktop-explorer-frame__header-stack {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.desktop-explorer-frame__header-below-nav {
  min-width: 0;
}

.owd-window.resizable-component {
  box-sizing: border-box;
  display: inline-block;
  min-width: 160px;
  min-height: 160px;
  text-align: left;
  cursor: default;
}
@media (max-width: 599px) {
  .owd-window.resizable-component {
    top: 0 !important;
    bottom: 0 !important;
    left: 0 !important;
    right: 0 !important;
    max-width: 100vw;
    max-height: 100vh;
    height: 100vh !important;
    z-index: 9999999999 !important;
  }
}
</style>
