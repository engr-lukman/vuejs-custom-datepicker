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
      :class="{
        'w-xs': !showQuickSelection,
        'w-sm': showQuickSelection,
      }"
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
import { computed, ref, watch, onMounted, onUnmounted } from 'vue'

interface DateRange {
  start: Date | null
  end: Date | null
}

interface DatePickerProps {
  modelValue?: (string | null)[] | string | null
  placeholder?: string
  mode?: 'range' | 'single'
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

const startOfDay = (date: Date): Date => {
  const d = new Date(date)
  d.setHours(0, 0, 0, 0)
  return d
}

const formatDate = (date: Date, format: 'YMD' | 'DMY' | 'MonYYYY'): string => {
  const y = date.getFullYear()
  const m = String(date.getMonth() + 1).padStart(2, '0')
  const d = String(date.getDate()).padStart(2, '0')
  const monthName = MONTH_NAMES[date.getMonth()].slice(0, 3)
  if (format === 'YMD') return `${y}-${m}-${d}`
  if (format === 'DMY') return `${d}/${m}/${y}`
  return `${monthName} ${y}`
}

const parseModelDate = (value?: string | null): Date | null => {
  if (!value || typeof value !== 'string') return null
  const ymd = /^(\d{4})-(\d{2})-(\d{2})$/
  if (ymd.test(value)) {
    const [y, m, d] = value.split('-').map(Number)
    const dt = new Date(y, m - 1, d)
    return Number.isNaN(dt.getTime()) ? null : dt
  }
  const dt = new Date(value)
  return Number.isNaN(dt.getTime()) ? null : dt
}

const isSameDay = (a: Date, b: Date): boolean =>
  a.getFullYear() === b.getFullYear() &&
  a.getMonth() === b.getMonth() &&
  a.getDate() === b.getDate()

const isToday = (date: Date): boolean => isSameDay(date, new Date())

const isWithinBounds = (
  value: Date,
  minNavigation?: string | null,
  maxNavigation?: string | null
): boolean => {
  const minDate = minNavigation ? parseModelDate(minNavigation) : null
  const maxDate = maxNavigation ? parseModelDate(maxNavigation) : null
  const dateValue = startOfDay(value)
  if (minDate && dateValue < startOfDay(minDate)) return false
  if (maxDate && dateValue > startOfDay(maxDate)) return false
  return true
}

const props = withDefaults(defineProps<DatePickerProps>(), {
  placeholder: '',
  mode: 'range',
  minNavigation: null,
  maxNavigation: null,
})

const emit = defineEmits<{
  'update:modelValue': [value: (string | null)[] | string | null]
}>()

const isOpen = ref(false)
const datePickerRef = ref<HTMLInputElement | null>(null)
const dropdown = ref<HTMLElement | null>(null)
const selectedSingle = ref<Date | null>(null)
const currentDate = ref(new Date())
const selectedRange = ref<DateRange>({ start: null, end: null })

const isSingleMode = computed(() => props.mode === 'single')

const placeholderText = computed(() =>
  props.placeholder ? props.placeholder : isSingleMode.value ? 'Select date' : 'Select date range'
)

const displayMonth = computed(() => MONTH_NAMES[currentDate.value.getMonth()])
const displayYear = computed(() => currentDate.value.getFullYear())

const canNavigatePrevious = computed(() => {
  if (!props.minNavigation) return true
  const currentYear = currentDate.value.getFullYear()
  const currentMonth = currentDate.value.getMonth()
  const prevMonth = currentMonth === 0 ? 11 : currentMonth - 1
  const prevYear = currentMonth === 0 ? currentYear - 1 : currentYear
  const minDate = parseModelDate(props.minNavigation)
  if (!minDate) return true
  const minYear = minDate.getFullYear()
  const minMonth = minDate.getMonth()
  return prevYear > minYear || (prevYear === minYear && prevMonth >= minMonth)
})

const canNavigateNext = computed(() => {
  if (!props.maxNavigation) return true
  const currentYear = currentDate.value.getFullYear()
  const currentMonth = currentDate.value.getMonth()
  const nextMonth = currentMonth === 11 ? 0 : currentMonth + 1
  const nextYear = currentMonth === 11 ? currentYear + 1 : currentYear
  const maxDate = parseModelDate(props.maxNavigation)
  if (!maxDate) return true
  const maxYear = maxDate.getFullYear()
  const maxMonth = maxDate.getMonth()
  return nextYear < maxYear || (nextYear === maxYear && nextMonth <= maxMonth)
})

const displayValue = computed(() => {
  if (isSingleMode.value) {
    return selectedSingle.value ? formatDate(selectedSingle.value, 'DMY') : ''
  }
  if (selectedRange.value?.start && selectedRange.value?.end) {
    return `${formatDate(selectedRange.value.start, 'DMY')} - ${formatDate(selectedRange.value.end, 'DMY')}`
  }
  return ''
})

const calendarDays = computed((): DayData[] => {
  const year = currentDate.value.getFullYear()
  const month = currentDate.value.getMonth()
  const firstDay = new Date(year, month, 1)
  const lastDay = new Date(year, month + 1, 0)
  const firstDayOfWeek = firstDay.getDay()
  const daysInMonth = lastDay.getDate()
  const days: DayData[] = []
  const prevMonthLastDay = new Date(year, month, 0).getDate()
  for (let i = firstDayOfWeek - 1; i >= 0; i--) {
    const day = prevMonthLastDay - i
    const date = new Date(year, month - 1, day)
    days.push({
      date,
      day,
      month: month - 1,
      year,
      isCurrentMonth: false,
      isToday: isToday(date),
      dateString: `prev-${formatDate(date, 'YMD')}`,
    })
  }
  for (let day = 1; day <= daysInMonth; day++) {
    const date = new Date(year, month, day)
    days.push({
      date,
      day,
      month,
      year,
      isCurrentMonth: true,
      isToday: isToday(date),
      dateString: `curr-${formatDate(date, 'YMD')}`,
    })
  }
  const remainingSlots = 42 - days.length
  for (let day = 1; day <= remainingSlots; day++) {
    const date = new Date(year, month + 1, day)
    days.push({
      date,
      day,
      month: month + 1,
      year,
      isCurrentMonth: false,
      isToday: isToday(date),
      dateString: `next-${formatDate(date, 'YMD')}`,
    })
  }
  return days
})

const showQuickSelection = computed(() => !isSingleMode.value)

const quickSelectionOptions = computed((): QuickSelectionOption[] => {
  const today = new Date()
  const createDateRange = (start: Date, end: Date = today): string[] => [
    formatDate(start, 'YMD'),
    formatDate(end, 'YMD'),
  ]
  const daysAgo = (days: number): Date => {
    const date = new Date(today)
    date.setDate(today.getDate() - days)
    return date
  }
  const options = [
    {
      key: 'today',
      label: 'Today',
      getValue: () => createDateRange(today),
    },
    {
      key: 'yesterday',
      label: 'Yesterday',
      getValue: () => {
        const yesterday = daysAgo(1)
        return createDateRange(yesterday, yesterday)
      },
    },
    {
      key: 'last30days',
      label: 'Last 30 days',
      getValue: () => createDateRange(daysAgo(29)),
    },
    {
      key: 'thismonth',
      label: 'This month',
      getValue: () => createDateRange(new Date(today.getFullYear(), today.getMonth(), 1)),
    },
    {
      key: 'last180days',
      label: 'Last 6 months',
      getValue: () => createDateRange(daysAgo(179)),
    },
  ]
  return options
    .filter((option) => {
      if (!props.minNavigation) return true
      const [startDateStr] = option.getValue()
      return isWithinBounds(new Date(startDateStr), props.minNavigation, props.maxNavigation)
    })
    .map((option) => ({ ...option, isEnabled: true }))
})

const isDateSelectable = (date: Date): boolean =>
  isWithinBounds(date, props.minNavigation, props.maxNavigation)

const isDateInSelectedRange = (date: Date): boolean =>
  !!(
    selectedRange.value?.start &&
    selectedRange.value?.end &&
    date >= selectedRange.value.start &&
    date <= selectedRange.value.end
  )

const isRangeStart = (date: Date): boolean =>
  !!(selectedRange.value?.start && isSameDay(date, selectedRange.value.start))

const isRangeEnd = (date: Date): boolean =>
  !!(selectedRange.value?.end && isSameDay(date, selectedRange.value.end))

const isSingleSelected = (date: Date): boolean =>
  !!(selectedSingle.value && isSameDay(date, selectedSingle.value))

const getDayClasses = (dayData: DayData): string => {
  if (!dayData.isCurrentMonth || !isDateSelectable(dayData.date)) {
    return 'text-gray-400 cursor-not-allowed'
  }
  let classes = 'text-gray-900 hover:bg-primary-300'
  if (dayData.isToday) classes += ' border border-primary-300'
  if (isSingleMode.value) {
    if (isSingleSelected(dayData.date)) classes += ' bg-primary-700 text-white'
  } else {
    if (isRangeStart(dayData.date) || isRangeEnd(dayData.date)) {
      classes += ' bg-primary-700 text-white'
    } else if (isDateInSelectedRange(dayData.date)) {
      classes += ' bg-primary-300 text-white'
    }
  }
  return classes
}

const handleDateClick = (date: Date): void => {
  if (!isDateSelectable(date)) return
  if (isSingleMode.value) {
    selectedSingle.value = date
    emit('update:modelValue', formatDate(date, 'YMD'))
    isOpen.value = false
  } else {
    if (!selectedRange.value?.start || (selectedRange.value?.start && selectedRange.value?.end)) {
      selectedRange.value = { start: date, end: null }
    } else if (selectedRange.value?.start && !selectedRange.value?.end) {
      const startDate = date < selectedRange.value.start ? date : selectedRange.value.start
      const endDate = date < selectedRange.value.start ? selectedRange.value.start : date
      selectedRange.value = { start: startDate, end: endDate }
      emit('update:modelValue', [formatDate(startDate, 'YMD'), formatDate(endDate, 'YMD')])
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

const handleQuickSelection = (option: QuickSelectionOption): void => {
  const value = option.getValue()
  const startDate = new Date(value[0])
  const endDate = new Date(value[1])
  selectedRange.value = { start: startDate, end: endDate }
  emit('update:modelValue', value)
  isOpen.value = false
}

const getQuickSelectionClasses = (option: QuickSelectionOption): string => {
  const value = option.getValue()
  const currentValue = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  const isSelected = currentValue[0] === value[0] && currentValue[1] === value[1]
  return isSelected
    ? 'bg-primary-700 text-white font-medium shadow-sm'
    : 'text-gray-700 hover:bg-primary-300 hover:text-primary-700'
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
  const monthOffset = direction === 'previous' ? -1 : 1
  currentDate.value = new Date(
    currentDate.value.getFullYear(),
    currentDate.value.getMonth() + monthOffset,
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
