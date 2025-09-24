<template>
  <div class="relative">
    <input
      ref="monthPickerRef"
      :value="displayValue"
      :placeholder="placeholderText"
      readonly
      @click="togglePicker"
      @keydown.enter="togglePicker"
      @keydown.space.prevent="togglePicker"
      @keydown.escape="closePicker"
      :aria-expanded="isOpen"
      aria-haspopup="dialog"
      class="focus:ring-primary-500 focus:border-primary-500 w-full cursor-pointer rounded-md border border-gray-300 bg-white px-3 py-2 pr-10 text-sm text-gray-900 placeholder-gray-500 transition-colors duration-200 hover:border-gray-400 focus:ring-1 focus:outline-none"
    />

    <div
      v-if="isOpen"
      ref="dropdown"
      role="dialog"
      aria-modal="false"
      aria-label="Month picker"
      class="absolute top-full left-0 z-50 mt-1 w-xs rounded-lg border border-gray-200 bg-white shadow-lg"
    >
      <div class="flex flex-col">
        <div class="flex items-center justify-between border-b border-gray-100 p-3">
          <button
            type="button"
            @click="navigate('previous')"
            :disabled="!canNavigatePrevious"
            aria-label="Go to previous year"
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

          <h2 class="text-sm font-medium text-gray-900" aria-live="polite">{{ displayYear }}</h2>

          <button
            type="button"
            @click="navigate('next')"
            :disabled="!canNavigateNext"
            aria-label="Go to next year"
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

        <div class="grid grid-cols-3 gap-2 p-3">
          <button
            v-for="item in monthGrid"
            :key="item.month"
            @click="handleMonthClick(item)"
            :disabled="!isMonthSelectable(displayYear, item.month)"
            :class="getMonthClasses(item)"
            class="relative h-10 w-auto cursor-pointer rounded text-sm transition-colors"
          >
            {{ item.shortName }}
          </button>
        </div>

        <div class="flex justify-end border-t border-gray-100 p-1.5">
          <button
            @click="clearSelection"
            class="text-primary-700 hover:bg-primary-300 bg-primary-200 cursor-pointer rounded px-3 py-1 text-sm transition-colors"
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

interface MonthData {
  month: number
  name: string
  shortName: string
  date: string
  isCurrentMonth: boolean
}

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

const props = withDefaults(
  defineProps<{
    modelValue?: string | string[] | null
    placeholder?: string
    mode?: 'single' | 'range'
    minNavigation?: string | null
    maxNavigation?: string | null
  }>(),
  {
    placeholder: '',
    mode: 'single',
    minNavigation: null,
    maxNavigation: null,
  }
)

const emit = defineEmits<{ 'update:modelValue': [string | string[] | null] }>()

const isOpen = ref(false)
const monthPickerRef = ref<HTMLInputElement | null>(null)
const dropdown = ref<HTMLElement | null>(null)
const currentDate = ref(new Date())
const selectedSingle = ref<Date | null>(null)
const selectedRange = ref<{ start: Date | null; end: Date | null }>({ start: null, end: null })
const isSingleMode = computed(() => props.mode === 'single')

const formatDate = (d: Date, fmt: 'YM' | 'MonYYYY') => {
  const y = d.getFullYear()
  const m = String(d.getMonth() + 1).padStart(2, '0')
  const shortMonth = MONTH_NAMES[d.getMonth()].slice(0, 3)
  return fmt === 'YM' ? `${y}-${m}` : `${shortMonth} ${y}`
}

const parseModelDate = (v?: string | null) => {
  if (!v) return null
  const ymMatch = /^(\d{4})-(\d{2})$/.exec(v)
  if (ymMatch) return new Date(+ymMatch[1], +ymMatch[2] - 1, 1)
  const dt = new Date(v)
  return isNaN(dt.getTime()) ? null : dt
}

const isMonthSelectable = (year: number, month: number) => {
  const min = props.minNavigation && parseModelDate(props.minNavigation)
  const max = props.maxNavigation && parseModelDate(props.maxNavigation)
  if (min && (year < min.getFullYear() || (year === min.getFullYear() && month < min.getMonth())))
    return false
  if (max && (year > max.getFullYear() || (year === max.getFullYear() && month > max.getMonth())))
    return false
  return true
}

const displayYear = computed(() => currentDate.value.getFullYear())

const placeholderText = computed(() => {
  if (props.placeholder) return props.placeholder
  return isSingleMode.value ? 'Select month' : 'Select month range'
})

const canNavigatePrevious = computed(() => {
  const min = props.minNavigation && parseModelDate(props.minNavigation)
  if (!min) return true
  return displayYear.value - 1 >= min.getFullYear()
})

const canNavigateNext = computed(() => {
  const max = props.maxNavigation && parseModelDate(props.maxNavigation)
  if (!max) return true
  return displayYear.value + 1 <= max.getFullYear()
})

const monthGrid = computed<MonthData[]>(() => {
  return MONTH_NAMES.map((name, index) => ({
    month: index,
    name,
    shortName: name.slice(0, 3),
    date: formatDate(new Date(displayYear.value, index, 1), 'YM'),
    isCurrentMonth:
      new Date().getFullYear() === displayYear.value && new Date().getMonth() === index,
  }))
})

const getSelectedValues = (): string[] => {
  if (isSingleMode.value)
    return selectedSingle.value ? [formatDate(selectedSingle.value, 'YM')] : []
  return selectedRange.value.start && selectedRange.value.end
    ? [formatDate(selectedRange.value.start, 'YM'), formatDate(selectedRange.value.end, 'YM')]
    : []
}

const getMonthClasses = (m: MonthData): string => {
  if (!isMonthSelectable(displayYear.value, m.month)) return 'text-gray-400 cursor-not-allowed'
  const values = getSelectedValues()
  const start = values[0]
  const end = values[1]
  const isSelected = start === m.date || end === m.date
  const isInRange = start && end && m.date > start && m.date < end

  if (isSelected) return 'bg-primary-700 text-white font-medium shadow-md'
  if (isInRange) return 'bg-primary-300 text-white border border-primary-400'
  if (m.isCurrentMonth) return 'border border-primary-300 text-primary-800 font-medium'
  return 'text-gray-700 hover:bg-primary-300'
}

const displayValue = computed(() => {
  if (isSingleMode.value)
    return selectedSingle.value ? formatDate(selectedSingle.value, 'MonYYYY') : ''
  const { start, end } = selectedRange.value
  if (start && end) return `${formatDate(start, 'MonYYYY')} - ${formatDate(end, 'MonYYYY')}`
  if (start) return formatDate(start, 'MonYYYY')
  if (end) return formatDate(end, 'MonYYYY')
  return ''
})

const togglePicker = () => {
  return (isOpen.value = !isOpen.value)
}

const closePicker = () => {
  return (isOpen.value = false)
}

const navigate = (dir: 'previous' | 'next') => {
  if (
    (dir === 'previous' && !canNavigatePrevious.value) ||
    (dir === 'next' && !canNavigateNext.value)
  )
    return
  return (currentDate.value = new Date(
    displayYear.value + (dir === 'previous' ? -1 : 1),
    currentDate.value.getMonth(),
    1
  ))
}

const handleMonthClick = (m: MonthData) => {
  if (!isMonthSelectable(displayYear.value, m.month)) return
  const selected = new Date(displayYear.value, m.month, 1)
  if (isSingleMode.value) {
    selectedSingle.value = selected
    emit('update:modelValue', formatDate(selected, 'YM'))
    return (isOpen.value = false)
  }

  if (!selectedRange.value.start || selectedRange.value.end) {
    selectedRange.value = { start: selected, end: null }
  } else {
    const start = selected < selectedRange.value.start ? selected : selectedRange.value.start
    const end = selected > selectedRange.value.start ? selected : selectedRange.value.start
    selectedRange.value = { start, end }
    emit('update:modelValue', [formatDate(start, 'YM'), formatDate(end, 'YM')])
    return (isOpen.value = false)
  }
}

const clearSelection = () => {
  selectedSingle.value = null
  selectedRange.value = { start: null, end: null }
  emit('update:modelValue', null)
  return (isOpen.value = false)
}

const handleClickOutside = (e: Event) => {
  const t = e.target as Node
  if (!dropdown.value?.contains(t) && !monthPickerRef.value?.contains(t)) return closePicker()
}

watch(
  () => props.modelValue,
  (newVal) => {
    if (isSingleMode.value) {
      selectedSingle.value = typeof newVal === 'string' ? parseModelDate(newVal) : null
    } else {
      const arr = Array.isArray(newVal) ? newVal : []
      selectedRange.value = {
        start: arr[0] ? parseModelDate(arr[0]) : null,
        end: arr[1] ? parseModelDate(arr[1]) : null,
      }
    }
  },
  { immediate: true }
)

onMounted(() => document.addEventListener('click', handleClickOutside))
onUnmounted(() => document.removeEventListener('click', handleClickOutside))
</script>
