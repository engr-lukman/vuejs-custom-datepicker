<template>
  <div class="min-h-screen bg-gray-50 px-4 py-12 sm:px-6 lg:px-8">
    <div class="mx-auto max-w-4xl">
      <!-- Application Header with Title and Description -->
      <header class="mb-12 text-center">
        <h1 class="text-3xl font-bold text-gray-900 sm:text-4xl">Custom Date Picker</h1>
        <p class="mt-4 text-lg text-gray-600">
          A clean, reusable date picker component for range and month selections
        </p>
      </header>

      <!-- Main Content Grid - Examples of Different Date Picker Configurations -->
      <main class="grid gap-8 md:grid-cols-2 xl:grid-cols-3">
        <!-- Example 1: Date Range Selection for Last 30 Days -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Date Range - 1 Month</h2>
            <p class="text-sm text-gray-600">Select any date range within the last 30 days</p>
          </header>
          <DatePicker
            v-model="selectedOneMonthDateRange"
            placeholder="Select date range"
            :min-date="datePickerConfigurations.oneMonth.minDate"
            :max-date="datePickerConfigurations.oneMonth.maxDate"
            aria-label="Select one month date range"
          />
          <div
            v-if="selectedOneMonthDateRange?.[0] || selectedOneMonthDateRange?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
            role="status"
            aria-live="polite"
          >
            <strong>Selected:</strong>
            {{ selectedOneMonthDateRange?.[0] || 'Not set' }} to
            {{ selectedOneMonthDateRange?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Example 2: Single Month Selection from Available Range -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Single Month Selection</h2>
            <p class="text-sm text-gray-600">
              Select a single month from current + previous 5 months
            </p>
          </header>
          <DatePicker
            v-model="selectedSingleMonthRange"
            view="month"
            placeholder="Select month"
            :min-date="datePickerConfigurations.singleMonthSelection.minDate"
            :max-date="datePickerConfigurations.singleMonthSelection.maxDate"
            aria-label="Select single month from available range"
          />
          <div
            v-if="selectedSingleMonthRange?.[0] || selectedSingleMonthRange?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
            role="status"
            aria-live="polite"
          >
            <strong>Selected:</strong>
            {{ selectedSingleMonthRange?.[0] || 'Not set' }} to
            {{ selectedSingleMonthRange?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Example 3: Extended Date Range Selection for Last 60 Days -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Date Range - 2 Months</h2>
            <p class="text-sm text-gray-600">Select any date range within the last 60 days</p>
          </header>
          <DatePicker
            v-model="selectedTwoMonthDateRange"
            placeholder="Select date range"
            :min-date="datePickerConfigurations.twoMonths.minDate"
            :max-date="datePickerConfigurations.twoMonths.maxDate"
            aria-label="Select two month date range"
          />
          <div
            v-if="selectedTwoMonthDateRange?.[0] || selectedTwoMonthDateRange?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
            role="status"
            aria-live="polite"
          >
            <strong>Selected:</strong>
            {{ selectedTwoMonthDateRange?.[0] || 'Not set' }} to
            {{ selectedTwoMonthDateRange?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Example 4: Two Month Range Selection (Current + Previous Month) -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Two Month Range</h2>
            <p class="text-sm text-gray-600">Select from current and previous month</p>
          </header>
          <DatePicker
            v-model="selectedTwoMonthRange"
            view="month"
            placeholder="Select month range"
            :min-date="datePickerConfigurations.twoMonthRange.minDate"
            :max-date="datePickerConfigurations.twoMonthRange.maxDate"
            aria-label="Select two month range"
          />
          <div
            v-if="selectedTwoMonthRange?.[0] || selectedTwoMonthRange?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
            role="status"
            aria-live="polite"
          >
            <strong>Selected:</strong>
            {{ selectedTwoMonthRange?.[0] || 'Not set' }} to
            {{ selectedTwoMonthRange?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Example 5: Extended Date Range Selection for Last 180 Days -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Date Range - 6 Months</h2>
            <p class="text-sm text-gray-600">Select any date range within the last 180 days</p>
          </header>
          <DatePicker
            v-model="selectedSixMonthDateRange"
            placeholder="Select date range"
            :min-date="datePickerConfigurations.sixMonths.minDate"
            :max-date="datePickerConfigurations.sixMonths.maxDate"
            aria-label="Select six month date range"
          />
          <div
            v-if="selectedSixMonthDateRange?.[0] || selectedSixMonthDateRange?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
            role="status"
            aria-live="polite"
          >
            <strong>Selected:</strong>
            {{ selectedSixMonthDateRange?.[0] || 'Not set' }} to
            {{ selectedSixMonthDateRange?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Example 6: Six Month Range Selection (Current + Previous 5 Months) -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Six Month Range</h2>
            <p class="text-sm text-gray-600">Select from current and previous 5 months</p>
          </header>
          <DatePicker
            v-model="selectedSixMonthRange"
            view="month"
            placeholder="Select month range"
            :min-date="datePickerConfigurations.sixMonthRange.minDate"
            :max-date="datePickerConfigurations.sixMonthRange.maxDate"
            aria-label="Select six month range"
          />
          <div
            v-if="selectedSixMonthRange?.[0] || selectedSixMonthRange?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
            role="status"
            aria-live="polite"
          >
            <strong>Selected:</strong>
            {{ selectedSixMonthRange?.[0] || 'Not set' }} to
            {{ selectedSixMonthRange?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Example 7: Single Date Selection -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Single Date Selection</h2>
            <p class="text-sm text-gray-600">Select a single date from the last 30 days</p>
          </header>
          <DatePicker
            v-model="selectedSingleDate"
            mode="single"
            :min-date="datePickerConfigurations.oneMonth.minDate"
            :max-date="datePickerConfigurations.oneMonth.maxDate"
            aria-label="Select single date"
          />
          <div
            v-if="selectedSingleDate"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
            role="status"
            aria-live="polite"
          >
            <strong>Selected:</strong> {{ selectedSingleDate }}
          </div>
        </section>

        <!-- Example 8: Single Month Selection -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Single Month Selection</h2>
            <p class="text-sm text-gray-600">
              Select a single month from the current + previous 5 months
            </p>
          </header>
          <DatePicker
            v-model="selectedSingleMonth"
            view="month"
            mode="single"
            :min-date="datePickerConfigurations.singleMonthSelection.minDate"
            :max-date="datePickerConfigurations.singleMonthSelection.maxDate"
            aria-label="Select single month"
          />
          <div
            v-if="selectedSingleMonth"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
            role="status"
            aria-live="polite"
          >
            <strong>Selected:</strong> {{ selectedSingleMonth }}
          </div>
        </section>
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from '@/components/DatePicker.vue'

// Define types for date picker functionality
type SelectedDateRange = string[] | null
type SelectedSingleValue = string | null
type DatePickerConfiguration = {
  minDate: string
  maxDate: string
}

// Define constant values for different date range periods
const AVAILABLE_DAY_PERIODS = {
  THIRTY_DAYS: 30,
  SIXTY_DAYS: 60,
  ONE_HUNDRED_EIGHTY_DAYS: 180,
} as const

// Define constant values for month lookback periods
const AVAILABLE_MONTH_PERIODS = {
  PREVIOUS_ONE_MONTH: 1,
  PREVIOUS_FIVE_MONTHS: 5,
} as const

// Helper function to format a date object into YYYY-MM-DD string format
const formatDateToString = (date: Date): string => {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${year}-${month}-${day}`
}

// Helper function to format a date object into YYYY-MM string format for month selection
const formatDateToMonthString = (date: Date): string => {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  return `${year}-${month}`
}

// Creates configuration for date pickers with day-based ranges
const createDayBasedDateConfiguration = (
  totalDays: number,
  baseDate: Date = new Date()
): DatePickerConfiguration => {
  const earliestDate = new Date(baseDate)
  earliestDate.setDate(baseDate.getDate() - (totalDays - 1)) // -1 to include today

  return {
    minDate: formatDateToString(earliestDate),
    maxDate: formatDateToString(baseDate),
  }
}

// Creates configuration for date pickers with month-based ranges
const createMonthBasedDateConfiguration = (
  monthsBack: number,
  baseDate: Date = new Date()
): DatePickerConfiguration => {
  const earliestMonth = new Date(baseDate.getFullYear(), baseDate.getMonth() - monthsBack, 1)

  return {
    minDate: formatDateToMonthString(earliestMonth),
    maxDate: formatDateToMonthString(baseDate),
  }
}

// Factory function to generate all date picker configurations
const createAllDatePickerConfigurations = () => {
  const currentDate = new Date()

  return {
    oneMonth: createDayBasedDateConfiguration(AVAILABLE_DAY_PERIODS.THIRTY_DAYS, currentDate),
    twoMonths: createDayBasedDateConfiguration(AVAILABLE_DAY_PERIODS.SIXTY_DAYS, currentDate),
    sixMonths: createDayBasedDateConfiguration(
      AVAILABLE_DAY_PERIODS.ONE_HUNDRED_EIGHTY_DAYS,
      currentDate
    ),
    singleMonthSelection: createMonthBasedDateConfiguration(
      AVAILABLE_MONTH_PERIODS.PREVIOUS_FIVE_MONTHS,
      currentDate
    ),
    twoMonthRange: createMonthBasedDateConfiguration(
      AVAILABLE_MONTH_PERIODS.PREVIOUS_ONE_MONTH,
      currentDate
    ),
    sixMonthRange: createMonthBasedDateConfiguration(
      AVAILABLE_MONTH_PERIODS.PREVIOUS_FIVE_MONTHS,
      currentDate
    ),
  }
}

// Generate all date picker configurations using the factory function
const datePickerConfigurations = createAllDatePickerConfigurations()

// Reactive state variables to store user selections for different date range examples
const selectedOneMonthDateRange = ref<SelectedDateRange>(null)
const selectedTwoMonthDateRange = ref<SelectedDateRange>(null)
const selectedSixMonthDateRange = ref<SelectedDateRange>(null)
const selectedSingleMonthRange = ref<SelectedDateRange>(null)
const selectedTwoMonthRange = ref<SelectedDateRange>(null)
const selectedSixMonthRange = ref<SelectedDateRange>(null)

// Reactive state variables for single selections
const selectedSingleDate = ref<SelectedSingleValue>(null)
const selectedSingleMonth = ref<SelectedSingleValue>(null)
</script>
