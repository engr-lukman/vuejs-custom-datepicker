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
      class="w-full px-3 py-2 pr-10 text-sm border border-gray-300 rounded-md bg-white placeholder-gray-500 text-gray-900 cursor-pointer transition-colors duration-200 hover:border-gray-400 focus:outline-none focus:ring-2 focus:ring-primary-500 focus:border-primary-500"
    />
    
    <!-- Calendar Icon -->
    <div class="absolute inset-y-0 right-0 flex items-center pr-3 pointer-events-none">
      <svg class="w-4 h-4 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
      </svg>
    </div>

    <!-- Dropdown -->
    <div 
      v-if="isOpen"
      ref="dropdown"
      class="absolute top-full left-0 z-50 mt-1 w-80 max-w-[calc(100vw-1rem)] bg-white border border-gray-200 rounded-lg shadow-lg"
    >
      <!-- Header -->
      <div class="flex items-center justify-between p-3 border-b border-gray-100">
        <button 
          @click="navigatePrevious"
          class="p-1 rounded hover:bg-gray-100 transition-colors"
        >
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
          </svg>
        </button>

        <h2 class="text-sm font-medium text-gray-900">
          <span v-if="monthPicker">{{ displayYear }}</span>
          <span v-else>{{ monthNames[displayMonth] }} {{ displayYear }}</span>
        </h2>

        <button 
          @click="navigateNext"
          class="p-1 rounded hover:bg-gray-100 transition-colors"
        >
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
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
            class="p-2 text-xs rounded hover:bg-blue-50 transition-colors"
          >
            {{ month.shortName }}
          </button>
        </div>

        <!-- Day Picker -->
        <template v-else>
          <!-- Day Headers -->
          <div class="grid grid-cols-7 gap-1 mb-1">
            <div 
              v-for="day in dayNames"
              :key="day"
              class="h-8 flex items-center justify-center text-xs font-medium text-gray-500"
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
              class="h-8 flex items-center justify-center text-xs rounded transition-colors"
            >
              {{ day.day }}
            </button>
          </div>
        </template>
        
        <!-- Range Selection Info -->
        <div v-if="range && isSelectingRange" class="mt-2 p-2 bg-primary-50 rounded text-xs text-primary-700 text-center">
          <span v-if="monthPicker">Select the end month for your range</span>
          <span v-else>Select the end date for your range</span>
        </div>
        <div v-else-if="range && !isSelectingRange && !hasRangeStart" class="mt-2 p-2 bg-gray-50 rounded text-xs text-gray-600 text-center">
          <span v-if="monthPicker">Select the start month for your range</span>
          <span v-else>Select the start date for your range</span>
        </div>
      </div>

      <!-- Footer -->
      <div class="flex items-center justify-between p-3 border-t border-gray-100 bg-gray-50 rounded-b-lg">
        <button 
          @click="setToday"
          class="px-3 py-1 text-xs font-medium text-white bg-primary-600 rounded hover:bg-primary-700 transition-colors"
        >
          Today
        </button>
        <button 
          @click="clearDate"
          class="px-3 py-1 text-xs font-medium text-gray-700 bg-white border border-gray-300 rounded hover:bg-gray-50 transition-colors"
        >
          Clear
        </button>
      </div>
    </div>

    <!-- Mobile Backdrop -->
    <div 
      v-if="isOpen"
      @click="closePicker"
      class="fixed inset-0 z-40 sm:hidden bg-black bg-opacity-25"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'

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
  enableDateValidation: true
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
  'January', 'February', 'March', 'April', 'May', 'June',
  'July', 'August', 'September', 'October', 'November', 'December'
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
  return props.modelValue as string || ''
})

const hasRangeStart = computed(() => {
  if (!props.range) return false
  const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
  return Boolean(values[0])
})

const monthGrid = computed(() => {
  return monthNames.map((name, index) => ({
    month: index,
    name,
    shortName: name.slice(0, 3),
    date: `${displayYear.value}-${String(index + 1).padStart(2, '0')}`
  }))
})

const calendarDays = computed(() => {
  const firstDay = new Date(displayYear.value, displayMonth.value, 1)
  const startDate = new Date(firstDay)
  startDate.setDate(startDate.getDate() - firstDay.getDay())

  const days = []
  const current = new Date(startDate)

  for (let i = 0; i < 42; i++) {
    days.push({
      date: new Date(current),
      day: current.getDate(),
      month: current.getMonth(),
      year: current.getFullYear(),
      isCurrentMonth: current.getMonth() === displayMonth.value,
      isToday: isToday(current),
      dateString: formatDate(current)
    })
    current.setDate(current.getDate() + 1)
  }

  return days
})

// Helper Functions
const isDateDisabled = (dateString: string): boolean => {
  if (!props.enableDateValidation) return false
  
  const date = new Date(dateString)
  const minDate = minDateForValidation.value ? new Date(minDateForValidation.value) : null
  const maxDate = maxDateForValidation.value ? new Date(maxDateForValidation.value) : null
  
  if (minDate && date < minDate) return true
  if (maxDate && date > maxDate) return true
  
  return false
}

const isToday = (date: Date): boolean => {
  const today = new Date()
  return date.toDateString() === today.toDateString()
}

const formatDate = (date: Date): string => {
  return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`
}

const formatMonth = (year: number, month: number): string => {
  return `${year}-${String(month + 1).padStart(2, '0')}`
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

const selectDate = (day: any) => {
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

const selectMonth = (month: any) => {
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
  const today = new Date()
  
  if (props.monthPicker) {
    const monthString = formatMonth(today.getFullYear(), today.getMonth())
    if (props.range) {
      emit('update:modelValue', [monthString, monthString])
    } else {
      emit('update:modelValue', monthString)
    }
  } else {
    const dateString = formatDate(today)
    if (props.range) {
      emit('update:modelValue', [dateString, dateString])
    } else {
      emit('update:modelValue', dateString)
    }
  }
  
  displayMonth.value = today.getMonth()
  displayYear.value = today.getFullYear()
  closePicker()
}

const clearDate = () => {
  if (props.range) {
    emit('update:modelValue', [null, null])
  } else {
    emit('update:modelValue', null)
  }
  isSelectingRange.value = false
  closePicker()
}

// Styling Methods
const getDayClasses = (day: any) => {
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

  if (isSelected || isRangeStart || isRangeEnd) {
    return 'bg-primary-600 text-white font-medium'
  } else if (isInRange) {
    return 'bg-primary-100 text-primary-800'
  } else if (day.isToday) {
    return 'bg-primary-500 text-white font-medium'
  } else {
    return 'text-gray-700 hover:bg-gray-100'
  }
}

const getMonthClasses = (month: any) => {
  const isSelected = isMonthSelected(month.date)
  const isInRange = isMonthInRange(month.date)
  const isRangeStart = isMonthRangeStart(month.date)
  const isRangeEnd = isMonthRangeEnd(month.date)

  if (isSelected || isRangeStart || isRangeEnd) {
    return 'bg-primary-600 text-white font-medium'
  } else if (isInRange) {
    return 'bg-primary-100 text-primary-800'
  } else {
    return 'text-gray-700 hover:bg-gray-100'
  }
}

// Selection Helpers
const isDateSelected = (dateString: string): boolean => {
  if (props.range) {
    const values = Array.isArray(props.modelValue) ? props.modelValue : [null, null]
    return values.some(v => v === dateString)
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
    return values.some(v => v === monthString)
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
watch(() => props.modelValue, () => {
  isSelectingRange.value = false
})
</script>