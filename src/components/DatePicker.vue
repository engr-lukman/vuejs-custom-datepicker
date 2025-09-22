<template>
  <div class="relative">
    <!-- Standard Input Field -->
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
      class="absolute top-full left-0 z-50 mt-1 w-80 max-w-[calc(100vw-1rem)] rounded-lg border border-gray-200 bg-white shadow-lg"
    >
      <!-- Header -->
      <div class="flex items-center justify-between border-b border-gray-100 p-3">
        <button @click="navigatePrevious" class="rounded p-1 transition-colors hover:bg-gray-100">
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
          <span v-if="monthPicker">{{ displayYear }}</span>
          <span v-else>{{ monthNames[displayMonth] }} {{ displayYear }}</span>
        </h2>

        <button @click="navigateNext" class="rounded p-1 transition-colors hover:bg-gray-100">
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

      <!-- Body -->
      <div class="p-3">
        <!-- Month Picker Grid -->
        <div v-if="monthPicker" class="grid grid-cols-3 gap-1">
          <button
            v-for="month in monthGrid"
            :key="month.month"
            @click="selectMonth(month)"
            :class="getMonthClasses(month)"
            class="rounded p-2 text-xs transition-colors hover:bg-blue-50"
          >
            {{ month.shortName }}
          </button>
        </div>

        <!-- Day Picker -->
        <template v-else>
          <!-- Day Headers -->
          <div class="mb-1 grid grid-cols-7 gap-1">
            <div
              v-for="day in dayNames"
              :key="day"
              class="flex h-8 items-center justify-center text-xs font-medium text-gray-500"
            >
              {{ day }}
            </div>
          </div>

          <!-- Calendar Grid -->
          <div class="grid grid-cols-7 gap-1">
            <button
              v-for="(day, index) in calendarDays"
              :key="index"
              @click="selectDate(day)"
              :class="getDayClasses(day)"
              class="flex h-8 items-center justify-center rounded text-xs transition-colors"
            >
              {{ day.day }}
            </button>
          </div>
        </template>

        <!-- Range Selection Info -->
        <div
          v-if="range && isSelectingRange"
          class="bg-primary-50 text-primary-700 mt-2 rounded p-2 text-center text-xs"
        >
          <span v-if="monthPicker">Select the end month for your range</span>
          <span v-else>Select the end date for your range</span>
        </div>
        <div
          v-else-if="range && !isSelectingRange && !hasRangeStart"
          class="mt-2 rounded bg-gray-50 p-2 text-center text-xs text-gray-600"
        >
          <span v-if="monthPicker">Select the start month for your range</span>
          <span v-else>Select the start date for your range</span>
        </div>
      </div>

      <!-- Footer -->
      <div
        class="flex items-center justify-between rounded-b-lg border-t border-gray-100 bg-gray-50 p-3"
      >
        <button
          @click="setToday"
          class="bg-primary-600 hover:bg-primary-700 rounded px-3 py-1 text-xs font-medium text-white transition-colors"
        >
          Today
        </button>
        <button
          @click="clearDate"
          class="rounded border border-gray-300 bg-white px-3 py-1 text-xs font-medium text-gray-700 transition-colors hover:bg-gray-50"
        >
          Clear
        </button>
      </div>
    </div>

    <!-- Mobile Backdrop -->
    <div
      v-if="isOpen"
      @click="closePicker"
      class="bg-opacity-25 fixed inset-0 z-40 bg-black sm:hidden"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'

// Types
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
}

interface Props {
  modelValue?: string | (string | null)[] | null
  placeholder?: string
  range?: boolean
  monthPicker?: boolean
  minDate?: string | null
  maxDate?: string | null
  enableDateValidation?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  placeholder: 'Select date',
  range: false,
  monthPicker: false,
  minDate: null,
  maxDate: null,
  enableDateValidation: true,
})

const emit = defineEmits<{
  'update:modelValue': [value: string | (string | null)[] | null]
}>()

// Refs
const dropdown = ref<HTMLDivElement>()
const inputElement = ref<HTMLInputElement>()
const isOpen = ref(false)
const displayMonth = ref(new Date().getMonth())
const displayYear = ref(new Date().getFullYear())
const isSelectingRange = ref(false)

// Constants
const monthNames = [
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

const dayNames = ['Su', 'Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa']

// Computed Properties
const minDateForValidation = computed(() => {
  if (props.minDate) return props.minDate
  if (!props.enableDateValidation) return null

  // 180 days before current date
  const today = new Date()
  const minDate = new Date(today)
  minDate.setDate(today.getDate() - 180)
  return formatDate(minDate)
})

const maxDateForValidation = computed(() => {
  if (props.maxDate) return props.maxDate
  if (!props.enableDateValidation) return null

  // Current date
  const today = new Date()
  return formatDate(today)
})

const displayValue = computed(() => {
  if (props.range) {
    const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
    if (values[0] && values[1]) {
      return `${values[0]} - ${values[1]}`
    } else if (values[0]) {
      return `${values[0]} - ...`
    }
    return ''
  }
  return (props.modelValue as string) || ''
})

const hasRangeStart = computed(() => {
  if (!props.range) return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  return Boolean(values[0])
})

const monthGrid = computed((): MonthData[] => {
  return monthNames.map((name, index) => ({
    month: index,
    name,
    shortName: name.slice(0, 3),
    date: `${displayYear.value}-${String(index + 1).padStart(2, '0')}`,
  }))
})

const calendarDays = computed((): DayData[] => {
  const firstDay = new Date(displayYear.value, displayMonth.value, 1)
  const startDate = new Date(firstDay)
  startDate.setDate(startDate.getDate() - firstDay.getDay())

  const days: DayData[] = []
  const current = new Date(startDate)

  for (let i = 0; i < 42; i++) {
    days.push({
      date: new Date(current),
      day: current.getDate(),
      month: current.getMonth(),
      year: current.getFullYear(),
      isCurrentMonth: current.getMonth() === displayMonth.value,
      isToday: isToday(current),
      dateString: formatDate(current),
    })
    current.setDate(current.getDate() + 1)
  }

  return days
})

// Helper Functions
const isDateDisabled = (dateString: string): boolean => {
  if (!props.enableDateValidation) return false

  try {
    const date = new Date(dateString)
    if (isNaN(date.getTime())) return true

    const minDate = minDateForValidation.value ? new Date(minDateForValidation.value) : null
    const maxDate = maxDateForValidation.value ? new Date(maxDateForValidation.value) : null

    if (minDate && date < minDate) return true
    if (maxDate && date > maxDate) return true

    return false
  } catch {
    return true
  }
}

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

const formatDate = (date: Date): string => {
  try {
    return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`
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

// Methods
const togglePicker = () => {
  isOpen.value = !isOpen.value
}

const closePicker = () => {
  isOpen.value = false
  isSelectingRange.value = false
}

const navigatePrevious = () => {
  if (props.monthPicker) {
    displayYear.value--
  } else {
    if (displayMonth.value === 0) {
      displayMonth.value = 11
      displayYear.value--
    } else {
      displayMonth.value--
    }
  }
}

const navigateNext = () => {
  if (props.monthPicker) {
    displayYear.value++
  } else {
    if (displayMonth.value === 11) {
      displayMonth.value = 0
      displayYear.value++
    } else {
      displayMonth.value++
    }
  }
}

const selectDate = (day: DayData) => {
  if (!day.isCurrentMonth) return
  if (isDateDisabled(day.dateString)) return

  const selectedDate = day.dateString

  if (props.range) {
    const currentRange = Array.isArray(props.modelValue) ? [...props.modelValue] : [null, null]

    if (!currentRange[0] || (currentRange[0] && currentRange[1])) {
      emit('update:modelValue', [selectedDate, null])
      isSelectingRange.value = true
    } else {
      const startDate = new Date(currentRange[0])
      const endDate = new Date(selectedDate)

      if (endDate < startDate) {
        emit('update:modelValue', [selectedDate, currentRange[0]])
      } else {
        emit('update:modelValue', [currentRange[0], selectedDate])
      }
      isSelectingRange.value = false
      closePicker()
    }
  } else {
    emit('update:modelValue', selectedDate)
    closePicker()
  }
}

const selectMonth = (month: MonthData) => {
  const selectedMonth = month.date

  if (props.range) {
    const currentRange = Array.isArray(props.modelValue) ? [...props.modelValue] : [null, null]

    if (!currentRange[0] || (currentRange[0] && currentRange[1])) {
      emit('update:modelValue', [selectedMonth, null])
      isSelectingRange.value = true
    } else {
      const startMonth = currentRange[0]
      const endMonth = selectedMonth

      if (endMonth < startMonth) {
        emit('update:modelValue', [selectedMonth, currentRange[0]])
      } else {
        emit('update:modelValue', [currentRange[0], selectedMonth])
      }
      isSelectingRange.value = false
      closePicker()
    }
  } else {
    emit('update:modelValue', selectedMonth)
    closePicker()
  }
}

const setToday = () => {
  try {
    const today = new Date()

    // Check if today is disabled before setting it
    const todayString = formatDate(today)
    if (isDateDisabled(todayString)) {
      // If today is disabled, don't set it and close the picker
      closePicker()
      return
    }

    if (props.monthPicker) {
      const monthString = formatMonth(today.getFullYear(), today.getMonth())
      if (props.range) {
        emit('update:modelValue', [monthString, monthString])
      } else {
        emit('update:modelValue', monthString)
      }
    } else {
      if (props.range) {
        emit('update:modelValue', [todayString, todayString])
      } else {
        emit('update:modelValue', todayString)
      }
    }

    displayMonth.value = today.getMonth()
    displayYear.value = today.getFullYear()
    closePicker()
  } catch (error) {
    console.warn('Error setting today date:', error)
    closePicker()
  }
}

const clearDate = () => {
  try {
    if (props.range) {
      emit('update:modelValue', [null, null])
    } else {
      emit('update:modelValue', null)
    }
    isSelectingRange.value = false
    closePicker()
  } catch (error) {
    console.warn('Error clearing date:', error)
    closePicker()
  }
}

// Styling Methods
const getDayClasses = (day: DayData): string => {
  if (!day.isCurrentMonth) {
    return 'text-gray-300 cursor-not-allowed'
  }

  if (isDateDisabled(day.dateString)) {
    return 'text-gray-300 cursor-not-allowed line-through'
  }

  const isSelected = isDateSelected(day.dateString)
  const isInRange = isDateInRange(day.dateString)
  const isRangeStart = isDateRangeStart(day.dateString)
  const isRangeEnd = isDateRangeEnd(day.dateString)

  if (isRangeStart) {
    return 'bg-primary-600 text-white font-medium shadow-md'
  } else if (isRangeEnd) {
    return 'bg-primary-700 text-white font-medium shadow-md'
  } else if (isSelected) {
    return 'bg-primary-600 text-white font-medium shadow-md'
  } else if (isInRange) {
    return 'bg-range-100 text-range-800 border border-range-200'
  } else if (day.isToday) {
    return 'border-2 border-today-500 bg-today-50 text-today-800 font-medium'
  } else {
    return 'text-gray-700 hover:bg-gray-100'
  }
}

const getMonthClasses = (month: MonthData): string => {
  const isSelected = isMonthSelected(month.date)
  const isInRange = isMonthInRange(month.date)
  const isRangeStart = isMonthRangeStart(month.date)
  const isRangeEnd = isMonthRangeEnd(month.date)

  if (isRangeStart) {
    return 'bg-primary-600 text-white font-medium shadow-md'
  } else if (isRangeEnd) {
    return 'bg-primary-700 text-white font-medium shadow-md'
  } else if (isSelected) {
    return 'bg-primary-600 text-white font-medium shadow-md'
  } else if (isInRange) {
    return 'bg-range-100 text-range-800 border border-range-200'
  } else {
    return 'text-gray-700 hover:bg-gray-100'
  }
}

// Selection Helpers
const isDateSelected = (dateString: string): boolean => {
  if (props.range) {
    const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
    return values.some((v) => v === dateString)
  }
  return props.modelValue === dateString
}

const isDateInRange = (dateString: string): boolean => {
  if (!props.range) return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  if (!values[0] || !values[1]) return false

  const date = new Date(dateString)
  const start = new Date(values[0])
  const end = new Date(values[1])

  return date > start && date < end
}

const isDateRangeStart = (dateString: string): boolean => {
  if (!props.range) return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  return values[0] === dateString
}

const isDateRangeEnd = (dateString: string): boolean => {
  if (!props.range) return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  return values[1] === dateString
}

const isMonthSelected = (monthString: string): boolean => {
  if (props.range) {
    const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
    return values.some((v) => v === monthString)
  }
  return props.modelValue === monthString
}

const isMonthInRange = (monthString: string): boolean => {
  if (!props.range) return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  if (!values[0] || !values[1]) return false

  return monthString > values[0] && monthString < values[1]
}

const isMonthRangeStart = (monthString: string): boolean => {
  if (!props.range) return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  return values[0] === monthString
}

const isMonthRangeEnd = (monthString: string): boolean => {
  if (!props.range) return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  return values[1] === monthString
}

// Click outside functionality
const handleClickOutside = (event: MouseEvent) => {
  if (
    dropdown.value &&
    !dropdown.value.contains(event.target as Node) &&
    inputElement.value &&
    !inputElement.value.contains(event.target as Node)
  ) {
    closePicker()
  }
}

// Lifecycle
onMounted(() => {
  document.addEventListener('click', handleClickOutside)
})

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside)
})

// Watch for external changes
watch(
  () => props.modelValue,
  () => {
    isSelectingRange.value = false
  }
)
</script>
