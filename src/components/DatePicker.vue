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
      class="focus:ring-primary-500 focus:border-primary-500 w-full cursor-pointer rounded-md border border-gray-300 bg-white px-3 py-2 pr-10 text-sm text-gray-900 placeholder-gray-500 transition-colors duration-200 hover:border-gray-400 focus:ring-2 focus:outline-none"
    />

    <!-- Calendar Icon -->
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

    <!-- Dropdown Panel -->
    <div
      v-if="isOpen"
      ref="dropdown"
      role="dialog"
      aria-modal="false"
      aria-label="Date picker"
      class="absolute top-full left-0 z-50 mt-1 max-w-[calc(100vw-1rem)] rounded-lg border border-gray-200 bg-white shadow-lg"
      :class="showQuickSelection ? 'w-md' : 'w-xs'"
    >
      <div class="flex">
        <!-- Quick Selection Sidebar -->
        <div
          v-if="showQuickSelection"
          class="w-32 border-r border-gray-100 bg-gradient-to-b from-gray-50 to-gray-100"
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
                :aria-label="`Select ${option.label}`"
                class="hover:bg-primary-100 : w-full cursor-pointer rounded px-2 py-1.5 text-left text-xs transition-colors disabled:cursor-not-allowed disabled:text-gray-400"
              >
                {{ option.label }}
              </button>
            </div>
          </div>
        </div>

        <!-- Main Calendar Content -->
        <div class="flex-1">
          <!-- Calendar Header -->
          <div class="flex items-center justify-between border-b border-gray-100 p-3">
            <button
              type="button"
              @click="navigatePrevious"
              :disabled="!canNavigatePrevious"
              :aria-label="`Go to ${isMonthView ? 'previous year' : 'previous month'}`"
              class="focus:ring-primary-500 cursor-pointer rounded p-1 transition-colors hover:bg-gray-100 focus:ring-2 focus:outline-none disabled:cursor-not-allowed disabled:opacity-50"
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
              {{ isMonthView ? displayYear : `${displayMonth} ${displayYear}` }}
            </h2>

            <button
              type="button"
              @click="navigateNext"
              :disabled="!canNavigateNext"
              :aria-label="`Go to ${isMonthView ? 'next year' : 'next month'}`"
              class="focus:ring-primary-500 cursor-pointer rounded p-1 transition-colors hover:bg-gray-100 focus:ring-2 focus:outline-none disabled:cursor-not-allowed disabled:opacity-50"
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
  modelValue?: (string | null)[] | string | null
  placeholder?: string
  view?: 'date' | 'month'
  mode?: 'range' | 'single'
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
  getValue: () => string[]
  isEnabled?: boolean
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

// Utility Functions with improved error handling
const isToday = (date: Date): boolean => {
  try {
    const today = new Date()
    today.setHours(0, 0, 0, 0)
    const compareDate = new Date(date)
    compareDate.setHours(0, 0, 0, 0)
    return compareDate.getTime() === today.getTime()
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
    const year = date.getFullYear()
    const month = String(date.getMonth() + 1).padStart(2, '0')
    const day = String(date.getDate()).padStart(2, '0')
    return `${year}-${month}-${day}`
  } catch {
    return ''
  }
}

const formatDateForDisplay = (date: Date): string => {
  try {
    const day = String(date.getDate()).padStart(2, '0')
    const month = String(date.getMonth() + 1).padStart(2, '0')
    const year = date.getFullYear()
    return `${day}/${month}/${year}`
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
  placeholder: '',
  view: 'date',
  mode: 'range',
  minDate: null,
  maxDate: null,
})

const emit = defineEmits<{
  'update:modelValue': [value: (string | null)[] | string | null]
  'date-selected': [date: Date]
  'date-range-selected': [range: { start: Date; end: Date }]
  'month-selected': [date: Date]
  'month-range-selected': [range: { start: Date; end: Date }]
}>()

// Refs
const isOpen = ref(false)
const datePickerRef = ref<HTMLInputElement | null>(null)
const dropdown = ref<HTMLElement | null>(null)
const selectedSingle = ref<Date | null>(null)
const currentDate = ref(new Date())
const selectedRange = ref<{ start: Date | null; end: Date | null }>({ start: null, end: null })

// Computed Properties
const isMonthView = computed(() => props.view === 'month')
const isSingleMode = computed(() => props.mode === 'single')

const placeholderText = computed(() => {
  if (props.placeholder) return props.placeholder

  if (isSingleMode.value) {
    return isMonthView.value ? 'Select month' : 'Select date'
  } else {
    return isMonthView.value ? 'Select month range' : 'Select date range'
  }
})

const displayMonth = computed(() => MONTH_NAMES[currentDate.value.getMonth()])
const displayYear = computed(() => currentDate.value.getFullYear())

const canNavigatePrevious = computed(() => {
  if (!props.minDate) return true

  const currentYear = currentDate.value.getFullYear()
  const currentMonth = currentDate.value.getMonth()

  if (isMonthView.value) {
    // For month view: check if we can navigate to previous year
    const prevYear = currentYear - 1

    // Parse minDate to get minimum allowed year
    const minYear =
      props.minDate.length === 7
        ? parseInt(props.minDate.split('-')[0])
        : new Date(props.minDate).getFullYear()

    return prevYear >= minYear
  } else {
    // For date view: check if we can navigate to previous month
    const prevMonth = currentMonth === 0 ? 11 : currentMonth - 1
    const prevYear = currentMonth === 0 ? currentYear - 1 : currentYear

    const minDate = new Date(props.minDate)
    const minYear = minDate.getFullYear()
    const minMonth = minDate.getMonth()

    return prevYear > minYear || (prevYear === minYear && prevMonth >= minMonth)
  }
})

const canNavigateNext = computed(() => {
  if (!props.maxDate) return true

  const currentYear = currentDate.value.getFullYear()
  const currentMonth = currentDate.value.getMonth()

  if (isMonthView.value) {
    // For month view: check if we can navigate to next year
    const nextYear = currentYear + 1

    // Parse maxDate to get maximum allowed year
    const maxYear =
      props.maxDate.length === 7
        ? parseInt(props.maxDate.split('-')[0])
        : new Date(props.maxDate).getFullYear()

    return nextYear <= maxYear
  } else {
    // For date view: check if we can navigate to next month
    const nextMonth = currentMonth === 11 ? 0 : currentMonth + 1
    const nextYear = currentMonth === 11 ? currentYear + 1 : currentYear

    const maxDate = new Date(props.maxDate)
    const maxYear = maxDate.getFullYear()
    const maxMonth = maxDate.getMonth()

    return nextYear < maxYear || (nextYear === maxYear && nextMonth <= maxMonth)
  }
})

const displayValue = computed(() => {
  if (isSingleMode.value) {
    // Single mode
    if (selectedSingle.value) {
      return isMonthView.value
        ? formatMonthForDisplay(selectedSingle.value)
        : formatDateForDisplay(selectedSingle.value)
    }
    return ''
  } else {
    // Range mode
    if (isMonthView.value) {
      if (selectedRange.value?.start && selectedRange.value?.end) {
        return `${formatMonthForDisplay(selectedRange.value.start)} - ${formatMonthForDisplay(selectedRange.value.end)}`
      } else if (selectedRange.value?.start) {
        return formatMonthForDisplay(selectedRange.value.start)
      }
      return ''
    } else {
      if (selectedRange.value?.start && selectedRange.value?.end) {
        return `${formatDateForDisplay(selectedRange.value.start)} - ${formatDateForDisplay(selectedRange.value.end)}`
      } else if (selectedRange.value?.start) {
        return formatDateForDisplay(selectedRange.value.start)
      }
      return ''
    }
  }
})

const calendarDays = computed((): DayData[] => {
  const year = currentDate.value.getFullYear()
  const month = currentDate.value.getMonth()

  // Get first and last day of current month
  const firstDay = new Date(year, month, 1)
  const lastDay = new Date(year, month + 1, 0)
  const firstDayOfWeek = firstDay.getDay()
  const daysInMonth = lastDay.getDate()

  const days: DayData[] = []

  // Add previous month days to fill the first week
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
      dateString: `prev-${formatDate(date)}`,
    })
  }

  // Add current month days
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

  // Add next month days to fill remaining slots (ensuring 6 weeks = 42 days)
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
  // Show quick selection only for date view in range mode
  return !isMonthView.value && !isSingleMode.value
})

console.log('Min Date:', props.minDate)

const quickSelectionOptions = computed((): QuickSelectionOption[] => {
  const today = new Date()

  return [
    {
      key: 'today',
      label: 'Today',
      getValue: () => [formatDate(today), formatDate(today)],
      isEnabled: true,
    },
    {
      key: 'yesterday',
      label: 'Yesterday',
      getValue: () => {
        const yesterday = new Date(today)
        yesterday.setDate(today.getDate() - 1)
        return [formatDate(yesterday), formatDate(yesterday)]
      },
      isEnabled: true,
    },
    {
      key: 'last30days',
      label: 'Last 30 days',
      getValue: () => {
        const last30Days = new Date(today)
        last30Days.setDate(today.getDate() - 29)
        return [formatDate(last30Days), formatDate(today)]
      },
      isEnabled: true,
    },
    {
      key: 'thismonth',
      label: 'This month',
      getValue: () => {
        const startOfMonth = new Date(today.getFullYear(), today.getMonth(), 1)
        return [formatDate(startOfMonth), formatDate(today)]
      },
      isEnabled: true,
    },
    {
      key: 'last180days',
      label: '6 months',
      getValue: () => {
        const last180Days = new Date(today)
        last180Days.setDate(today.getDate() - 179)
        return [formatDate(last180Days), formatDate(today)]
      },
      isEnabled: props?.minDate
        ? (() => {
            const minDate = new Date(props.minDate as string)
            const last180Days = new Date(today)
            last180Days.setDate(today.getDate() - 179)
            return last180Days >= minDate
          })()
        : true,
    },
  ]
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
  // Allow all dates if no constraints are set
  if (!props.minDate && !props.maxDate) return true

  const checkDate = new Date(date)
  checkDate.setHours(0, 0, 0, 0) // Reset time to start of day

  let isValid = true

  if (props.minDate) {
    const minDate = new Date(props.minDate)
    minDate.setHours(0, 0, 0, 0)
    isValid = isValid && checkDate >= minDate
  }

  if (props.maxDate) {
    const maxDate = new Date(props.maxDate)
    maxDate.setHours(0, 0, 0, 0)
    isValid = isValid && checkDate <= maxDate
  }

  return isValid
}

const isMonthSelectable = (year: number, month: number): boolean => {
  // Allow all months if no constraints are set
  if (!props.minDate && !props.maxDate) return true

  // Create date for the month being checked (first day of month)
  const checkDate = new Date(year, month, 1)

  // Parse min and max dates
  const parseDate = (dateStr: string): Date => {
    if (dateStr.length === 7) {
      // YYYY-MM format
      const [year, month] = dateStr.split('-').map(Number)
      return new Date(year, month - 1, 1)
    } else {
      // YYYY-MM-DD format - use first day of that month
      const dateObj = new Date(dateStr)
      return new Date(dateObj.getFullYear(), dateObj.getMonth(), 1)
    }
  }

  let isValid = true

  if (props.minDate) {
    const minDate = parseDate(props.minDate)
    isValid = isValid && checkDate >= minDate
  }

  if (props.maxDate) {
    const maxDate = parseDate(props.maxDate)
    isValid = isValid && checkDate <= maxDate
  }

  return isValid
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

const isSingleSelected = (date: Date): boolean => {
  return !!(selectedSingle.value && isSameDay(date, selectedSingle.value))
}

const getDayClasses = (dayData: DayData): string => {
  const classes = []

  // Handle non-current month days and non-selectable dates
  if (!dayData.isCurrentMonth || !isDateSelectable(dayData.date)) {
    classes.push('text-gray-400 cursor-not-allowed')
    return classes.join(' ')
  }

  // Base classes for selectable dates
  classes.push('text-gray-900 hover:bg-gray-100')

  // Today indicator
  if (dayData.isToday) {
    classes.push('bg-primary-100 border border-primary-300')
  }

  if (isSingleMode.value) {
    // Single mode styling
    if (isSingleSelected(dayData.date)) {
      classes.push('bg-primary-700 text-white')
    }
  } else {
    // Range selection styling
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

  if (isSingleMode.value) {
    // Single mode styling
    if (isSelected) {
      return 'bg-primary-700 text-white font-medium shadow-md'
    } else if (monthData.isCurrentMonth) {
      return 'bg-primary-100 border border-primary-300 text-primary-800 font-medium'
    } else {
      return 'text-gray-700 hover:bg-gray-100'
    }
  } else {
    // Range mode styling
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
}

const isMonthSelected = (monthString: string): boolean => {
  if (isSingleMode.value) {
    const value = typeof props.modelValue === 'string' ? props.modelValue : null
    return value === monthString
  } else {
    const values = Array.isArray(props.modelValue) ? props.modelValue : []
    return values.includes(monthString)
  }
}

const isMonthInSelectedRange = (monthString: string): boolean => {
  if (isSingleMode.value) return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : []
  if (values.length < 2 || !values[0] || !values[1]) return false
  return monthString > values[0] && monthString < values[1]
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

// Event Handlers
const handleDateClick = (date: Date): void => {
  if (!isDateSelectable(date)) return

  if (isSingleMode.value) {
    // Single mode logic
    selectedSingle.value = date
    emit('date-selected', date)
    emit(
      'update:modelValue',
      isMonthView.value ? formatMonth(date.getFullYear(), date.getMonth()) : formatDate(date)
    )
    isOpen.value = false
  } else {
    // Range selection logic
    if (!selectedRange.value?.start || (selectedRange.value?.start && selectedRange.value?.end)) {
      // Start new range
      selectedRange.value = { start: date, end: null }
    } else if (selectedRange.value?.start && !selectedRange.value?.end) {
      // Complete the range
      if (date >= selectedRange.value.start) {
        selectedRange.value.end = date
        emit('date-range-selected', {
          start: selectedRange.value.start,
          end: selectedRange.value.end,
        })
        emit('update:modelValue', [
          formatDate(selectedRange.value.start),
          formatDate(selectedRange.value.end),
        ])
        isOpen.value = false
      } else {
        // Start new range if selected date is before start
        selectedRange.value = { start: date, end: null }
      }
    }
  }
}

const handleMonthClick = (monthData: MonthData): void => {
  if (!isMonthSelectable(displayYear.value, monthData.month)) return

  const selectedMonth = new Date(displayYear.value, monthData.month, 1)

  if (isSingleMode.value) {
    // Single mode logic
    selectedSingle.value = selectedMonth
    emit('month-selected', selectedMonth)
    emit('update:modelValue', formatMonth(selectedMonth.getFullYear(), selectedMonth.getMonth()))
    isOpen.value = false
  } else {
    // Range selection logic for months
    if (!selectedRange.value?.start || (selectedRange.value?.start && selectedRange.value?.end)) {
      // Start new range
      selectedRange.value = { start: selectedMonth, end: null }
    } else if (selectedRange.value?.start && !selectedRange.value?.end) {
      // Complete the range
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
        // Start new range if selected month is before start
        selectedRange.value = { start: selectedMonth, end: null }
      }
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
  emit('date-range-selected', { start: startDate, end: endDate })
  emit('update:modelValue', value)

  // Auto-close after quick selection
  isOpen.value = false
}

const getQuickSelectionClasses = (option: QuickSelectionOption): string => {
  const value = option.getValue()
  const currentValue = Array.isArray(props.modelValue) ? props.modelValue : [null, null]

  // Check if the option's value matches current selection
  const isSelected = currentValue[0] === value[0] && currentValue[1] === value[1]

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

  switch (event.key) {
    case 'Escape':
      event.preventDefault()
      closePicker()
      break
    case 'Tab':
      // Allow normal tab behavior to close picker when focus leaves
      if (!dropdown.value?.contains(event.target as Node)) {
        closePicker()
      }
      break
  }
}

// Optimized watcher with better performance
watch(
  () => props.modelValue,
  (newValue) => {
    if (isSingleMode.value) {
      // Single mode
      if (typeof newValue === 'string' && newValue) {
        try {
          const dateFormat = isMonthView.value ? '-01' : ''
          selectedSingle.value = new Date(newValue + dateFormat)
        } catch {
          selectedSingle.value = null
        }
      } else {
        selectedSingle.value = null
      }
    } else {
      // Range mode
      if (!newValue || !Array.isArray(newValue)) {
        selectedRange.value = { start: null, end: null }
        return
      }

      if (newValue.length >= 2) {
        try {
          const dateFormat = isMonthView.value ? '-01' : ''
          selectedRange.value = {
            start: newValue[0] ? new Date(newValue[0] + dateFormat) : null,
            end: newValue[1] ? new Date(newValue[1] + dateFormat) : null,
          }
        } catch {
          // Reset on invalid date format
          selectedRange.value = { start: null, end: null }
        }
      } else {
        selectedRange.value = { start: null, end: null }
      }
    }
  },
  { immediate: true }
)

// Lifecycle
onMounted(() => {
  document.addEventListener('click', handleClickOutside)
  document.addEventListener('keydown', handleKeyDown)
})

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside)
  document.removeEventListener('keydown', handleKeyDown)
})
</script>
