<template>
  <div class="relative">
    <!-- Month Input -->
    <input
      ref="monthInputRef"
      :value="formattedDisplayValue"
      :placeholder="placeholderText"
      readonly
      @click="togglePicker"
      @keydown.enter="togglePicker"
      @keydown.space.prevent="togglePicker"
      @keydown.escape="closePicker"
      :aria-expanded="isPickerOpen"
      aria-haspopup="dialog"
      class="focus:ring-primary-500 focus:border-primary-500 w-full cursor-pointer rounded-md border border-gray-300 bg-white px-3 py-2 pr-10 text-sm text-gray-900 placeholder-gray-500 transition-colors duration-200 hover:border-gray-400 focus:ring-1 focus:outline-none"
    />

    <!-- Month Picker Dropdown -->
    <div
      v-if="isPickerOpen"
      ref="dropdownRef"
      role="dialog"
      aria-modal="false"
      aria-label="Month picker"
      class="absolute top-full left-0 z-50 mt-1 w-xs rounded-lg border border-gray-200 bg-white shadow-lg"
    >
      <div class="flex flex-col">
        <!-- Year Navigation -->
        <div class="flex items-center justify-between border-b border-gray-100 p-3">
          <button
            type="button"
            @click="navigateYear('previous')"
            :disabled="!canNavigatePreviousYear"
            aria-label="Previous year"
            class="hover:bg-primary-300 cursor-pointer rounded p-1 transition-colors disabled:cursor-not-allowed disabled:opacity-50"
          >
            <svg class="h-5 w-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M15 19l-7-7 7-7"
              />
            </svg>
          </button>

          <h2 class="text-sm font-medium text-gray-900" aria-live="polite">{{ displayedYear }}</h2>

          <button
            type="button"
            @click="navigateYear('next')"
            :disabled="!canNavigateNextYear"
            aria-label="Next year"
            class="hover:bg-primary-300 cursor-pointer rounded p-1 transition-colors disabled:cursor-not-allowed disabled:opacity-50"
          >
            <svg class="h-5 w-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M9 5l7 7-7 7"
              />
            </svg>
          </button>
        </div>

        <!-- Month Grid -->
        <div class="grid grid-cols-3 gap-y-1 p-3">
          <button
            v-for="month in monthList"
            :key="month.index"
            @click="selectMonth(month)"
            :disabled="!isMonthSelectable(displayedYear, month.index)"
            :class="getMonthButtonClasses(month)"
            class="relative h-10 w-auto cursor-pointer border-4 border-white text-sm transition-colors"
          >
            {{ month.shortName }}
          </button>
        </div>

        <!-- Clear Selection -->
        <div class="flex justify-end border-t border-gray-100 p-1.5">
          <button
            @click="clearSelection"
            class="hover:bg-primary-700 bg-primary-500 cursor-pointer rounded px-3 py-1 text-sm text-white transition-colors"
          >
            Clear
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  ref,
  computed,
  watch,
  onMounted,
  onUnmounted,
  withDefaults,
  defineProps,
  defineEmits,
} from 'vue'

/** ========================
 *  Interfaces
 * ======================== */
interface MonthItem {
  index: number
  fullName: string
  shortName: string
  ymValue: string
  isCurrentMonth: boolean
}

interface DateRange {
  start: Date | null
  end: Date | null
}

/** ========================
 *  Constants
 * ======================== */
const MONTH_NAMES = [
  'January',
  'February',
  'March',
  'April',
  'May',
  'June',
  'July',
  'August',
  'September',
  'October',
  'November',
  'December',
]

/** ========================
 *  Props & Emits
 * ======================== */
const props = withDefaults(
  defineProps<{
    modelValue?: string | string[] | null
    placeholder?: string
    mode?: 'single' | 'range'
    minNavigation?: string | null
    maxNavigation?: string | null
  }>(),
  { placeholder: '', mode: 'single', minNavigation: null, maxNavigation: null }
)

const emit = defineEmits<{ 'update:modelValue': [string | string[] | null] }>()

/** ========================
 *  Reactive State
 * ======================== */
const isPickerOpen = ref(false)
const monthInputRef = ref<HTMLInputElement | null>(null)
const dropdownRef = ref<HTMLElement | null>(null)
const currentDate = ref(new Date())
const selectedSingleMonth = ref<Date | null>(null)
const selectedMonthRange = ref<DateRange>({ start: null, end: null })
const isSingleMode = computed(() => props.mode === 'single')

/** ========================
 *  Computed Properties
 * ======================== */
const displayedYear = computed(() => currentDate.value.getFullYear())
const placeholderText = computed(
  () => props.placeholder || (isSingleMode.value ? 'Select month' : 'Select month range')
)

const monthList = computed<MonthItem[]>(() =>
  MONTH_NAMES.map((name, index) => {
    const date = new Date(displayedYear.value, index, 1)
    return {
      index,
      fullName: name,
      shortName: name.slice(0, 3),
      ymValue: `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`,
      isCurrentMonth:
        new Date().getFullYear() === displayedYear.value && new Date().getMonth() === index,
    }
  })
)

const formattedDisplayValue = computed(() => {
  if (isSingleMode.value)
    return selectedSingleMonth.value
      ? `${MONTH_NAMES[selectedSingleMonth.value.getMonth()]} ${selectedSingleMonth.value.getFullYear()}`
      : ''
  const { start, end } = selectedMonthRange.value
  if (start && end)
    return `${MONTH_NAMES[start.getMonth()]} ${start.getFullYear()} - ${MONTH_NAMES[end.getMonth()]} ${end.getFullYear()}`
  if (start) return `${MONTH_NAMES[start.getMonth()]} ${start.getFullYear()}`
  if (end) return `${MONTH_NAMES[end.getMonth()]} ${end.getFullYear()}`
  return ''
})

/** ========================
 *  Navigation Logic
 * ======================== */
const canNavigatePreviousYear = computed(() => {
  if (!props.minNavigation) return true
  const minDate = new Date(props.minNavigation)
  // Check if there is any selectable month in the previous year
  return (
    displayedYear.value > minDate.getFullYear() ||
    (displayedYear.value === minDate.getFullYear() &&
      monthList.value.some((m) => isMonthSelectable(displayedYear.value - 1, m.index)))
  )
})

const canNavigateNextYear = computed(() => {
  if (!props.maxNavigation) return true
  const maxDate = new Date(props.maxNavigation)
  return (
    displayedYear.value < maxDate.getFullYear() ||
    (displayedYear.value === maxDate.getFullYear() &&
      monthList.value.some((m) => isMonthSelectable(displayedYear.value + 1, m.index)))
  )
})

/** ========================
 *  Utility Functions
 * ======================== */
const parseModelValue = (value?: string | null) => (value ? new Date(value) : null)

const isMonthSelectable = (year: number, month: number) => {
  const minDate = props.minNavigation ? new Date(props.minNavigation) : null
  const maxDate = props.maxNavigation ? new Date(props.maxNavigation) : null
  if (
    minDate &&
    (year < minDate.getFullYear() || (year === minDate.getFullYear() && month < minDate.getMonth()))
  )
    return false
  if (
    maxDate &&
    (year > maxDate.getFullYear() || (year === maxDate.getFullYear() && month > maxDate.getMonth()))
  )
    return false
  return true
}

const getSelectedMonths = (): string[] => {
  if (isSingleMode.value) {
    return selectedSingleMonth.value ? [selectedSingleMonth.value.toISOString()] : []
  }
  const { start, end } = selectedMonthRange.value
  const months: string[] = []
  if (start) months.push(start.toISOString())
  if (end) months.push(end.toISOString())
  return months
}

const getMonthButtonClasses = (month: MonthItem): string => {
  if (!isMonthSelectable(displayedYear.value, month.index)) {
    return 'text-gray-400 !cursor-not-allowed bg-transparent'
  }

  const selectedMonths = getSelectedMonths().map((d) => {
    const date = new Date(d)
    return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`
  })

  const [start, end] = selectedMonths
  const monthValue = month.ymValue

  if (isSingleMode.value) {
    return start === monthValue
      ? 'bg-primary-500 text-white rounded-lg !border-primary-500'
      : 'text-gray-900'
  }

  if (start === monthValue && end === monthValue) {
    return 'bg-primary-500 !border-primary-500 rounded-lg text-white'
  }
  if (start === monthValue && !end) {
    return 'bg-primary-500 !border-primary-500 rounded-lg text-white'
  }
  if (start === monthValue) {
    return 'bg-primary-500 !border-primary-500 rounded-l-lg text-white'
  }
  if (end === monthValue) {
    return 'bg-primary-500 !border-primary-500 rounded-r-lg text-white'
  }
  if (start && end && monthValue > start && monthValue < end) {
    return 'bg-primary-200 !border-primary-200 text-white'
  }
  return 'text-gray-900'
}

/** ========================
 *  Picker Actions
 * ======================== */
const togglePicker = () => (isPickerOpen.value = !isPickerOpen.value)

const closePicker = () => {
  isPickerOpen.value = false

  if (isSingleMode.value) {
    currentDate.value = selectedSingleMonth.value ? new Date(selectedSingleMonth.value) : new Date()
  } else {
    currentDate.value = selectedMonthRange.value.start
      ? new Date(selectedMonthRange.value.start)
      : new Date()
  }
}

const navigateYear = (direction: 'previous' | 'next') => {
  if (
    (direction === 'previous' && !canNavigatePreviousYear.value) ||
    (direction === 'next' && !canNavigateNextYear.value)
  )
    return

  currentDate.value = new Date(
    currentDate.value.getFullYear() + (direction === 'previous' ? -1 : 1),
    currentDate.value.getMonth(),
    1
  )
}

const selectMonth = (month: MonthItem) => {
  if (!isMonthSelectable(displayedYear.value, month.index)) return
  const selected = new Date(displayedYear.value, month.index, 1)
  if (isSingleMode.value) {
    selectedSingleMonth.value = selected
    emit('update:modelValue', selected.toISOString())
    return (isPickerOpen.value = false)
  }

  if (!selectedMonthRange.value.start || selectedMonthRange.value.end) {
    selectedMonthRange.value = { start: selected, end: null }
  } else {
    const start =
      selected < selectedMonthRange.value.start ? selected : selectedMonthRange.value.start
    const end =
      selected > selectedMonthRange.value.start ? selected : selectedMonthRange.value.start
    selectedMonthRange.value = { start, end }
    emit('update:modelValue', [start.toISOString(), end.toISOString()])
    isPickerOpen.value = false
  }
}

const clearSelection = () => {
  selectedSingleMonth.value = null
  selectedMonthRange.value = { start: null, end: null }
  emit('update:modelValue', null)
  isPickerOpen.value = false
}

/** ========================
 *  Click Outside Handler
 * ======================== */
const handleClickOutside = (event: Event) => {
  const target = event.target as Node
  if (!dropdownRef.value?.contains(target) && !monthInputRef.value?.contains(target)) closePicker()
}

/** ========================
 *  Watchers
 * ======================== */
watch(
  () => props.modelValue,
  (newVal) => {
    if (isSingleMode.value) {
      selectedSingleMonth.value = typeof newVal === 'string' ? parseModelValue(newVal) : null
    } else {
      const arr = Array.isArray(newVal) ? newVal : []
      selectedMonthRange.value = {
        start: arr[0] ? parseModelValue(arr[0]) : null,
        end: arr[1] ? parseModelValue(arr[1]) : null,
      }
    }
  },
  { immediate: true }
)

/** ========================
 *  Lifecycle
 * ======================== */
onMounted(() => document.addEventListener('click', handleClickOutside))
onUnmounted(() => document.removeEventListener('click', handleClickOutside))
</script>
