<template>
  <div class="relative">
    <!-- Date Input Field -->
    <input
      ref="dateInputRef"
      :value="formattedDisplayValue"
      :placeholder="placeholderText"
      readonly
      @click="toggleDatePicker"
      @keydown.enter="toggleDatePicker"
      @keydown.space.prevent="toggleDatePicker"
      @keydown.escape="closeDatePicker"
      :aria-expanded="isPickerOpen"
      aria-haspopup="dialog"
      class="focus:ring-primary-500 focus:border-primary-500 w-full cursor-pointer rounded-md border border-gray-300 bg-white px-3 py-2 pr-10 text-sm text-gray-900 placeholder-gray-500 transition-colors duration-200 hover:border-gray-400 focus:ring-1 focus:outline-none"
    />

    <!-- Date Picker Dropdown -->
    <div
      v-if="isPickerOpen"
      ref="dropdownRef"
      role="dialog"
      aria-modal="false"
      aria-label="Date picker"
      class="absolute top-full left-0 z-50 mt-1 rounded-lg border border-gray-200 bg-white shadow-lg"
      :class="{ 'w-xs': !isQuickSelectVisible, 'w-sm': isQuickSelectVisible }"
    >
      <div class="flex flex-col sm:flex-row">
        <!-- Quick Selection Panel -->
        <div
          v-if="isQuickSelectVisible"
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
                v-for="option in availableQuickSelectOptions"
                :key="option.key"
                type="button"
                :disabled="!option.isEnabled"
                @click="applyQuickSelect(option)"
                :class="getQuickSelectButtonClass(option)"
                class="w-full cursor-pointer rounded px-2 py-1.5 text-left text-xs transition-colors"
              >
                {{ option.label }}
              </button>
            </div>
          </div>
        </div>

        <!-- Calendar Panel -->
        <div class="flex-1">
          <!-- Month Navigation Header -->
          <div class="flex items-center justify-between border-b border-gray-100 p-3">
            <button
              type="button"
              @click="navigateMonth('previous')"
              :disabled="!canNavigatePreviousMonth"
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
              {{ `${currentMonthName} ${currentYear}` }}
            </h2>

            <button
              type="button"
              @click="navigateMonth('next')"
              :disabled="!canNavigateNextMonth"
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

          <!-- Calendar Days -->
          <div class="p-3">
            <!-- Weekday Names -->
            <div class="mb-2 grid grid-cols-7 gap-2">
              <div
                v-for="weekday in WEEKDAY_NAMES"
                :key="weekday"
                class="text-center text-xs font-medium text-gray-500"
              >
                {{ weekday }}
              </div>
            </div>

            <!-- Days Grid -->
            <div class="grid grid-cols-7 gap-2">
              <button
                v-for="day in calendarDays"
                :key="day.dateString"
                :disabled="!day.isCurrentMonth || !isDateSelectable(day.date)"
                @click="selectDate(day.date)"
                :class="getCalendarDayClass(day)"
                class="relative h-8 w-8 cursor-pointer rounded text-sm transition-colors"
              >
                {{ day.day }}
              </button>
            </div>
          </div>

          <!-- Clear Selection -->
          <div class="flex justify-end border-t border-gray-100 p-1.5">
            <button
              @click="clearDateSelection"
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
  ref,
  computed,
  watch,
  onMounted,
  onUnmounted,
  defineProps,
  defineEmits,
  withDefaults,
} from 'vue'

/** ========================
 *  Interfaces & Types
 * ======================== */
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

interface CalendarDay {
  date: Date
  day: number
  month: number
  year: number
  isCurrentMonth: boolean
  isToday: boolean
  dateString: string
}

interface QuickSelectOption {
  key: string
  label: string
  getValue: () => string[]
  isEnabled?: boolean
}

/** ========================
 *  Constants
 * ======================== */
const WEEKDAY_NAMES = ['Su', 'Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa']
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
 *  Helper Functions
 * ======================== */
const startOfDay = (date: Date): Date => {
  const d = new Date(date)
  d.setHours(0, 0, 0, 0)
  return d
}

const formatDate = (date: Date, format: 'YMD' | 'DMY' | 'MonYYYY'): string => {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  if (format === 'YMD') return `${year}-${month}-${day}`
  if (format === 'DMY') return `${day}/${month}/${year}`
  return `${MONTH_NAMES[date.getMonth()].slice(0, 3)} ${year}`
}

const parseDateString = (value?: string | null): Date | null => {
  if (!value) return null
  const match = /^(\d{4})-(\d{2})-(\d{2})$/.exec(value)
  if (match) return new Date(+match[1], +match[2] - 1, +match[3])
  const parsed = new Date(value)
  return isNaN(parsed.getTime()) ? null : parsed
}

const isSameDate = (a: Date, b: Date): boolean =>
  a.getFullYear() === b.getFullYear() &&
  a.getMonth() === b.getMonth() &&
  a.getDate() === b.getDate()

const isToday = (date: Date): boolean => isSameDate(date, new Date())

const isWithinLimits = (date: Date, min?: string | null, max?: string | null): boolean => {
  const current = startOfDay(date)
  const minDate = min ? startOfDay(parseDateString(min)!) : null
  const maxDate = max ? startOfDay(parseDateString(max)!) : null
  if (minDate && current < minDate) return false
  if (maxDate && current > maxDate) return false
  return true
}

/** ========================
 *  Props & Emits
 * ======================== */
const props = withDefaults(defineProps<DatePickerProps>(), {
  placeholder: '',
  mode: 'range',
  minNavigation: null,
  maxNavigation: null,
})
const emit = defineEmits<{ 'update:modelValue': [string | string[] | null] }>()

/** ========================
 *  Reactive State
 * ======================== */
const isPickerOpen = ref(false)
const dateInputRef = ref<HTMLInputElement | null>(null)
const dropdownRef = ref<HTMLElement | null>(null)
const currentCalendarDate = ref(new Date())
const selectedDateSingle = ref<Date | null>(null)
const selectedDateRange = ref<DateRange>({ start: null, end: null })

/** ========================
 *  Computed Properties
 * ======================== */
const isSingleMode = computed(() => props.mode === 'single')
const placeholderText = computed(
  () => props.placeholder || (isSingleMode.value ? 'Select date' : 'Select date range')
)
const currentMonthName = computed(() => MONTH_NAMES[currentCalendarDate.value.getMonth()])
const currentYear = computed(() => currentCalendarDate.value.getFullYear())

/** ========================
 *  Navigation
 * ======================== */
const canNavigatePreviousMonth = computed(() => {
  if (!props.minNavigation) return true
  const year = currentCalendarDate.value.getFullYear()
  const month = currentCalendarDate.value.getMonth()
  const prevMonth = month === 0 ? 11 : month - 1
  const prevYear = month === 0 ? year - 1 : year
  const minDate = parseDateString(props.minNavigation)
  if (!minDate) return true
  return (
    prevYear > minDate.getFullYear() ||
    (prevYear === minDate.getFullYear() && prevMonth >= minDate.getMonth())
  )
})

const canNavigateNextMonth = computed(() => {
  if (!props.maxNavigation) return true
  const year = currentCalendarDate.value.getFullYear()
  const month = currentCalendarDate.value.getMonth()
  const nextMonth = month === 11 ? 0 : month + 1
  const nextYear = month === 11 ? year + 1 : year
  const maxDate = parseDateString(props.maxNavigation)
  if (!maxDate) return true
  return (
    nextYear < maxDate.getFullYear() ||
    (nextYear === maxDate.getFullYear() && nextMonth <= maxDate.getMonth())
  )
})

/** ========================
 *  Calendar Generation
 * ======================== */
const calendarDays = computed<CalendarDay[]>(() => {
  const year = currentCalendarDate.value.getFullYear()
  const month = currentCalendarDate.value.getMonth()
  const firstOfMonth = new Date(year, month, 1)
  const startWeekDay = firstOfMonth.getDay()
  const startDate = new Date(year, month, 1 - startWeekDay)

  return Array.from({ length: 42 }).map((_, idx) => {
    const date = new Date(startDate)
    date.setDate(startDate.getDate() + idx)
    return {
      date,
      day: date.getDate(),
      month: date.getMonth(),
      year: date.getFullYear(),
      isCurrentMonth: date.getMonth() === month,
      isToday: isToday(date),
      dateString: `${date.getMonth() === month ? 'curr' : date < firstOfMonth ? 'prev' : 'next'}-${formatDate(date, 'YMD')}`,
    }
  })
})

/** ========================
 *  Quick Select Options
 * ======================== */
const isQuickSelectVisible = computed(() => !isSingleMode.value)

const availableQuickSelectOptions = computed<QuickSelectOption[]>(() => {
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

  const options: QuickSelectOption[] = [
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

  return options
    .map((opt) => {
      const [startStr, endStr] = opt.getValue()
      const start = parseDateString(startStr)
      const end = parseDateString(endStr)
      const isEnabled =
        start &&
        end &&
        isWithinLimits(start, props.minNavigation, props.maxNavigation) &&
        isWithinLimits(end, props.minNavigation, props.maxNavigation)
      return { ...opt, isEnabled: !!isEnabled }
    })
    .filter((opt) => opt.isEnabled)
})

/** ========================
 *  Selection Logic
 * ======================== */
const isDateSelectable = (date: Date) =>
  isWithinLimits(date, props.minNavigation, props.maxNavigation)
const isDateSelected = (date: Date) =>
  isSingleMode.value
    ? selectedDateSingle.value && isSameDate(date, selectedDateSingle.value)
    : !!(
        selectedDateRange.value.start &&
        selectedDateRange.value.end &&
        date >= selectedDateRange.value.start &&
        date <= selectedDateRange.value.end
      )
const isRangeStartDate = (date: Date) =>
  selectedDateRange.value.start && isSameDate(date, selectedDateRange.value.start)
const isRangeEndDate = (date: Date) =>
  selectedDateRange.value.end && isSameDate(date, selectedDateRange.value.end)

/** ========================
 *  CSS Classes
 * ======================== */
const getCalendarDayClass = (day: CalendarDay): string => {
  if (!day.isCurrentMonth || !isDateSelectable(day.date)) return 'text-gray-400 cursor-not-allowed'
  let cls = 'text-gray-900 hover:bg-primary-300'
  if (day.isToday) cls += ' border border-primary-300'
  if (isSingleMode.value) {
    if (isDateSelected(day.date)) cls += ' bg-primary-700 text-white'
  } else {
    if (isRangeStartDate(day.date) || isRangeEndDate(day.date)) cls += ' bg-primary-700 text-white'
    else if (isDateSelected(day.date)) cls += ' bg-primary-300 text-white'
  }
  return cls
}

const getQuickSelectButtonClass = (option: QuickSelectOption) => {
  const currentValue = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  const [start, end] = option.getValue()
  return currentValue[0] === start && currentValue[1] === end
    ? 'bg-primary-700 text-white font-medium shadow-sm'
    : 'text-gray-700 hover:bg-primary-300 hover:text-primary-700'
}

/** ========================
 *  Event Handlers
 * ======================== */
const selectDate = (date: Date) => {
  if (!isDateSelectable(date)) return
  if (isSingleMode.value) {
    selectedDateSingle.value = date
    emit('update:modelValue', formatDate(date, 'YMD'))
    closeDatePicker()
  } else {
    if (
      !selectedDateRange.value.start ||
      (selectedDateRange.value.start && selectedDateRange.value.end)
    ) {
      selectedDateRange.value = { start: date, end: null }
    } else if (selectedDateRange.value.start && !selectedDateRange.value.end) {
      const start = date < selectedDateRange.value.start ? date : selectedDateRange.value.start
      const end = date < selectedDateRange.value.start ? selectedDateRange.value.start : date
      selectedDateRange.value = { start, end }
      emit('update:modelValue', [formatDate(start, 'YMD'), formatDate(end, 'YMD')])
      closeDatePicker()
    }
  }
}

const clearDateSelection = () => {
  if (isSingleMode.value) selectedDateSingle.value = null
  else selectedDateRange.value = { start: null, end: null }
  emit('update:modelValue', null)
  closeDatePicker()
}

const applyQuickSelect = (option: QuickSelectOption) => {
  const [startStr, endStr] = option.getValue()
  selectedDateRange.value = { start: new Date(startStr), end: new Date(endStr) }
  emit('update:modelValue', [startStr, endStr])
  closeDatePicker()
}

const toggleDatePicker = () => {
  isPickerOpen.value = !isPickerOpen.value
}
const closeDatePicker = () => {
  isPickerOpen.value = false
}

const navigateMonth = (direction: 'previous' | 'next') => {
  const canMove =
    direction === 'previous' ? canNavigatePreviousMonth.value : canNavigateNextMonth.value
  if (!canMove) return
  const offset = direction === 'previous' ? -1 : 1
  currentCalendarDate.value = new Date(
    currentCalendarDate.value.getFullYear(),
    currentCalendarDate.value.getMonth() + offset,
    1
  )
}

/** ========================
 *  Outside Click & Keyboard
 * ======================== */
const handleClickOutside = (event: Event) => {
  const target = event.target as Node
  if (
    dropdownRef.value &&
    !dropdownRef.value.contains(target) &&
    dateInputRef.value &&
    !dateInputRef.value.contains(target)
  ) {
    closeDatePicker()
  }
}

const handleKeyDown = (event: KeyboardEvent) => {
  if (!isPickerOpen.value) return
  if (event.key === 'Escape') {
    event.preventDefault()
    closeDatePicker()
  }
}

/** ========================
 *  Watchers
 * ======================== */
watch(
  () => props.modelValue,
  (value) => {
    if (isSingleMode.value) {
      selectedDateSingle.value =
        typeof value === 'string' && value ? startOfDay(parseDateString(value)!) : null
    } else {
      if (!value || !Array.isArray(value)) selectedDateRange.value = { start: null, end: null }
      else
        selectedDateRange.value = {
          start: parseDateString(value[0]) ? startOfDay(parseDateString(value[0])!) : null,
          end: parseDateString(value[1]) ? startOfDay(parseDateString(value[1])!) : null,
        }
    }
  },
  { immediate: true }
)

/** ========================
 *  Lifecycle Hooks
 * ======================== */
onMounted(() => {
  document.addEventListener('click', handleClickOutside)
  document.addEventListener('keydown', handleKeyDown)
})
onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside)
  document.removeEventListener('keydown', handleKeyDown)
})

/** ========================
 *  Display Value
 * ======================== */
const formattedDisplayValue = computed(() => {
  if (isSingleMode.value)
    return selectedDateSingle.value ? formatDate(selectedDateSingle.value, 'DMY') : ''
  const { start, end } = selectedDateRange.value
  if (start && end) return `${formatDate(start, 'DMY')} - ${formatDate(end, 'DMY')}`
  return start ? formatDate(start, 'DMY') : end ? formatDate(end, 'DMY') : ''
})
</script>
