<script setup lang="ts">
import { useExplorerWindow } from '@owdproject/nuxt-module-fs/runtime/composables/useExplorerWindow'
import { useI18n } from 'vue-i18n'
import { explorerEntryAbsolutePath } from '@owdproject/nuxt-module-fs/runtime/utils/utilExplorerEntryPath'
import { computed } from 'vue'
import Frame from './Frame.vue'
import Chrome from './Chrome.vue'
import FilePane from './FilePane.vue'
import AddressBar from './AddressBar.vue'
import DefaultToolbar from './DefaultToolbar.vue'

const props = defineProps<{
  config?: Record<string, any>
  window: Record<string, any>
  frameClass?: string
  chromePadding?: boolean
}>()

const { t } = useI18n()
const fsExplorer = useExplorerWindow(props.window as any, t)
props.window.fsExplorer = fsExplorer
void fsExplorer.initialize()

function mapMenuItems(items: any[]) {
  if (!items) return []
  return [
    items.map(subItem => ({
      label: subItem.label,
      icon: subItem.icon,
      click: () => {
        if (typeof subItem.command === 'function') {
          subItem.command()
        }
      }
    }))
  ]
}
</script>

<template>
  <Frame
    :window="window"
    :frame-class="frameClass"
    :chrome-padding="chromePadding ?? false"
  >
    <template v-if="$slots.nav" v-slot:nav>
      <slot name="nav" />
    </template>

    <Chrome>
      <!-- Menu bar row -->
      <div v-if="window.menu && window.menu.length > 0" class="flex items-center space-x-1 px-2 py-1 bg-gray-50 dark:bg-gray-900 border-b border-gray-200 dark:border-gray-800 flex-shrink-0">
        <UDropdown
          v-for="menu of window.menu"
          :key="menu.label"
          :items="mapMenuItems(menu.items)"
          :popper="{ placement: 'bottom-start' }"
        >
          <UButton
            color="gray"
            variant="ghost"
            size="xs"
            class="text-xs font-medium py-1 px-2"
          >
            {{ menu.label }}
          </UButton>
        </UDropdown>
      </div>

      <slot name="toolbar">
        <DefaultToolbar
          :arrows-disabled="fsExplorer.fsDirectoryNavigation.history.value.length <= 1"
          @back="fsExplorer.directoryBack"
          @forward="fsExplorer.directoryForward"
          @up="fsExplorer.directoryUp"
          @cut="fsExplorer.cutSelectedFiles"
          @copy="fsExplorer.copySelectedFiles"
          @paste="() => fsExplorer.fsController?.pasteClipboardFiles()"
          @delete="() => fsExplorer.fsController?.deleteSelectedFiles()"
          @properties="fsExplorer.fileProperties"
        />
      </slot>

      <slot name="address">
        <AddressBar
          :address="fsExplorer.basePath.value ?? ''"
          @update:model-value="(value) => {
            fsExplorer.basePath.value = value
          }"
        />
      </slot>

      <slot name="file-pane">
        <FilePane>
          <DesktopExplorerSelectableArea
            v-if="!String(window.meta.path ?? '').startsWith('http')"
            :fs-explorer="fsExplorer"
          >
            <DesktopExplorerFileEntry
              v-for="fileName of fsExplorer.fsEntries.value"
              :key="fileName"
              :data-filename="fileName"
              :basePath="fsExplorer.basePath.value"
              :fileName="fileName"
              :layout="fsExplorer.layout.value"
              :selected="fsExplorer.selectedFiles.value.includes(explorerEntryAbsolutePath(fsExplorer.basePath.value, fileName))"
              :cutted="fsExplorer.fsClipboard.clipboardFiles.value.includes(explorerEntryAbsolutePath(fsExplorer.basePath.value, fileName)) && fsExplorer.fsClipboard.clipboardType.value === 'cut'"
              :window="window"
            />
          </DesktopExplorerSelectableArea>
          <iframe
            v-else
            class="w-full h-full border-0"
            :src="window.meta.path ?? ''"
            :style="{ pointerEvents: window?.state?.focused ? 'auto' : 'none' }"
          />
        </FilePane>
      </slot>
    </Chrome>
  </Frame>
</template>
