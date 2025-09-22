<template>
  <div class="min-h-screen bg-gray-50 px-4 py-12 sm:px-6 lg:px-8">
    <div class="mx-auto max-w-4xl">
      <!-- Header Section -->
      <header class="mb-12 text-center">
        <h1 class="text-3xl font-bold text-gray-900 sm:text-4xl">Custom Date Picker</h1>
        <p class="mt-4 text-lg text-gray-600">
          A clean, reusable date picker component for range and month selections
        </p>
      </header>

      <!-- Date Picker Examples Grid -->
      <main class="grid gap-8 lg:grid-cols-2">
        <!-- Date Range - 1 Month Example -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Date Range - 1 Month</h2>
            <p class="text-sm text-gray-600">Select any date range within the last 30 days</p>
          </header>
          <DatePicker
            v-model="oneMonthDateRange"
            range
            placeholder="Select date range"
            :min-date="dateRangeConfigs.oneMonth.minDate"
            :max-date="dateRangeConfigs.oneMonth.maxDate"
            aria-label="Select one month date range"
          />
          <div
            v-if="oneMonthDateRange?.[0] || oneMonthDateRange?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
            role="status"
            aria-live="polite"
          >
            <strong>Selected:</strong>
            {{ oneMonthDateRange?.[0] || 'Not set' }} to {{ oneMonthDateRange?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Single Month Selection Example -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Single Month Selection</h2>
            <p class="text-sm text-gray-600">
              Select a single month from current + previous 5 months
            </p>
          </header>
          <DatePicker
            v-model="currentMonthRangeState"
            view="month"
            range
            placeholder="Select month"
            :min-date="dateRangeConfigs.singleMonthSelection.minDate"
            :max-date="dateRangeConfigs.singleMonthSelection.maxDate"
            aria-label="Select single month from available range"
          />
          <div
            v-if="currentMonthRangeState?.[0] || currentMonthRangeState?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
            role="status"
            aria-live="polite"
          >
            <strong>Selected:</strong>
            {{ currentMonthRangeState?.[0] || 'Not set' }} to
            {{ currentMonthRangeState?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Date Range - 2 Months Example -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Date Range - 2 Months</h2>
            <p class="text-sm text-gray-600">Select any date range within the last 60 days</p>
          </header>
          <DatePicker
            v-model="twoMonthDateRange"
            range
            placeholder="Select date range"
            :min-date="dateRangeConfigs.twoMonthDateRange.minDate"
            :max-date="dateRangeConfigs.twoMonthDateRange.maxDate"
            aria-label="Select two month date range"
          />
          <div
            v-if="twoMonthDateRange?.[0] || twoMonthDateRange?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
            role="status"
            aria-live="polite"
          >
            <strong>Selected:</strong>
            {{ twoMonthDateRange?.[0] || 'Not set' }} to {{ twoMonthDateRange?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Month Range - 2 Months Example -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Two Month Range</h2>
            <p class="text-sm text-gray-600">Select from current and previous month</p>
          </header>
          <DatePicker
            v-model="twoMonthRange"
            view="month"
            range
            placeholder="Select month range"
            :min-date="dateRangeConfigs.twoMonths.minDate"
            :max-date="dateRangeConfigs.twoMonths.maxDate"
            aria-label="Select two month range"
          />
          <div
            v-if="twoMonthRange?.[0] || twoMonthRange?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
            role="status"
            aria-live="polite"
          >
            <strong>Selected:</strong>
            {{ twoMonthRange?.[0] || 'Not set' }} to {{ twoMonthRange?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Date Range - 6 Months Example -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Date Range - 6 Months</h2>
            <p class="text-sm text-gray-600">Select any date range within the last 180 days</p>
          </header>
          <DatePicker
            v-model="sixMonthDateRange"
            range
            placeholder="Select date range"
            :min-date="dateRangeConfigs.sixMonths.minDate"
            :max-date="dateRangeConfigs.sixMonths.maxDate"
            aria-label="Select six month date range"
          />
          <div
            v-if="sixMonthDateRange?.[0] || sixMonthDateRange?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
            role="status"
            aria-live="polite"
          >
            <strong>Selected:</strong>
            {{ sixMonthDateRange?.[0] || 'Not set' }} to {{ sixMonthDateRange?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Six Month Range Example -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Six Month Range</h2>
            <p class="text-sm text-gray-600">Select from current and previous 5 months</p>
          </header>
          <DatePicker
            v-model="sixMonthRangeState"
            view="month"
            range
            placeholder="Select month range"
            :min-date="dateRangeConfigs.sixMonthsRange.minDate"
            :max-date="dateRangeConfigs.sixMonthsRange.maxDate"
            aria-label="Select six month range"
          />
          <div
            v-if="sixMonthRangeState?.[0] || sixMonthRangeState?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
            role="status"
            aria-live="polite"
          >
            <strong>Selected:</strong>
            {{ sixMonthRangeState?.[0] || 'Not set' }} to {{ sixMonthRangeState?.[1] || 'Not set' }}
          </div>
        </section>
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from '@/components/DatePicker.vue'

// Type definitions
type DateRange = string[] | null
type DateRangeConfig = {
  minDate: string
  maxDate: string
}

// Constants
const DATE_RANGES = {
  ONE_MONTH: 30,
  TWO_MONTHS: 60,
  SIX_MONTHS: 180,
} as const

const MONTH_RANGES = {
  CURRENT: 0,
  TWO_MONTHS: 1,
  SIX_MONTHS: 5,
} as const

// Utility functions for date formatting
const formatDate = (date: Date): string => {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${year}-${month}-${day}`
}

const formatMonth = (date: Date): string => {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  return `${year}-${month}`
}

// Main utility function to create min and max date range
const createDateRange = (days: number, baseDate: Date = new Date()): DateRangeConfig => {
  const minDate = new Date(baseDate)
  minDate.setDate(baseDate.getDate() - (days - 1)) // -1 to include today

  return {
    minDate: formatDate(minDate),
    maxDate: formatDate(baseDate),
  }
}

// Utility function to create month range
const createMonthRange = (monthsBack: number, baseDate: Date = new Date()): DateRangeConfig => {
  const minMonth = new Date(baseDate.getFullYear(), baseDate.getMonth() - monthsBack, 1)

  return {
    minDate: formatMonth(minMonth),
    maxDate: formatMonth(baseDate),
  }
}

// Date range factory function for easy configuration
const createDateRangeConfig = () => {
  const today = new Date()

  return {
    oneMonth: createDateRange(DATE_RANGES.ONE_MONTH, today),
    twoMonthDateRange: createDateRange(DATE_RANGES.TWO_MONTHS, today),
    sixMonths: createDateRange(DATE_RANGES.SIX_MONTHS, today),
    singleMonthSelection: createMonthRange(MONTH_RANGES.SIX_MONTHS, today),
    twoMonths: createMonthRange(MONTH_RANGES.TWO_MONTHS, today),
    sixMonthsRange: createMonthRange(MONTH_RANGES.SIX_MONTHS, today),
  }
}

// Generate all date ranges using the factory
const dateRangeConfigs = createDateRangeConfig() // Reactive state for different date range examples
const oneMonthDateRange = ref<DateRange>(null)
const twoMonthDateRange = ref<DateRange>(null)
const sixMonthDateRange = ref<DateRange>(null)
const currentMonthRangeState = ref<DateRange>(null)
const twoMonthRange = ref<DateRange>(null)
const sixMonthRangeState = ref<DateRange>(null)
</script>
