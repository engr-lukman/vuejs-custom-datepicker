<template>
  <div class="relative">
    <input
      ref="datePickerRef"
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
    <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-3">
      <svg class="h-5 w-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"
        />
      </svg>
    </div>
    <div
      v-if="isOpen"
      ref="dropdown"
      role="dialog"
      aria-modal="false"
      aria-label="Month picker"
      class="absolute top-full left-0 z-50 mt-1 w-xs max-w-[calc(100vw-1rem)] rounded-lg border border-gray-200 bg-white shadow-lg"
    >
      <div class="flex">
        <div class="flex-1">
          <div class="flex items-center justify-between border-b border-gray-100 p-3">
            <button
              type="button"
              @click="navigate('previous')"
              :disabled="!canNavigatePrevious"
              aria-label="Go to previous year"
              class="hover:bg-primary-300 cursor-pointer rounded p-1 transition-colors disabled:cursor-not-allowed disabled:opacity-50"
            >
              <svg
                class="h-5 w-5"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
                aria-hidden="true"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M15 19l-7-7 7-7"
                />
              </svg>
            </button>
            <h2 class="text-sm font-medium text-gray-900" aria-live="polite">
              {{ displayYear }}
            </h2>
            <button
              type="button"
              @click="navigate('next')"
              :disabled="!canNavigateNext"
              aria-label="Go to next year"
              class="hover:bg-primary-300 cursor-pointer rounded p-1 transition-colors disabled:cursor-not-allowed disabled:opacity-50"
            >
              <svg
                class="h-5 w-5"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
                aria-hidden="true"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M9 5l7 7-7 7"
                />
              </svg>
            </button>
          </div>
          <div class="p-3">
            <div class="grid grid-cols-3 gap-2">
              <button
                v-for="monthData in monthGrid"
                :key="monthData.month"
                :disabled="!isMonthSelectable(displayYear, monthData.month)"
                @click="handleMonthClick(monthData)"
                :class="getMonthClasses(monthData)"
                class="hover:bg-primary-300 relative cursor-pointer rounded px-3 py-2 text-sm transition-colors disabled:cursor-not-allowed disabled:opacity-50"
              >
                {{ monthData.shortName }}
              </button>
            </div>
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
  </div>
</template>

<script setup lang="ts">
import { computed, ref, watch, onMounted, onUnmounted } from 'vue'

interface YearMonth {
  year: number
  month: number
}

interface DatePickerProps {
  modelValue?: (string | null)[] | string | null
  placeholder?: string
  mode?: 'range' | 'single'
  minNavigation?: string | null
  maxNavigation?: string | null
}

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

const startOfDay = (date: Date): Date => {
  const d = new Date(date)
  d.setHours(0, 0, 0, 0)
  return d
}

const createYearMonthDate = (year: number, month: number): Date => new Date(year, month, 1)

const formatDate = (date: Date, format: 'YM' | 'MonYYYY'): string => {
  const y = date.getFullYear()
  const m = String(date.getMonth() + 1).padStart(2, '0')
  const monthName = MONTH_NAMES[date.getMonth()].slice(0, 3)
  switch (format) {
    case 'YM':
      return `${y}-${m}`
    case 'MonYYYY':
      return `${monthName} ${y}`
  }
}

const parseModelDate = (value?: string | null): Date | null => {
  if (!value || typeof value !== 'string') return null
  const ym = /^(\d{4})-(\d{2})$/
  if (ym.test(value)) {
    const [y, m] = value.split('-').map(Number)
    const dt = new Date(y, m - 1, 1)
    return Number.isNaN(dt.getTime()) ? null : dt
  }
  const dt = new Date(value)
  return Number.isNaN(dt.getTime()) ? null : dt
}

const compareYearMonth = (a: YearMonth, b: YearMonth): number => {
  if (a.year !== b.year) return a.year < b.year ? -1 : 1
  return a.month - b.month
}

const isCurrentMonth = (year: number, month: number): boolean => {
  const today = new Date()
  return year === today.getFullYear() && month === today.getMonth()
}

const isWithinBounds = (
  value: Date | YearMonth,
  minNavigation?: string | null,
  maxNavigation?: string | null
): boolean => {
  const minDate = minNavigation ? parseModelDate(minNavigation) : null
  const maxDate = maxNavigation ? parseModelDate(maxNavigation) : null
  if (!minDate && !maxDate) return true
  const yearMonth =
    value instanceof Date ? { year: value.getFullYear(), month: value.getMonth() } : value
  if (
    minDate &&
    compareYearMonth(yearMonth, { year: minDate.getFullYear(), month: minDate.getMonth() }) < 0
  )
    return false
  if (
    maxDate &&
    compareYearMonth(yearMonth, { year: maxDate.getFullYear(), month: maxDate.getMonth() }) > 0
  )
    return false
  return true
}

const isMonthWithinBounds = (
  ym: YearMonth,
  minNavigation?: string | null,
  maxNavigation?: string | null
): boolean => isWithinBounds(ym, minNavigation, maxNavigation)

const props = withDefaults(defineProps<DatePickerProps>(), {
  placeholder: '',
  mode: 'single',
  minNavigation: null,
  maxNavigation: null,
})

const emit = defineEmits<{
  'update:modelValue': [value: (string | null)[] | string | null]
}>()

const isOpen = ref(false)
const datePickerRef = ref<HTMLInputElement | null>(null)
const dropdown = ref<HTMLElement | null>(null)
const currentDate = ref(new Date())
const selectedSingle = ref<Date | null>(null)
const selectedRange = ref<{ start: Date | null; end: Date | null }>({ start: null, end: null })

const isSingleMode = computed(() => props.mode === 'single')

const placeholderText = computed(() => {
  if (props.placeholder) return props.placeholder
  return isSingleMode.value ? 'Select month' : 'Select month range'
})

const displayYear = computed(() => currentDate.value.getFullYear())

const canNavigatePrevious = computed(() => {
  if (!props.minNavigation) return true
  const currentYear = currentDate.value.getFullYear()
  const minDate = parseModelDate(props.minNavigation)
  return minDate ? currentYear - 1 >= minDate.getFullYear() : true
})

const canNavigateNext = computed(() => {
  if (!props.maxNavigation) return true
  const currentYear = currentDate.value.getFullYear()
  const maxDate = parseModelDate(props.maxNavigation)
  return maxDate ? currentYear + 1 <= maxDate.getFullYear() : true
})

const displayValue = computed(() => {
  if (isSingleMode.value) {
    if (selectedSingle.value) {
      return formatDate(selectedSingle.value, 'MonYYYY')
    }
    return ''
  } else {
    if (selectedRange.value?.start && selectedRange.value?.end) {
      return `${formatDate(selectedRange.value.start, 'MonYYYY')} - ${formatDate(selectedRange.value.end, 'MonYYYY')}`
    }
    return ''
  }
})

const monthGrid = computed((): MonthData[] => {
  return MONTH_NAMES.map((name, index) => ({
    month: index,
    name,
    shortName: name.slice(0, 3),
    date: formatDate(createYearMonthDate(displayYear.value, index), 'YM'),
    isCurrentMonth: isCurrentMonth(displayYear.value, index),
  }))
})

const isMonthSelectable = (year: number, month: number): boolean =>
  isMonthWithinBounds({ year, month }, props.minNavigation, props.maxNavigation)

const isMonthSelected = (monthString: string): boolean => {
  if (isSingleMode.value) {
    return typeof props.modelValue === 'string' && props.modelValue === monthString
  }
  const values = Array.isArray(props.modelValue) ? props.modelValue : []
  return values.includes(monthString)
}

const isMonthInSelectedRange = (monthString: string): boolean => {
  if (isSingleMode.value) return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : []
  return (
    values.length >= 2 &&
    !!values[0] &&
    !!values[1] &&
    monthString > values[0] &&
    monthString < values[1]
  )
}

const isMonthRangeStart = (monthString: string): boolean => {
  if (isSingleMode.value) return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : []
  return values[0] === monthString
}

const isMonthRangeEnd = (monthString: string): boolean => {
  if (isSingleMode.value) return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : []
  return values[1] === monthString
}

const getMonthClasses = (monthData: MonthData): string => {
  if (!isMonthSelectable(displayYear.value, monthData.month)) {
    return 'text-gray-400 cursor-not-allowed'
  }
  const isSelected = isMonthSelected(monthData.date)
  if (isSingleMode.value) {
    if (isSelected) {
      return 'bg-primary-700 text-white font-medium shadow-md'
    } else if (monthData.isCurrentMonth) {
      return 'border border-primary-300 text-primary-800 font-medium'
    } else {
      return 'text-gray-700 hover:bg-primary-300'
    }
  } else {
    const isInRange = isMonthInSelectedRange(monthData.date)
    const isRangeStart = isMonthRangeStart(monthData.date)
    const isRangeEnd = isMonthRangeEnd(monthData.date)
    if (isRangeStart || isRangeEnd) {
      return 'bg-primary-700 text-white font-medium shadow-md'
    } else if (isSelected) {
      return 'bg-primary-700 text-white font-medium shadow-md'
    } else if (isInRange) {
      return 'bg-primary-300 text-white border border-primary-400'
    } else if (monthData.isCurrentMonth) {
      return 'border border-primary-300 text-primary-800 font-medium'
    } else {
      return 'text-gray-700 hover:bg-primary-300'
    }
  }
}

const handleMonthClick = (monthData: MonthData): void => {
  if (!isMonthSelectable(displayYear.value, monthData.month)) return
  const selectedMonth = new Date(displayYear.value, monthData.month, 1)
  if (isSingleMode.value) {
    selectedSingle.value = selectedMonth
    emit('update:modelValue', formatDate(selectedMonth, 'YM'))
    isOpen.value = false
  } else {
    if (!selectedRange.value?.start || (selectedRange.value?.start && selectedRange.value?.end)) {
      selectedRange.value = { start: selectedMonth, end: null }
    } else if (selectedRange.value?.start && !selectedRange.value?.end) {
      const startMonth =
        selectedMonth < selectedRange.value.start ? selectedMonth : selectedRange.value.start
      const endMonth =
        selectedMonth < selectedRange.value.start ? selectedRange.value.start : selectedMonth
      selectedRange.value = { start: startMonth, end: endMonth }
      emit('update:modelValue', [formatDate(startMonth, 'YM'), formatDate(endMonth, 'YM')])
      isOpen.value = false
    }
  }
}

const clearSelection = (): void => {
  if (isSingleMode.value) {
    selectedSingle.value = null
  } else {
    selectedRange.value = { start: null, end: null }
  }
  emit('update:modelValue', null)
  isOpen.value = false
}

const togglePicker = (): void => {
  isOpen.value = !isOpen.value
}

const closePicker = (): void => {
  isOpen.value = false
}

const navigate = (direction: 'previous' | 'next'): void => {
  const canNavigate = direction === 'previous' ? canNavigatePrevious.value : canNavigateNext.value
  if (!canNavigate) return
  const yearOffset = direction === 'previous' ? -1 : 1
  currentDate.value = new Date(
    currentDate.value.getFullYear() + yearOffset,
    currentDate.value.getMonth(),
    1
  )
}

const handleClickOutside = (event: Event): void => {
  const target = event.target as Node
  if (
    dropdown.value &&
    !dropdown.value.contains(target) &&
    datePickerRef.value &&
    !datePickerRef.value.contains(target)
  ) {
    closePicker()
  }
}

const handleKeyDown = (event: KeyboardEvent): void => {
  if (!isOpen.value) return
  if (event.key === 'Escape') {
    event.preventDefault()
    closePicker()
  }
}

watch(
  () => props.modelValue,
  (newValue) => {
    if (isSingleMode.value) {
      if (typeof newValue === 'string' && newValue) {
        const parsed = parseModelDate(newValue)
        selectedSingle.value = parsed ? startOfDay(parsed) : null
      } else {
        selectedSingle.value = null
      }
    } else {
      if (!newValue || !Array.isArray(newValue)) {
        selectedRange.value = { start: null, end: null }
        return
      }
      if (newValue.length >= 2) {
        const s = newValue[0]
        const e = newValue[1]
        const sParsed = s ? parseModelDate(s) : null
        const eParsed = e ? parseModelDate(e) : null
        selectedRange.value = {
          start: sParsed ? startOfDay(sParsed) : null,
          end: eParsed ? startOfDay(eParsed) : null,
        }
      } else {
        selectedRange.value = { start: null, end: null }
      }
    }
  },
  { immediate: true }
)

onMounted(() => {
  document.addEventListener('click', handleClickOutside)
  document.addEventListener('keydown', handleKeyDown)
})

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside)
  document.removeEventListener('keydown', handleKeyDown)
})
</script>
