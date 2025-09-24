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
    <div
      v-if="isOpen"
      ref="dropdown"
      role="dialog"
      aria-modal="false"
      aria-label="Date picker"
      class="absolute top-full left-0 z-50 mt-1 rounded-lg border border-gray-200 bg-white shadow-lg"
      :class="{ 'w-xs': !showQuickSelection, 'w-sm': showQuickSelection }"
    >
      <div class="flex flex-col sm:flex-row">
        <div
          v-if="showQuickSelection"
          class="w-full border-b border-gray-100 bg-gradient-to-b from-gray-50 to-gray-100 sm:w-28 sm:border-r sm:border-b-0"
          role="region"
          aria-label="Quick date selection options"
        >
          <div class="p-2">
            <div
              class="bg-primary-100 text-primary-700 mb-2 rounded px-2 py-1 text-xs font-medium tracking-wide"
            >
              Quick Select
            </div>
            <div class="space-y-1" role="group" aria-label="Quick selection buttons">
              <button
                v-for="option in quickSelectionOptions"
                :key="option.key"
                type="button"
                :disabled="!option.isEnabled"
                @click="handleQuickSelection(option)"
                :class="getQuickSelectionClasses(option)"
                class="w-full cursor-pointer rounded px-2 py-1.5 text-left text-xs transition-colors"
              >
                {{ option.label }}
              </button>
            </div>
          </div>
        </div>
        <div class="flex-1">
          <div class="flex items-center justify-between border-b border-gray-100 p-3">
            <button
              type="button"
              @click="navigate('previous')"
              :disabled="!canNavigatePrevious"
              aria-label="Go to previous month"
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
              {{ `${displayMonth} ${displayYear}` }}
            </h2>
            <button
              type="button"
              @click="navigate('next')"
              :disabled="!canNavigateNext"
              aria-label="Go to next month"
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
            <div class="mb-2 grid grid-cols-7 gap-2">
              <div
                v-for="day in DAY_NAMES"
                :key="day"
                class="text-center text-xs font-medium text-gray-500"
              >
                {{ day }}
              </div>
            </div>
            <div class="grid grid-cols-7 gap-2">
              <button
                v-for="dayData in calendarDays"
                :key="dayData.dateString"
                :disabled="!dayData.isCurrentMonth || !isDateSelectable(dayData.date)"
                @click="handleDateClick(dayData.date)"
                :class="getDayClasses(dayData)"
                class="relative h-8 w-8 cursor-pointer rounded text-sm transition-colors"
              >
                {{ dayData.day }}
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
import {
  computed,
  ref,
  watch,
  onMounted,
  onUnmounted,
  withDefaults,
  defineProps,
  defineEmits,
} from 'vue'

interface DateRange {
  start: Date | null
  end: Date | null
}
interface DatePickerProps {
  modelValue?: string | string[] | null
  placeholder?: string
  mode?: 'single' | 'range'
  minNavigation?: string | null
  maxNavigation?: string | null
}
interface DayData {
  date: Date
  day: number
  month: number
  year: number
  isCurrentMonth: boolean
  isToday: boolean
  dateString: string
}
interface QuickSelectionOption {
  key: string
  label: string
  getValue: () => string[]
  isEnabled?: boolean
}

const DAY_NAMES = ['Su', 'Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa']
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

const startOfDay = (d: Date): Date => {
  const dt = new Date(d)
  dt.setHours(0, 0, 0, 0)
  return dt
}
const formatDate = (d: Date, fmt: 'YMD' | 'DMY' | 'MonYYYY'): string => {
  const y = d.getFullYear(),
    m = String(d.getMonth() + 1).padStart(2, '0'),
    day = String(d.getDate()).padStart(2, '0')
  if (fmt === 'YMD') return `${y}-${m}-${day}`
  if (fmt === 'DMY') return `${day}/${m}/${y}`
  return `${MONTH_NAMES[d.getMonth()].slice(0, 3)} ${y}`
}
const parseModelDate = (v?: string | null): Date | null => {
  if (!v) return null
  const match = /^(\d{4})-(\d{2})-(\d{2})$/.exec(v)
  if (match) return new Date(+match[1], +match[2] - 1, +match[3])
  const dt = new Date(v)
  return isNaN(dt.getTime()) ? null : dt
}
const isSameDay = (a: Date, b: Date): boolean =>
  a.getFullYear() === b.getFullYear() &&
  a.getMonth() === b.getMonth() &&
  a.getDate() === b.getDate()
const isToday = (d: Date): boolean => isSameDay(d, new Date())
const isWithinBounds = (d: Date, min?: string | null, max?: string | null): boolean => {
  const date = startOfDay(d)
  const minDate = min ? startOfDay(parseModelDate(min)!) : null
  const maxDate = max ? startOfDay(parseModelDate(max)!) : null
  if (minDate && date < minDate) return false
  if (maxDate && date > maxDate) return false
  return true
}

const props = withDefaults(defineProps<DatePickerProps>(), {
  placeholder: '',
  mode: 'range',
  minNavigation: null,
  maxNavigation: null,
})
const emit = defineEmits<{ 'update:modelValue': [string | string[] | null] }>()

const isOpen = ref(false)
const datePickerRef = ref<HTMLInputElement | null>(null)
const dropdown = ref<HTMLElement | null>(null)
const currentDate = ref(new Date())
const selectedSingle = ref<Date | null>(null)
const selectedRange = ref<DateRange>({ start: null, end: null })

const isSingleMode = computed(() => props.mode === 'single')
const placeholderText = computed(
  () => props.placeholder || (isSingleMode.value ? 'Select date' : 'Select date range')
)
const displayMonth = computed(() => MONTH_NAMES[currentDate.value.getMonth()])
const displayYear = computed(() => currentDate.value.getFullYear())

const canNavigatePrevious = computed(() => {
  if (!props.minNavigation) return true
  const currYear = currentDate.value.getFullYear(),
    currMonth = currentDate.value.getMonth()
  const prevMonth = currMonth === 0 ? 11 : currMonth - 1,
    prevYear = currMonth === 0 ? currYear - 1 : currYear
  const min = parseModelDate(props.minNavigation)
  if (!min) return true
  return (
    prevYear > min.getFullYear() || (prevYear === min.getFullYear() && prevMonth >= min.getMonth())
  )
})
const canNavigateNext = computed(() => {
  if (!props.maxNavigation) return true
  const currYear = currentDate.value.getFullYear(),
    currMonth = currentDate.value.getMonth()
  const nextMonth = currMonth === 11 ? 0 : currMonth + 1,
    nextYear = currMonth === 11 ? currYear + 1 : currYear
  const max = parseModelDate(props.maxNavigation)
  if (!max) return true
  return (
    nextYear < max.getFullYear() || (nextYear === max.getFullYear() && nextMonth <= max.getMonth())
  )
})

// Optimized calendar days: single loop, 6 weeks (42 slots)
const calendarDays = computed<DayData[]>(() => {
  const year = currentDate.value.getFullYear(),
    month = currentDate.value.getMonth()
  const firstDayOfMonth = new Date(year, month, 1)
  const firstWeekDay = firstDayOfMonth.getDay()
  const startDate = new Date(year, month, 1 - firstWeekDay)
  const days: DayData[] = Array.from({ length: 42 }).map((_, i) => {
    const date = new Date(startDate)
    date.setDate(startDate.getDate() + i)
    return {
      date,
      day: date.getDate(),
      month: date.getMonth(),
      year: date.getFullYear(),
      isCurrentMonth: date.getMonth() === month,
      isToday: isToday(date),
      dateString: `${date.getMonth() === month ? 'curr' : date < firstDayOfMonth ? 'prev' : 'next'}-${formatDate(date, 'YMD')}`,
    }
  })
  return days
})

const showQuickSelection = computed(() => !isSingleMode.value)

const quickSelectionOptions = computed<QuickSelectionOption[]>(() => {
  const today = new Date()
  const createRange = (start: Date, end: Date = today) => [
    formatDate(start, 'YMD'),
    formatDate(end, 'YMD'),
  ]
  const daysAgo = (n: number) => {
    const d = new Date(today)
    d.setDate(today.getDate() - n)
    return d
  }
  const opts: QuickSelectionOption[] = [
    { key: 'today', label: 'Today', getValue: () => createRange(today) },
    { key: 'yesterday', label: 'Yesterday', getValue: () => createRange(daysAgo(1), daysAgo(1)) },
    { key: 'last30days', label: 'Last 30 days', getValue: () => createRange(daysAgo(29)) },
    {
      key: 'thismonth',
      label: 'This month',
      getValue: () => createRange(new Date(today.getFullYear(), today.getMonth(), 1)),
    },
    { key: 'last180days', label: 'Last 6 months', getValue: () => createRange(daysAgo(179)) },
  ]
  return opts.map((o) => ({ ...o, isEnabled: true }))
})

const isDateSelectable = (date: Date): boolean =>
  isWithinBounds(date, props.minNavigation, props.maxNavigation)
const isDateInSelectedRange = (date: Date): boolean =>
  !!(
    selectedRange.value.start &&
    selectedRange.value.end &&
    date >= selectedRange.value.start &&
    date <= selectedRange.value.end
  )
const isRangeStart = (date: Date): boolean =>
  !!(selectedRange.value.start && isSameDay(date, selectedRange.value.start))
const isRangeEnd = (date: Date): boolean =>
  !!(selectedRange.value.end && isSameDay(date, selectedRange.value.end))
const isSingleSelected = (date: Date): boolean =>
  !!(selectedSingle.value && isSameDay(date, selectedSingle.value))

const getDayClasses = (day: DayData): string => {
  if (!day.isCurrentMonth || !isDateSelectable(day.date)) {
    return 'text-gray-400 cursor-not-allowed'
  }
  let cls = 'text-gray-900 hover:bg-primary-300'
  if (day.isToday) cls += ' border border-primary-300'
  if (isSingleMode.value) {
    if (isSingleSelected(day.date)) cls += ' bg-primary-700 text-white'
  } else {
    if (isRangeStart(day.date) || isRangeEnd(day.date)) cls += ' bg-primary-700 text-white'
    else if (isDateInSelectedRange(day.date)) cls += ' bg-primary-300 text-white'
  }
  return cls
}

const handleDateClick = (date: Date): void => {
  if (!isDateSelectable(date)) return
  if (isSingleMode.value) {
    selectedSingle.value = date
    emit('update:modelValue', formatDate(date, 'YMD'))
    isOpen.value = false
  } else {
    if (!selectedRange.value.start || (selectedRange.value.start && selectedRange.value.end)) {
      selectedRange.value = { start: date, end: null }
    } else if (selectedRange.value.start && !selectedRange.value.end) {
      const start = date < selectedRange.value.start ? date : selectedRange.value.start
      const end = date < selectedRange.value.start ? selectedRange.value.start : date
      selectedRange.value = { start, end }
      emit('update:modelValue', [formatDate(start, 'YMD'), formatDate(end, 'YMD')])
      isOpen.value = false
    }
  }
}

const clearSelection = (): void => {
  if (isSingleMode.value) selectedSingle.value = null
  else selectedRange.value = { start: null, end: null }
  emit('update:modelValue', null)
  isOpen.value = false
}

const handleQuickSelection = (option: QuickSelectionOption): void => {
  const value = option.getValue()
  selectedRange.value = { start: new Date(value[0]), end: new Date(value[1]) }
  emit('update:modelValue', value)
  isOpen.value = false
}

const getQuickSelectionClasses = (option: QuickSelectionOption): string => {
  const value = option.getValue()
  const current = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  const selected = current[0] === value[0] && current[1] === value[1]
  return selected
    ? 'bg-primary-700 text-white font-medium shadow-sm'
    : 'text-gray-700 hover:bg-primary-300 hover:text-primary-700'
}

const togglePicker = (): void => {
  isOpen.value = !isOpen.value
}
const closePicker = (): void => {
  isOpen.value = false
}

const navigate = (dir: 'previous' | 'next'): void => {
  const canMove = dir === 'previous' ? canNavigatePrevious.value : canNavigateNext.value
  if (!canMove) return
  const offset = dir === 'previous' ? -1 : 1
  currentDate.value = new Date(
    currentDate.value.getFullYear(),
    currentDate.value.getMonth() + offset,
    1
  )
}

const handleClickOutside = (e: Event): void => {
  const t = e.target as Node
  if (
    dropdown.value &&
    !dropdown.value.contains(t) &&
    datePickerRef.value &&
    !datePickerRef.value.contains(t)
  ) {
    closePicker()
  }
}

const handleKeyDown = (e: KeyboardEvent): void => {
  if (!isOpen.value) return
  if (e.key === 'Escape') {
    e.preventDefault()
    closePicker()
  }
}

watch(
  () => props.modelValue,
  (v) => {
    if (isSingleMode.value) {
      if (typeof v === 'string' && v) {
        const p = parseModelDate(v)
        selectedSingle.value = p ? startOfDay(p) : null
      } else selectedSingle.value = null
    } else {
      if (!v || !Array.isArray(v)) {
        selectedRange.value = { start: null, end: null }
        return
      }
      if (v.length >= 2) {
        const s = parseModelDate(v[0]),
          e = parseModelDate(v[1])
        selectedRange.value = { start: s ? startOfDay(s) : null, end: e ? startOfDay(e) : null }
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

const displayValue = computed(() => {
  if (isSingleMode.value) return selectedSingle.value ? formatDate(selectedSingle.value, 'DMY') : ''
  if (selectedRange.value.start && selectedRange.value.end)
    return `${formatDate(selectedRange.value.start, 'DMY')} - ${formatDate(selectedRange.value.end, 'DMY')}`
  return ''
})
</script>
