<script setup lang="ts">
import type { VNodeRef } from '@vue/runtime-core'
import { EditColumnInj, EditModeInj, IsExpandedFormOpenInj, inject, useVModel } from '#imports'

interface Props {
  modelValue?: number | string | null
}

const props = defineProps<Props>()

const emits = defineEmits(['update:modelValue'])

const { showNull } = useGlobal()

const column = inject(ColumnInj)!

const editEnabled = inject(EditModeInj)

const isEditColumn = inject(EditColumnInj, ref(false))

const _vModel = useVModel(props, 'modelValue', emits)

const wrapperRef = ref<HTMLElement>()

const vModel = computed({
  get: () => _vModel.value,
  set: (value) => {
    if (value === '') {
      _vModel.value = null
    } else {
      _vModel.value = value
    }
  },
})

const isExpandedFormOpen = inject(IsExpandedFormOpenInj, ref(false))!

const focus: VNodeRef = (el) => !isExpandedFormOpen.value && !isEditColumn.value && (el as HTMLInputElement)?.focus()

const cellFocused = ref(false)

const expandedEditEnabled = ref(false)

const percentMeta = computed(() => {
  return {
    is_progress: false,
    ...parseProp(column.value?.meta),
  }
})

const onBlur = () => {
  if (editEnabled) {
    editEnabled.value = false
  }
  cellFocused.value = false
  expandedEditEnabled.value = false
}

const onFocus = () => {
  cellFocused.value = true
  editEnabled.value = true
  expandedEditEnabled.value = true
}

const onWrapperFocus = () => {
  cellFocused.value = true
  editEnabled.value = true
  expandedEditEnabled.value = true

  nextTick(() => {
    wrapperRef.value?.querySelector('input')?.focus()
  })
}

const onMouseover = () => {
  expandedEditEnabled.value = true
}

const onMouseleave = () => {
  if (!cellFocused.value) {
    expandedEditEnabled.value = false
  }
}

const onTabPress = (e: KeyboardEvent) => {
  if (e.shiftKey) {
    e.preventDefault()

    // Shift + Tab does not work for percent cell
    // so we manually focus on the last form item

    const focusesNcCellIndex = Array.from(document.querySelectorAll('.nc-expanded-form-row .nc-data-cell')).findIndex((el) => {
      return el.querySelector('.nc-filter-value-select') === wrapperRef.value
    })

    if (focusesNcCellIndex >= 0) {
      const nodes = document.querySelectorAll('.nc-expanded-form-row .nc-data-cell')
      for (let i = focusesNcCellIndex - 1; i >= 0; i--) {
        const lastFormItem = nodes[i].querySelector('[tabindex="0"]') as HTMLElement
        if (lastFormItem) {
          lastFormItem.focus()
          break
        }
      }
    }
  }
}
</script>

<template>
  <div
    ref="wrapperRef"
    tabindex="0"
    class="nc-filter-value-select w-full focus:outline-transparent"
    @mouseover="onMouseover"
    @mouseleave="onMouseleave"
    @focus="onWrapperFocus"
  >
    <input
      v-if="(!isExpandedFormOpen && editEnabled) || (isExpandedFormOpen && expandedEditEnabled)"
      :ref="focus"
      v-model="vModel"
      class="w-full !text-sm !border-none !outline-none focus:ring-0 text-base p-1"
      :class="{ '!px-2': editEnabled }"
      type="number"
      :placeholder="isEditColumn ? $t('labels.optional') : ''"
      @blur="onBlur"
      @focus="onFocus"
      @keydown.down.stop
      @keydown.left.stop
      @keydown.right.stop
      @keydown.up.stop
      @keydown.delete.stop
      @keydown.tab="onTabPress"
      @selectstart.capture.stop
      @mousedown.stop
    />
    <span v-else-if="vModel === null && showNull" class="nc-null capitalize">{{ $t('general.null') }}</span>
    <div v-else-if="percentMeta.is_progress === true && vModel !== null && vModel !== undefined" class="px-2">
      <a-progress
        :percent="Number(parseFloat(vModel.toString()).toFixed(2))"
        size="small"
        status="normal"
        stroke-color="#3366FF"
        trail-color="#E5E5E5"
        :show-info="false"
      />
    </div>
    <!-- nbsp to keep height even if vModel is zero length -->
    <span v-else>{{ vModel }}&nbsp;</span>
  </div>
</template>
