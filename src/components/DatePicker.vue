<template>
  <div class="relative">
    <!-- Input Field -->
    <input
      ref="inputElement"
      :value="displayValue"
      :placeholder="placeholder"
      readonly
      @click="togglePicker"
      @keydown.enter="togglePicker"
      @keydown.space.prevent="togglePicker"
      @keydown.escape="closePicker"
      :aria-expanded="isOpen"
      aria-haspopup="dialog"
      class="focus:ring-primary-500 focus:border-primary-500 w-full cursor-pointer rounded-md border border-gray-300 bg-white px-3 py-2 pr-10 text-sm text-gray-900 placeholder-gray-500 transition-colors duration-200 hover:border-gray-400 focus:ring-2 focus:outline-none"
    />

    <!-- Calendar Icon -->
    <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-3">
      <svg class="h-4 w-4 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"
        />
      </svg>
    </div>

    <!-- Dropdown -->
    <div
      v-if="isOpen"
      ref="dropdown"
      class="absolute top-full left-0 z-50 mt-1 max-w-[calc(100vw-1rem)] rounded-lg border border-gray-200 bg-white shadow-lg"
      :class="showQuickSelection ? 'w-96' : 'w-80'"
    >
      <div class="flex">
        <!-- Quick Selection Sidebar (only for date view and range mode) -->
        <div v-if="showQuickSelection" class="w-36 border-r border-gray-100 bg-gray-50">
          <div class="p-2">
            <div class="mb-2 text-xs font-medium tracking-wide text-gray-600 uppercase">
              Quick Select
            </div>
            <div class="space-y-1">
              <button
                v-for="option in quickSelectionOptions"
                :key="option.key"
                @click="handleQuickSelection(option)"
                :class="getQuickSelectionClasses(option)"
                class="hover:bg-primary-100 w-full rounded px-2 py-1.5 text-left text-xs transition-colors"
              >
                {{ option.label }}
              </button>
            </div>
          </div>
        </div>

        <!-- Main Calendar Content -->
        <div class="flex-1">
          <!-- Header -->
          <div class="flex items-center justify-between border-b border-gray-100 p-3">
            <button
              @click="navigatePrevious"
              :disabled="!canNavigatePrevious"
              class="cursor-pointer rounded p-1 transition-colors hover:bg-gray-100 disabled:cursor-not-allowed disabled:opacity-50"
            >
              <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M15 19l-7-7 7-7"
                />
              </svg>
            </button>

            <h2 class="text-sm font-medium text-gray-900">
              {{ isMonthView ? displayYear : `${displayMonth} ${displayYear}` }}
            </h2>

            <button
              @click="navigateNext"
              :disabled="!canNavigateNext"
              class="cursor-pointer rounded p-1 transition-colors hover:bg-gray-100 disabled:cursor-not-allowed disabled:opacity-50"
            >
              <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M9 5l7 7-7 7"
                />
              </svg>
            </button>
          </div>

          <!-- Month View -->
          <div v-if="isMonthView" class="p-3">
            <div class="grid grid-cols-3 gap-2">
              <button
                v-for="monthData in monthGrid"
                :key="monthData.month"
                :disabled="!isMonthSelectable(displayYear, monthData.month)"
                @click="handleMonthClick(monthData)"
                :class="getMonthClasses(monthData)"
                class="relative cursor-pointer rounded px-3 py-2 text-sm transition-colors hover:bg-gray-100 disabled:cursor-not-allowed disabled:opacity-50"
              >
                {{ monthData.shortName }}
              </button>
            </div>
          </div>

          <!-- Date View -->
          <div v-else class="p-3">
            <!-- Day headers -->
            <div class="mb-2 grid grid-cols-7 gap-1">
              <div
                v-for="day in DAY_NAMES"
                :key="day"
                class="text-center text-xs font-medium text-gray-500"
              >
                {{ day }}
              </div>
            </div>

            <!-- Date grid -->
            <div class="grid grid-cols-7 gap-1">
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

          <!-- Footer -->
          <div class="flex justify-between border-t border-gray-100 p-3">
            <div class="flex items-center space-x-2">
              <svg
                class="h-4 w-4 text-gray-400"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"
                />
              </svg>
            </div>
            <div class="flex space-x-2">
              <button
                @click="clearSelection"
                class="rounded bg-gray-100 px-3 py-1 text-sm text-gray-700 transition-colors hover:bg-gray-200"
              >
                Cancel
              </button>
              <button
                v-if="hasValidSelection"
                @click="confirmSelection"
                class="bg-primary-700 hover:bg-primary-800 rounded px-3 py-1 text-sm text-white transition-colors"
              >
                Select
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, watch, onMounted, onUnmounted } from 'vue'

// Types
interface DatePickerProps {
  modelValue?: string | (string | null)[] | null
  placeholder?: string
  mode?: 'single' | 'range'
  view?: 'date' | 'month'
  range?: boolean // backward compatibility
  minDate?: string | null
  maxDate?: string | null
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

interface MonthData {
  month: number
  name: string
  shortName: string
  date: string
  isCurrentMonth: boolean
}

interface QuickSelectionOption {
  key: string
  label: string
  getValue: () => string | string[]
}

// Constants
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

// Utility Functions
const isToday = (date: Date): boolean => {
  try {
    const today = new Date()
    return (
      date.getFullYear() === today.getFullYear() &&
      date.getMonth() === today.getMonth() &&
      date.getDate() === today.getDate()
    )
  } catch {
    return false
  }
}

const isCurrentMonth = (year: number, month: number): boolean => {
  try {
    const today = new Date()
    return year === today.getFullYear() && month === today.getMonth()
  } catch {
    return false
  }
}

const formatDate = (date: Date): string => {
  try {
    return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`
  } catch {
    return ''
  }
}

const formatDateForDisplay = (date: Date): string => {
  try {
    return `${String(date.getDate()).padStart(2, '0')}/${String(date.getMonth() + 1).padStart(2, '0')}/${date.getFullYear()}`
  } catch {
    return ''
  }
}

const formatMonth = (year: number, month: number): string => {
  try {
    return `${year}-${String(month + 1).padStart(2, '0')}`
  } catch {
    return ''
  }
}

// Props and Emits
const props = withDefaults(defineProps<DatePickerProps>(), {
  placeholder: 'Select date',
  mode: 'single',
  view: 'date',
  range: false,
  minDate: (() => {
    const date = new Date()
    date.setDate(date.getDate() - 179) // 179 days before + today = 180 days total
    return date.toISOString().split('T')[0]
  })(),
  maxDate: (() => {
    const date = new Date()
    return date.toISOString().split('T')[0]
  })(),
})

const emit = defineEmits<{
  'update:modelValue': [value: string | (string | null)[] | null]
  'date-selected': [date: Date]
  'range-selected': [range: { start: Date; end: Date }]
  'month-selected': [date: Date]
  'month-range-selected': [range: { start: Date; end: Date }]
}>()

// Refs
const inputElement = ref<HTMLInputElement | null>(null)
const dropdown = ref<HTMLElement | null>(null)
const isOpen = ref(false)
const currentDate = ref(new Date())
const selectedDate = ref<Date | null>(null)
const selectedRange = ref<{ start: Date | null; end: Date | null }>({ start: null, end: null })
const rangeStart = ref<Date | null>(null)

// Computed Properties
const actualMode = computed(() => (props.range ? 'range' : props.mode))
const isMonthView = computed(() => props.view === 'month')

const displayMonth = computed(() => MONTH_NAMES[currentDate.value.getMonth()])
const displayYear = computed(() => currentDate.value.getFullYear())

const canNavigatePrevious = computed(() => {
  const today = new Date()

  if (isMonthView.value) {
    // For month view: check if we can navigate to previous year
    // Allow navigation if the previous year contains any of the valid months (current + previous 5)
    const prevYear = currentDate.value.getFullYear() - 1
    const currentMonth = today.getMonth()

    // Check if previous year has any valid months
    // Valid months: current month + previous 5 months
    const minValidMonth = currentMonth - 5

    // If minValidMonth is negative, it means we need to check previous year
    if (minValidMonth < 0) {
      // Some valid months are in the previous year
      return true
    } else {
      // All valid months are in current year or later
      return prevYear >= today.getFullYear()
    }
  } else {
    // For date view: check if we can navigate to previous month
    // Calculate minimum allowed date (179 days before today)
    const minDate = new Date(today)
    minDate.setDate(today.getDate() - 179)

    // Allow navigation if the previous month contains any selectable dates
    const lastDayOfPrevMonth = new Date(
      currentDate.value.getFullYear(),
      currentDate.value.getMonth(),
      0
    )
    return lastDayOfPrevMonth >= minDate
  }
})

const canNavigateNext = computed(() => {
  const today = new Date()

  if (isMonthView.value) {
    // For month view: check if we can navigate to next year
    const nextYear = currentDate.value.getFullYear() + 1

    // Can navigate to next year only if it contains valid months
    // Valid months are current month + previous 5 months
    // Since we only allow past months, we can't navigate to future years
    return nextYear <= today.getFullYear()
  } else {
    // For date view: check if we can navigate to next month
    const nextMonth = new Date(currentDate.value.getFullYear(), currentDate.value.getMonth() + 1, 1)

    // Can navigate to next month only if current date is today or earlier
    return nextMonth <= new Date(today.getFullYear(), today.getMonth() + 1, 1)
  }
})

const displayValue = computed(() => {
  if (isMonthView.value) {
    if (actualMode.value === 'range') {
      if (selectedRange.value.start && selectedRange.value.end) {
        return `${formatMonthForDisplay(selectedRange.value.start)} - ${formatMonthForDisplay(selectedRange.value.end)}`
      } else if (selectedRange.value.start) {
        return formatMonthForDisplay(selectedRange.value.start)
      }
      return ''
    } else {
      return selectedDate.value ? formatMonthForDisplay(selectedDate.value) : ''
    }
  } else {
    if (actualMode.value === 'range') {
      if (selectedRange.value.start && selectedRange.value.end) {
        return `${formatDateForDisplay(selectedRange.value.start)} - ${formatDateForDisplay(selectedRange.value.end)}`
      } else if (selectedRange.value.start) {
        return formatDateForDisplay(selectedRange.value.start)
      }
      return ''
    } else {
      return selectedDate.value ? formatDateForDisplay(selectedDate.value) : ''
    }
  }
})

const calendarDays = computed((): DayData[] => {
  const year = currentDate.value.getFullYear()
  const month = currentDate.value.getMonth()
  const firstDay = new Date(year, month, 1)
  const lastDay = new Date(year, month + 1, 0)
  const firstDayOfWeek = firstDay.getDay()
  const daysInMonth = lastDay.getDate()

  const days: DayData[] = []

  // Previous month days
  const prevMonth = new Date(year, month - 1, 0)
  for (let i = firstDayOfWeek - 1; i >= 0; i--) {
    const day = prevMonth.getDate() - i
    const date = new Date(year, month - 1, day)
    days.push({
      date,
      day,
      month: month - 1,
      year,
      isCurrentMonth: false,
      isToday: isToday(date),
      dateString: `prev-${formatDate(date)}`,
    })
  }

  // Current month days
  for (let day = 1; day <= daysInMonth; day++) {
    const date = new Date(year, month, day)
    days.push({
      date,
      day,
      month,
      year,
      isCurrentMonth: true,
      isToday: isToday(date),
      dateString: `curr-${formatDate(date)}`,
    })
  }

  // Next month days
  const remainingDays = 42 - days.length
  for (let day = 1; day <= remainingDays; day++) {
    const date = new Date(year, month + 1, day)
    days.push({
      date,
      day,
      month: month + 1,
      year,
      isCurrentMonth: false,
      isToday: isToday(date),
      dateString: `next-${formatDate(date)}`,
    })
  }

  return days
})

const monthGrid = computed((): MonthData[] => {
  return MONTH_NAMES.map((name, index) => ({
    month: index,
    name,
    shortName: name.slice(0, 3),
    date: formatMonth(displayYear.value, index),
    isCurrentMonth: isCurrentMonth(displayYear.value, index),
  }))
})

const showQuickSelection = computed(() => {
  return !isMonthView.value && actualMode.value === 'range'
})

const quickSelectionOptions = computed((): QuickSelectionOption[] => {
  const today = new Date()
  const yesterday = new Date(today)
  yesterday.setDate(today.getDate() - 1)

  const startOfMonth = new Date(today.getFullYear(), today.getMonth(), 1)
  const startOfLastMonth = new Date(today.getFullYear(), today.getMonth() - 1, 1)
  const endOfLastMonth = new Date(today.getFullYear(), today.getMonth(), 0)

  const last30Days = new Date(today)
  last30Days.setDate(today.getDate() - 29)

  const last180Days = new Date(today)
  last180Days.setDate(today.getDate() - 179)

  return [
    {
      key: 'today',
      label: 'Today',
      getValue: () => [formatDate(today), formatDate(today)],
    },
    {
      key: 'yesterday',
      label: 'Yesterday',
      getValue: () => [formatDate(yesterday), formatDate(yesterday)],
    },
    {
      key: 'last30days',
      label: 'Last 30 days',
      getValue: () => [formatDate(last30Days), formatDate(today)],
    },
    {
      key: 'thismonth',
      label: 'This month',
      getValue: () => [formatDate(startOfMonth), formatDate(today)],
    },
    {
      key: 'lastmonth',
      label: 'Last month',
      getValue: () => [formatDate(startOfLastMonth), formatDate(endOfLastMonth)],
    },
    {
      key: 'last180days',
      label: '180 days',
      getValue: () => [formatDate(last180Days), formatDate(today)],
    },
  ]
})

const hasValidSelection = computed(() => {
  if (actualMode.value === 'range') {
    return selectedRange.value.start && selectedRange.value.end
  } else {
    return selectedDate.value !== null
  }
})

// Helper Functions
const formatMonthForDisplay = (date: Date): string => {
  try {
    return `${MONTH_NAMES[date.getMonth()].slice(0, 3)} ${date.getFullYear()}`
  } catch {
    return ''
  }
}

const isSameDay = (date1: Date, date2: Date): boolean => {
  return (
    date1.getFullYear() === date2.getFullYear() &&
    date1.getMonth() === date2.getMonth() &&
    date1.getDate() === date2.getDate()
  )
}

const isDateSelectable = (date: Date): boolean => {
  const today = new Date()
  today.setHours(0, 0, 0, 0) // Reset time to start of day

  const checkDate = new Date(date)
  checkDate.setHours(0, 0, 0, 0) // Reset time to start of day

  // Calculate minimum date (179 days before today)
  const minDate = new Date(today)
  minDate.setDate(today.getDate() - 179)

  // Date is selectable if it's within the range: today and previous 179 days (total 180 days)
  return checkDate >= minDate && checkDate <= today
}

const isMonthSelectable = (year: number, month: number): boolean => {
  const today = new Date()
  const currentYear = today.getFullYear()
  const currentMonth = today.getMonth()

  // Create date for the month being checked
  const checkDate = new Date(year, month, 1)

  // Create date for 5 months ago from current month
  const minDate = new Date(currentYear, currentMonth - 5, 1)

  // Create date for current month
  const maxDate = new Date(currentYear, currentMonth, 1)

  // Month is selectable if it's within the range: current month + previous 5 months
  return checkDate >= minDate && checkDate <= maxDate
}

const isDateInSelectedRange = (date: Date): boolean => {
  if (actualMode.value !== 'range' || !selectedRange.value.start || !selectedRange.value.end)
    return false
  return date >= selectedRange.value.start && date <= selectedRange.value.end
}

const isRangeStart = (date: Date): boolean => {
  return !!(
    actualMode.value === 'range' &&
    selectedRange.value.start &&
    isSameDay(date, selectedRange.value.start)
  )
}

const isRangeEnd = (date: Date): boolean => {
  return !!(
    actualMode.value === 'range' &&
    selectedRange.value.end &&
    isSameDay(date, selectedRange.value.end)
  )
}

const isSelected = (date: Date): boolean => {
  if (actualMode.value === 'range') {
    return isRangeStart(date) || isRangeEnd(date)
  } else {
    return selectedDate.value ? isSameDay(date, selectedDate.value) : false
  }
}

const getDayClasses = (dayData: DayData): string => {
  const classes = []

  if (!dayData.isCurrentMonth) {
    classes.push('text-gray-400 cursor-not-allowed')
  } else if (!isDateSelectable(dayData.date)) {
    classes.push('text-gray-400 cursor-not-allowed')
  } else {
    classes.push('text-gray-900 hover:bg-gray-100')

    if (dayData.isToday) {
      classes.push('bg-primary-100 border border-primary-300')
    }

    if (actualMode.value === 'range') {
      if (isRangeStart(dayData.date) || isRangeEnd(dayData.date)) {
        classes.push('bg-primary-700 text-white')
      } else if (isDateInSelectedRange(dayData.date)) {
        classes.push('bg-primary-300 text-white')
      }
    } else if (isSelected(dayData.date)) {
      classes.push('bg-primary-700 text-white')
    }
  }

  return classes.join(' ')
}

const getMonthClasses = (monthData: MonthData): string => {
  if (!isMonthSelectable(displayYear.value, monthData.month)) {
    return 'text-gray-400 cursor-not-allowed'
  }

  const isSelected = isMonthSelected(monthData.date)
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
    return 'bg-primary-100 border border-primary-300 text-primary-800 font-medium'
  } else {
    return 'text-gray-700 hover:bg-gray-100'
  }
}

const isMonthSelected = (monthString: string): boolean => {
  if (actualMode.value === 'range') {
    const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
    return values.some((v) => v === monthString)
  }
  return props.modelValue === monthString
}

const isMonthInSelectedRange = (monthString: string): boolean => {
  if (actualMode.value !== 'range') return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  if (!values[0] || !values[1]) return false
  return monthString > values[0] && monthString < values[1]
}

const isMonthRangeStart = (monthString: string): boolean => {
  if (actualMode.value !== 'range') return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  return values[0] === monthString
}

const isMonthRangeEnd = (monthString: string): boolean => {
  if (actualMode.value !== 'range') return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  return values[1] === monthString
}

// Event Handlers
const handleDateClick = (date: Date): void => {
  if (!isDateSelectable(date)) return

  if (actualMode.value === 'range') {
    if (!selectedRange.value.start || (selectedRange.value.start && selectedRange.value.end)) {
      selectedRange.value = { start: date, end: null }
      rangeStart.value = date
    } else if (selectedRange.value.start && !selectedRange.value.end) {
      if (date >= selectedRange.value.start) {
        selectedRange.value.end = date
        emit('range-selected', { start: selectedRange.value.start, end: selectedRange.value.end })
        emit('update:modelValue', [
          selectedRange.value.start.toISOString().split('T')[0],
          selectedRange.value.end.toISOString().split('T')[0],
        ])
        isOpen.value = false
      } else {
        selectedRange.value = { start: date, end: null }
        rangeStart.value = date
      }
    }
  } else {
    selectedDate.value = date
    emit('date-selected', date)
    emit('update:modelValue', date.toISOString().split('T')[0])
    isOpen.value = false
  }
}

const handleMonthClick = (monthData: MonthData): void => {
  if (!isMonthSelectable(displayYear.value, monthData.month)) return

  const selectedMonth = new Date(displayYear.value, monthData.month, 1)

  if (actualMode.value === 'range') {
    if (!selectedRange.value.start || (selectedRange.value.start && selectedRange.value.end)) {
      selectedRange.value = { start: selectedMonth, end: null }
      rangeStart.value = selectedMonth
    } else if (selectedRange.value.start && !selectedRange.value.end) {
      if (selectedMonth >= selectedRange.value.start) {
        selectedRange.value.end = selectedMonth
        emit('month-range-selected', {
          start: selectedRange.value.start,
          end: selectedRange.value.end,
        })
        emit('update:modelValue', [
          formatMonth(
            selectedRange.value.start.getFullYear(),
            selectedRange.value.start.getMonth()
          ),
          formatMonth(selectedRange.value.end.getFullYear(), selectedRange.value.end.getMonth()),
        ])
        isOpen.value = false
      } else {
        selectedRange.value = { start: selectedMonth, end: null }
        rangeStart.value = selectedMonth
      }
    }
  } else {
    selectedDate.value = selectedMonth
    emit('month-selected', selectedMonth)
    emit('update:modelValue', formatMonth(displayYear.value, monthData.month))
    isOpen.value = false
  }
}

const clearSelection = (): void => {
  if (actualMode.value === 'range') {
    selectedRange.value = { start: null, end: null }
    rangeStart.value = null
  } else {
    selectedDate.value = null
  }
  emit('update:modelValue', null)
  isOpen.value = false
}

const handleQuickSelection = (option: QuickSelectionOption): void => {
  const value = option.getValue()
  if (Array.isArray(value)) {
    // Range selection
    const startDate = new Date(value[0])
    const endDate = new Date(value[1])
    selectedRange.value = { start: startDate, end: endDate }
    emit('range-selected', { start: startDate, end: endDate })
    emit('update:modelValue', value)
  } else {
    // Single date selection
    const date = new Date(value)
    selectedDate.value = date
    emit('date-selected', date)
    emit('update:modelValue', value)
  }
}

const getQuickSelectionClasses = (option: QuickSelectionOption): string => {
  const value = option.getValue()
  const isSelected = Array.isArray(props.modelValue)
    ? props.modelValue[0] === value[0] && props.modelValue[1] === value[1]
    : props.modelValue === value

  return isSelected ? 'bg-primary-700 text-white font-medium' : 'text-gray-700 hover:bg-primary-100'
}

const confirmSelection = (): void => {
  isOpen.value = false
}

const togglePicker = (): void => {
  isOpen.value = !isOpen.value
}

const closePicker = (): void => {
  isOpen.value = false
}

const navigatePrevious = (): void => {
  if (canNavigatePrevious.value) {
    if (isMonthView.value) {
      currentDate.value = new Date(currentDate.value.getFullYear() - 1, 0, 1)
    } else {
      currentDate.value = new Date(
        currentDate.value.getFullYear(),
        currentDate.value.getMonth() - 1,
        1
      )
    }
  }
}

const navigateNext = (): void => {
  if (canNavigateNext.value) {
    if (isMonthView.value) {
      currentDate.value = new Date(currentDate.value.getFullYear() + 1, 0, 1)
    } else {
      currentDate.value = new Date(
        currentDate.value.getFullYear(),
        currentDate.value.getMonth() + 1,
        1
      )
    }
  }
}

const handleClickOutside = (event: Event): void => {
  if (
    dropdown.value &&
    !dropdown.value.contains(event.target as Node) &&
    inputElement.value &&
    !inputElement.value.contains(event.target as Node)
  ) {
    isOpen.value = false
  }
}

// Watchers
watch(
  () => props.modelValue,
  (newValue) => {
    if (actualMode.value === 'range' && Array.isArray(newValue)) {
      if (isMonthView.value) {
        selectedRange.value = {
          start: newValue[0] ? new Date(newValue[0] + '-01') : null,
          end: newValue[1] ? new Date(newValue[1] + '-01') : null,
        }
      } else {
        selectedRange.value = {
          start: newValue[0] ? new Date(newValue[0]) : null,
          end: newValue[1] ? new Date(newValue[1]) : null,
        }
      }
    } else if (actualMode.value === 'single' && typeof newValue === 'string') {
      if (isMonthView.value) {
        selectedDate.value = new Date(newValue + '-01')
      } else {
        selectedDate.value = new Date(newValue)
      }
    }
  },
  { immediate: true }
)

// Lifecycle
onMounted(() => {
  document.addEventListener('click', handleClickOutside)
})

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside)
})
</script>
