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
      :class="showQuickSelection ? 'w-md' : 'w-xs'"
    >
      <div class="flex">
        <!-- Quick Selection Sidebar (only for date range picker) -->
        <div
          v-if="showQuickSelection"
          class="w-36 border-r border-gray-100 bg-gradient-to-b from-gray-50 to-gray-100 shadow-sm"
        >
          <div class="p-2">
            <div
              class="bg-primary-100 text-primary-700 mb-2 rounded px-2 py-1 text-xs font-medium tracking-wide shadow-sm"
            >
              Quick Select
            </div>
            <div class="space-y-1">
              <button
                v-for="option in quickSelectionOptions"
                :key="option.key"
                @click="!option.disabled && handleQuickSelection(option)"
                :disabled="option.disabled"
                :class="getQuickSelectionClasses(option)"
                class="hover:bg-primary-100 w-full cursor-pointer rounded px-2 py-1.5 text-left text-xs transition-colors disabled:cursor-not-allowed disabled:opacity-50 disabled:hover:bg-transparent"
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
            <div class="mb-2 grid grid-cols-7 gap-2">
              <div
                v-for="day in DAY_NAMES"
                :key="day"
                class="text-center text-xs font-medium text-gray-500"
              >
                {{ day }}
              </div>
            </div>

            <!-- Date grid -->
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

          <!-- Footer -->
          <div class="flex justify-end border-t border-gray-100 p-3">
            <button
              @click="clearSelection"
              class="rounded bg-gray-100 px-3 py-1 text-sm text-gray-700 transition-colors hover:bg-gray-200"
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

// Types
interface DatePickerProps {
  modelValue?: (string | null)[] | null
  placeholder?: string
  view?: 'date' | 'month'
  range?: boolean // keep for backward compatibility, always true for range selection
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
  disabled?: boolean
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
  placeholder: 'Select date range',
  view: 'date',
  range: true,
  minDate: null,
  maxDate: null,
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
const actualMode = computed(() => 'range') // Always range mode now
const isMonthView = computed(() => props.view === 'month')

const displayMonth = computed(() => MONTH_NAMES[currentDate.value.getMonth()])
const displayYear = computed(() => currentDate.value.getFullYear())

const canNavigatePrevious = computed(() => {
  if (!props.minDate) return false

  if (isMonthView.value) {
    // For month view: check if we can navigate to previous year
    const prevYear = currentDate.value.getFullYear() - 1

    // Parse minDate to get minimum allowed year/month
    let minYear: number
    if (props.minDate.length === 7) {
      // YYYY-MM format
      const [year] = props.minDate.split('-').map(Number)
      minYear = year
    } else {
      // YYYY-MM-DD format
      const minDateObj = new Date(props.minDate)
      minYear = minDateObj.getFullYear()
    }

    return prevYear >= minYear
  } else {
    // For date view: check if we can navigate to previous month
    const minDate = new Date(props.minDate)
    const lastDayOfPrevMonth = new Date(
      currentDate.value.getFullYear(),
      currentDate.value.getMonth(),
      0
    )
    return lastDayOfPrevMonth >= minDate
  }
})

const canNavigateNext = computed(() => {
  if (!props.maxDate) return false

  if (isMonthView.value) {
    // For month view: check if we can navigate to next year
    const nextYear = currentDate.value.getFullYear() + 1

    // Parse maxDate to get maximum allowed year/month
    let maxYear: number
    if (props.maxDate.length === 7) {
      // YYYY-MM format
      const [year] = props.maxDate.split('-').map(Number)
      maxYear = year
    } else {
      // YYYY-MM-DD format
      const maxDateObj = new Date(props.maxDate)
      maxYear = maxDateObj.getFullYear()
    }

    return nextYear <= maxYear
  } else {
    // For date view: check if we can navigate to next month
    const maxDate = new Date(props.maxDate)
    const firstDayOfNextMonth = new Date(
      currentDate.value.getFullYear(),
      currentDate.value.getMonth() + 1,
      1
    )
    return firstDayOfNextMonth <= new Date(maxDate.getFullYear(), maxDate.getMonth() + 1, 1)
  }
})

const displayValue = computed(() => {
  if (isMonthView.value) {
    if (actualMode.value === 'range') {
      if (selectedRange.value?.start && selectedRange.value?.end) {
        return `${formatMonthForDisplay(selectedRange.value.start)} - ${formatMonthForDisplay(selectedRange.value.end)}`
      } else if (selectedRange.value?.start) {
        return formatMonthForDisplay(selectedRange.value.start)
      }
      return ''
    } else {
      return selectedDate.value ? formatMonthForDisplay(selectedDate.value) : ''
    }
  } else {
    if (actualMode.value === 'range') {
      if (selectedRange.value?.start && selectedRange.value?.end) {
        return `${formatDateForDisplay(selectedRange.value.start)} - ${formatDateForDisplay(selectedRange.value.end)}`
      } else if (selectedRange.value?.start) {
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
  return !isMonthView.value // Show quick selection for date view only
})

const quickSelectionOptions = computed((): QuickSelectionOption[] => {
  if (!props.minDate || !props.maxDate) return []

  const minDate = new Date(props.minDate)
  const maxDate = new Date(props.maxDate)
  const today = new Date()

  const options: QuickSelectionOption[] = []

  // Helper to check if a date range is valid
  const isRangeValid = (startDate: Date, endDate: Date): boolean => {
    return startDate >= minDate && endDate <= maxDate
  }

  // Today
  if (isRangeValid(today, today)) {
    options.push({
      key: 'today',
      label: 'Today',
      getValue: () => [formatDate(today), formatDate(today)],
      disabled: false,
    })
  }

  // Yesterday
  const yesterday = new Date(today)
  yesterday.setDate(today.getDate() - 1)
  if (isRangeValid(yesterday, yesterday)) {
    options.push({
      key: 'yesterday',
      label: 'Yesterday',
      getValue: () => [formatDate(yesterday), formatDate(yesterday)],
      disabled: false,
    })
  }

  // Last 30 days
  const last30Days = new Date(today)
  last30Days.setDate(today.getDate() - 29)
  const isLast30Valid = isRangeValid(last30Days, today)
  options.push({
    key: 'last30days',
    label: 'Last 30 days',
    getValue: () => [formatDate(last30Days), formatDate(today)],
    disabled: !isLast30Valid,
  })

  // This month
  const startOfMonth = new Date(today.getFullYear(), today.getMonth(), 1)
  const isThisMonthValid = isRangeValid(startOfMonth, today)
  options.push({
    key: 'thismonth',
    label: 'This month',
    getValue: () => [formatDate(startOfMonth), formatDate(today)],
    disabled: !isThisMonthValid,
  })

  // Last 180 days
  const last180Days = new Date(today)
  last180Days.setDate(today.getDate() - 179)
  const isLast180Valid = isRangeValid(last180Days, today)
  options.push({
    key: 'last180days',
    label: '180 days',
    getValue: () => [formatDate(last180Days), formatDate(today)],
    disabled: !isLast180Valid,
  })

  return options
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
  if (!props.minDate || !props.maxDate) return false

  const checkDate = new Date(date)
  checkDate.setHours(0, 0, 0, 0) // Reset time to start of day

  const minDate = new Date(props.minDate)
  minDate.setHours(0, 0, 0, 0)

  const maxDate = new Date(props.maxDate)
  maxDate.setHours(0, 0, 0, 0)

  return checkDate >= minDate && checkDate <= maxDate
}

const isMonthSelectable = (year: number, month: number): boolean => {
  if (!props.minDate || !props.maxDate) return false

  // Create date for the month being checked
  const checkDate = new Date(year, month, 1)

  // Parse min and max dates - could be YYYY-MM or YYYY-MM-DD format
  const minDateStr = props.minDate
  const maxDateStr = props.maxDate

  let minDate: Date
  let maxDate: Date

  if (minDateStr.length === 7) {
    // YYYY-MM format
    const [minYear, minMonth] = minDateStr.split('-').map(Number)
    minDate = new Date(minYear, minMonth - 1, 1)
  } else {
    // YYYY-MM-DD format
    const minDateObj = new Date(minDateStr)
    minDate = new Date(minDateObj.getFullYear(), minDateObj.getMonth(), 1)
  }

  if (maxDateStr.length === 7) {
    // YYYY-MM format
    const [maxYear, maxMonth] = maxDateStr.split('-').map(Number)
    maxDate = new Date(maxYear, maxMonth - 1, 1)
  } else {
    // YYYY-MM-DD format
    const maxDateObj = new Date(maxDateStr)
    maxDate = new Date(maxDateObj.getFullYear(), maxDateObj.getMonth(), 1)
  }

  return checkDate >= minDate && checkDate <= maxDate
}

const isDateInSelectedRange = (date: Date): boolean => {
  if (!selectedRange.value?.start || !selectedRange.value?.end) return false
  return date >= selectedRange.value.start && date <= selectedRange.value.end
}

const isRangeStart = (date: Date): boolean => {
  return !!(selectedRange.value?.start && isSameDay(date, selectedRange.value.start))
}

const isRangeEnd = (date: Date): boolean => {
  return !!(selectedRange.value?.end && isSameDay(date, selectedRange.value.end))
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

    // Always handle as range mode
    if (isRangeStart(dayData.date) || isRangeEnd(dayData.date)) {
      classes.push('bg-primary-700 text-white')
    } else if (isDateInSelectedRange(dayData.date)) {
      classes.push('bg-primary-300 text-white')
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
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  return values.some((v) => v === monthString)
}

const isMonthInSelectedRange = (monthString: string): boolean => {
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  if (!values[0] || !values[1]) return false
  return monthString > values[0] && monthString < values[1]
}

const isMonthRangeStart = (monthString: string): boolean => {
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  return values[0] === monthString
}

const isMonthRangeEnd = (monthString: string): boolean => {
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  return values[1] === monthString
}

// Event Handlers
const handleDateClick = (date: Date): void => {
  if (!isDateSelectable(date)) return

  // Always handle as range selection
  if (!selectedRange.value?.start || (selectedRange.value?.start && selectedRange.value?.end)) {
    selectedRange.value = { start: date, end: null }
    rangeStart.value = date
  } else if (selectedRange.value?.start && !selectedRange.value?.end) {
    if (date >= selectedRange.value.start) {
      selectedRange.value.end = date
      emit('range-selected', { start: selectedRange.value.start, end: selectedRange.value.end })
      emit('update:modelValue', [
        formatDate(selectedRange.value.start),
        formatDate(selectedRange.value.end),
      ])
      isOpen.value = false
    } else {
      selectedRange.value = { start: date, end: null }
      rangeStart.value = date
    }
  }
}

const handleMonthClick = (monthData: MonthData): void => {
  if (!isMonthSelectable(displayYear.value, monthData.month)) return

  const selectedMonth = new Date(displayYear.value, monthData.month, 1)

  // Always handle as range selection
  if (!selectedRange.value?.start || (selectedRange.value?.start && selectedRange.value?.end)) {
    selectedRange.value = { start: selectedMonth, end: null }
    rangeStart.value = selectedMonth
  } else if (selectedRange.value?.start && !selectedRange.value?.end) {
    if (selectedMonth >= selectedRange.value.start) {
      selectedRange.value.end = selectedMonth
      emit('month-range-selected', {
        start: selectedRange.value.start,
        end: selectedRange.value.end,
      })
      emit('update:modelValue', [
        formatMonth(selectedRange.value.start.getFullYear(), selectedRange.value.start.getMonth()),
        formatMonth(selectedRange.value.end.getFullYear(), selectedRange.value.end.getMonth()),
      ])
      isOpen.value = false
    } else {
      selectedRange.value = { start: selectedMonth, end: null }
      rangeStart.value = selectedMonth
    }
  }
}

const clearSelection = (): void => {
  selectedRange.value = { start: null, end: null }
  rangeStart.value = null
  emit('update:modelValue', null)
  isOpen.value = false
}

const handleQuickSelection = (option: QuickSelectionOption): void => {
  if (option.disabled) return

  const value = option.getValue()
  // Always handle as range selection
  if (Array.isArray(value)) {
    const startDate = new Date(value[0])
    const endDate = new Date(value[1])
    selectedRange.value = { start: startDate, end: endDate }
    emit('range-selected', { start: startDate, end: endDate })
    emit('update:modelValue', value)
  }
  // Auto-close after quick selection
  isOpen.value = false
}

const getQuickSelectionClasses = (option: QuickSelectionOption): string => {
  if (option.disabled) {
    return 'text-gray-400 cursor-not-allowed'
  }

  const value = option.getValue()
  const currentValue = Array.isArray(props.modelValue) ? props.modelValue : [null, null]

  // Check if the option's value matches current selection
  const isSelected =
    Array.isArray(value) && currentValue[0] === value[0] && currentValue[1] === value[1]

  return isSelected
    ? 'bg-primary-700 text-white font-medium shadow-sm'
    : 'text-gray-700 hover:bg-primary-100 hover:text-primary-700'
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
    if (Array.isArray(newValue)) {
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
    } else {
      // Reset to empty range if not array
      selectedRange.value = { start: null, end: null }
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
