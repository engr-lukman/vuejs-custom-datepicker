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
                class="w-full cursor-pointer rounded px-2 py-1.5 text-left text-xs transition-colors"
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
              @click="navigate('previous')"
              :disabled="!canNavigatePrevious"
              :aria-label="`Go to ${isMonthView ? 'previous year' : 'previous month'}`"
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
              {{ isMonthView ? displayYear : `${displayMonth} ${displayYear}` }}
            </h2>

            <button
              type="button"
              @click="navigate('next')"
              :disabled="!canNavigateNext"
              :aria-label="`Go to ${isMonthView ? 'next year' : 'next month'}`"
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

          <!-- Month View -->
          <div v-if="isMonthView" class="p-3">
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
// ===== IMPORTS =====
import { computed, ref, watch, onMounted, onUnmounted } from 'vue'

// ===== INTERFACES AND TYPES =====
interface YearMonth {
  year: number
  month: number
}

interface DatePickerProps {
  modelValue?: (string | null)[] | string | null
  placeholder?: string
  view?: 'date' | 'month'
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

// ===== CONSTANTS =====
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

// ===== UTILITY FUNCTIONS =====
/** Normalizes a Date to start of day */
const startOfDay = (date: Date): Date => {
  const d = new Date(date)
  d.setHours(0, 0, 0, 0)
  return d
}

/** Parses model date string (YYYY-MM-DD or YYYY-MM) to Date */
const parseModelDate = (value?: string | null): Date | null => {
  if (!value || typeof value !== 'string') return null

  const ymd = /^(\d{4})-(\d{2})-(\d{2})$/
  const ym = /^(\d{4})-(\d{2})$/

  if (ymd.test(value)) {
    const [y, m, d] = value.split('-').map(Number)
    const dt = new Date(y, m - 1, d)
    return Number.isNaN(dt.getTime()) ? null : dt
  }

  if (ym.test(value)) {
    const [y, m] = value.split('-').map(Number)
    const dt = new Date(y, m - 1, 1)
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

/** Format a date to various string formats */
const formatDate = (date: Date, format: 'YMD' | 'DMY' | 'YM' | 'MonYYYY'): string => {
  const y = date.getFullYear()
  const m = String(date.getMonth() + 1).padStart(2, '0')
  const d = String(date.getDate()).padStart(2, '0')
  const monthName = MONTH_NAMES[date.getMonth()].slice(0, 3)

  switch (format) {
    case 'YMD':
      return `${y}-${m}-${d}`
    case 'DMY':
      return `${d}/${m}/${y}`
    case 'YM':
      return `${y}-${m}`
    case 'MonYYYY':
      return `${monthName} ${y}`
  }
}

// Convenience functions that use the core formatDate
const formatYMD = (date: Date): string => formatDate(date, 'YMD')
const formatDMY = (date: Date): string => formatDate(date, 'DMY')
const formatYM = (year: number, month: number): string => formatDate(new Date(year, month, 1), 'YM')
const formatMonYYYY = (date: Date): string => formatDate(date, 'MonYYYY')

const toYearMonth = (date: Date): YearMonth => ({
  year: date.getFullYear(),
  month: date.getMonth(),
})

const compareYearMonth = (a: YearMonth, b: YearMonth): number => {
  if (a.year !== b.year) return a.year < b.year ? -1 : 1
  if (a.month !== b.month) return a.month < b.month ? -1 : 1
  return 0
}

const isCurrentMonth = (year: number, month: number): boolean => {
  const today = new Date()
  return year === today.getFullYear() && month === today.getMonth()
}

const isDateWithinBounds = (
  date: Date,
  minNavigation?: string | null,
  maxNavigation?: string | null
): boolean => {
  const d = startOfDay(date)

  if (minNavigation) {
    const minDate = parseModelDate(minNavigation)
    if (minDate && d < startOfDay(minDate)) return false
  }

  if (maxNavigation) {
    const maxDate = parseModelDate(maxNavigation)
    if (maxDate && d > startOfDay(maxDate)) return false
  }

  return true
}

const isMonthWithinBounds = (
  ym: YearMonth,
  minNavigation?: string | null,
  maxNavigation?: string | null
): boolean => {
  const minDate = minNavigation ? parseModelDate(minNavigation) : null
  const maxDate = maxNavigation ? parseModelDate(maxNavigation) : null

  if (minDate && compareYearMonth(ym, toYearMonth(minDate)) < 0) return false
  if (maxDate && compareYearMonth(ym, toYearMonth(maxDate)) > 0) return false

  return true
}

// ===== PROPS =====
const props = withDefaults(defineProps<DatePickerProps>(), {
  placeholder: '',
  view: 'date',
  mode: 'range',
  minNavigation: null,
  maxNavigation: null,
})

// ===== EMITS =====
const emit = defineEmits<{
  'update:modelValue': [value: (string | null)[] | string | null]
}>()

// ===== REFS =====
const isOpen = ref(false)
const datePickerRef = ref<HTMLInputElement | null>(null)
const dropdown = ref<HTMLElement | null>(null)
const selectedSingle = ref<Date | null>(null)
const currentDate = ref(new Date())
const selectedRange = ref<{ start: Date | null; end: Date | null }>({ start: null, end: null })

// ===== COMPUTED PROPERTIES =====
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
  if (!props.minNavigation) return true

  const currentYear = currentDate.value.getFullYear()
  const currentMonth = currentDate.value.getMonth()

  if (isMonthView.value) {
    const minDate = parseModelDate(props.minNavigation)
    return minDate ? currentYear - 1 >= minDate.getFullYear() : true
  } else {
    const prevMonth = currentMonth === 0 ? 11 : currentMonth - 1
    const prevYear = currentMonth === 0 ? currentYear - 1 : currentYear
    const minDate = parseModelDate(props.minNavigation)

    if (!minDate) return true
    const minYear = minDate.getFullYear()
    const minMonth = minDate.getMonth()

    return prevYear > minYear || (prevYear === minYear && prevMonth >= minMonth)
  }
})

const canNavigateNext = computed(() => {
  if (!props.maxNavigation) return true

  const currentYear = currentDate.value.getFullYear()
  const currentMonth = currentDate.value.getMonth()

  if (isMonthView.value) {
    const maxDate = parseModelDate(props.maxNavigation)
    return maxDate ? currentYear + 1 <= maxDate.getFullYear() : true
  } else {
    const nextMonth = currentMonth === 11 ? 0 : currentMonth + 1
    const nextYear = currentMonth === 11 ? currentYear + 1 : currentYear
    const maxDate = parseModelDate(props.maxNavigation)

    if (!maxDate) return true
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
        ? formatMonYYYY(selectedSingle.value)
        : formatDMY(selectedSingle.value)
    }
    return ''
  } else {
    // Range mode
    if (isMonthView.value) {
      if (selectedRange.value?.start && selectedRange.value?.end) {
        return `${formatMonYYYY(selectedRange.value.start)} - ${formatMonYYYY(selectedRange.value.end)}`
      }
      return ''
    } else {
      if (selectedRange.value?.start && selectedRange.value?.end) {
        return `${formatDMY(selectedRange.value.start)} - ${formatDMY(selectedRange.value.end)}`
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
      dateString: `prev-${formatYMD(date)}`,
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
      dateString: `curr-${formatYMD(date)}`,
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
      dateString: `next-${formatYMD(date)}`,
    })
  }

  return days
})

const monthGrid = computed((): MonthData[] => {
  return MONTH_NAMES.map((name, index) => ({
    month: index,
    name,
    shortName: name.slice(0, 3),
    date: formatYM(displayYear.value, index),
    isCurrentMonth: isCurrentMonth(displayYear.value, index),
  }))
})

const showQuickSelection = computed(() => !isMonthView.value && !isSingleMode.value)

const quickSelectionOptions = computed((): QuickSelectionOption[] => {
  const today = new Date()

  const options = [
    {
      key: 'today',
      label: 'Today',
      getValue: () => [formatYMD(today), formatYMD(today)],
    },
    {
      key: 'yesterday',
      label: 'Yesterday',
      getValue: () => {
        const yesterday = new Date(today)
        yesterday.setDate(today.getDate() - 1)
        return [formatYMD(yesterday), formatYMD(yesterday)]
      },
    },
    {
      key: 'last30days',
      label: 'Last 30 days',
      getValue: () => {
        const last30Days = new Date(today)
        last30Days.setDate(today.getDate() - 29)
        return [formatYMD(last30Days), formatYMD(today)]
      },
    },
    {
      key: 'thismonth',
      label: 'This month',
      getValue: () => {
        const startOfMonth = new Date(today.getFullYear(), today.getMonth(), 1)
        return [formatYMD(startOfMonth), formatYMD(today)]
      },
    },
    {
      key: 'last180days',
      label: '6 months',
      getValue: () => {
        const last180Days = new Date(today)
        last180Days.setDate(today.getDate() - 179)
        return [formatYMD(last180Days), formatYMD(today)]
      },
    },
  ]

  // Filter based on minDate constraints
  return options
    .filter((option) => {
      if (!props.minNavigation) return true
      const [startDate] = option.getValue()
      return isDateWithinBounds(new Date(startDate), props.minNavigation, props.maxNavigation)
    })
    .map((option) => ({ ...option, isEnabled: true }))
})

// ===== BUSINESS LOGIC HELPERS =====
const isDateSelectable = (date: Date): boolean =>
  isDateWithinBounds(date, props.minNavigation, props.maxNavigation)

const isMonthSelectable = (year: number, month: number): boolean =>
  isMonthWithinBounds({ year, month }, props.minNavigation, props.maxNavigation)

// Range selection helpers
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

// Month selection helpers
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

const getDayClasses = (dayData: DayData): string => {
  const classes = []

  // Handle non-current month days and non-selectable dates
  if (!dayData.isCurrentMonth || !isDateSelectable(dayData.date)) {
    classes.push('text-gray-400 cursor-not-allowed')
    return classes.join(' ')
  }

  // Base classes for selectable dates
  classes.push('text-gray-900 hover:bg-primary-300')

  // Today indicator
  if (dayData.isToday) {
    classes.push('border border-primary-300')
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
      return 'border border-primary-300 text-primary-800 font-medium'
    } else {
      return 'text-gray-700 hover:bg-primary-300'
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
      return 'border border-primary-300 text-primary-800 font-medium'
    } else {
      return 'text-gray-700 hover:bg-primary-300'
    }
  }
}

// Event Handlers
const handleDateClick = (date: Date): void => {
  if (!isDateSelectable(date)) return

  if (isSingleMode.value) {
    // Single mode logic
    selectedSingle.value = date
    emit(
      'update:modelValue',
      isMonthView.value ? formatYM(date.getFullYear(), date.getMonth()) : formatYMD(date)
    )
    isOpen.value = false
  } else {
    // Range selection logic
    if (!selectedRange.value?.start || (selectedRange.value?.start && selectedRange.value?.end)) {
      // Start new range
      selectedRange.value = { start: date, end: null }
    } else if (selectedRange.value?.start && !selectedRange.value?.end) {
      // Complete the range - auto-swap if needed
      const startDate = date < selectedRange.value.start ? date : selectedRange.value.start
      const endDate = date < selectedRange.value.start ? selectedRange.value.start : date

      selectedRange.value = { start: startDate, end: endDate }
      emit('update:modelValue', [formatYMD(startDate), formatYMD(endDate)])
      isOpen.value = false
    }
  }
}

const handleMonthClick = (monthData: MonthData): void => {
  if (!isMonthSelectable(displayYear.value, monthData.month)) return

  const selectedMonth = new Date(displayYear.value, monthData.month, 1)

  if (isSingleMode.value) {
    // Single mode logic
    selectedSingle.value = selectedMonth
    emit('update:modelValue', formatYM(selectedMonth.getFullYear(), selectedMonth.getMonth()))
    isOpen.value = false
  } else {
    // Range selection logic for months
    if (!selectedRange.value?.start || (selectedRange.value?.start && selectedRange.value?.end)) {
      // Start new range
      selectedRange.value = { start: selectedMonth, end: null }
    } else if (selectedRange.value?.start && !selectedRange.value?.end) {
      // Complete the range - auto-swap if needed
      const startMonth =
        selectedMonth < selectedRange.value.start ? selectedMonth : selectedRange.value.start
      const endMonth =
        selectedMonth < selectedRange.value.start ? selectedRange.value.start : selectedMonth

      selectedRange.value = { start: startMonth, end: endMonth }
      emit('update:modelValue', [
        formatYM(startMonth.getFullYear(), startMonth.getMonth()),
        formatYM(endMonth.getFullYear(), endMonth.getMonth()),
      ])
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

  const yearOffset = isMonthView.value ? (direction === 'previous' ? -1 : 1) : 0
  const monthOffset = !isMonthView.value ? (direction === 'previous' ? -1 : 1) : 0

  currentDate.value = new Date(
    currentDate.value.getFullYear() + yearOffset,
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

// Keep local state in sync with v-model changes from parent
watch(
  () => props.modelValue,
  (newValue) => {
    if (isSingleMode.value) {
      // Single mode
      if (typeof newValue === 'string' && newValue) {
        const parsed = parseModelDate(isMonthView.value ? `${newValue}-01` : newValue)
        selectedSingle.value = parsed ? startOfDay(parsed) : null
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
        const s = newValue[0]
        const e = newValue[1]
        const sParsed = s ? parseModelDate(isMonthView.value ? `${s}-01` : s) : null
        const eParsed = e ? parseModelDate(isMonthView.value ? `${e}-01` : e) : null
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
