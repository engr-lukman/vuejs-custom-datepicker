<template>
  <div class="min-h-screen bg-gray-50 px-4 py-12 sm:px-6 lg:px-8">
    <div class="mx-auto max-w-6xl">
      <!-- Application Header -->
      <header class="mb-12 text-center">
        <h1 class="text-3xl font-bold text-gray-900 sm:text-4xl">Custom Date Picker</h1>
        <p class="mt-4 text-lg text-gray-600">
          A flexible Vue.js date picker component with single and range selection modes
        </p>
      </header>

      <!-- Main Content Grid -->
      <main class="grid gap-8 md:grid-cols-2 lg:grid-cols-3">
        <!-- Single Date Selection -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Single Date</h2>
            <p class="text-sm text-gray-600">Select a single date</p>
          </header>
          <DatePicker
            v-model="selectedSingleDate"
            mode="single"
            :min-date="dateConfig.minDate"
            :max-date="dateConfig.maxDate"
            aria-label="Select single date"
          />
          <div v-if="selectedSingleDate" class="mt-4 rounded bg-gray-50 p-3 text-sm">
            <strong>Selected:</strong> {{ selectedSingleDate }}
          </div>
        </section>

        <!-- Date Range Selection -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Date Range</h2>
            <p class="text-sm text-gray-600">Select a date range with quick options</p>
          </header>
          <DatePicker
            v-model="selectedDateRange"
            :min-date="dateConfig.minDate"
            :max-date="dateConfig.maxDate"
            aria-label="Select date range"
          />
          <div
            v-if="selectedDateRange?.[0] || selectedDateRange?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
          >
            <strong>Selected:</strong>
            {{ selectedDateRange?.[0] || 'Not set' }} to
            {{ selectedDateRange?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Extended Date Range -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Extended Date Range</h2>
            <p class="text-sm text-gray-600">Select from last 6 months</p>
          </header>
          <DatePicker
            v-model="selectedExtendedRange"
            :min-date="extendedConfig.minDate"
            :max-date="extendedConfig.maxDate"
            aria-label="Select extended date range"
          />
          <div
            v-if="selectedExtendedRange?.[0] || selectedExtendedRange?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
          >
            <strong>Selected:</strong>
            {{ selectedExtendedRange?.[0] || 'Not set' }} to
            {{ selectedExtendedRange?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Single Month Selection -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Single Month</h2>
            <p class="text-sm text-gray-600">Select a single month</p>
          </header>
          <DatePicker
            v-model="selectedSingleMonth"
            view="month"
            mode="single"
            :min-date="monthConfig.minDate"
            :max-date="monthConfig.maxDate"
            aria-label="Select single month"
          />
          <div v-if="selectedSingleMonth" class="mt-4 rounded bg-gray-50 p-3 text-sm">
            <strong>Selected:</strong> {{ selectedSingleMonth }}
          </div>
        </section>

        <!-- Month Range Selection -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Month Range</h2>
            <p class="text-sm text-gray-600">Select a range of months</p>
          </header>
          <DatePicker
            v-model="selectedMonthRange"
            view="month"
            :min-date="monthConfig.minDate"
            :max-date="monthConfig.maxDate"
            aria-label="Select month range"
          />
          <div
            v-if="selectedMonthRange?.[0] || selectedMonthRange?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
          >
            <strong>Selected:</strong>
            {{ selectedMonthRange?.[0] || 'Not set' }} to
            {{ selectedMonthRange?.[1] || 'Not set' }}
          </div>
        </section>
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from '@/components/DatePicker.vue'

// Types
type SelectedDateRange = string[] | null
type SelectedSingleValue = string | null

// Helper functions
const formatDateToString = (date: Date): string => {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${year}-${month}-${day}`
}

const formatDateToMonthString = (date: Date): string => {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  return `${year}-${month}`
}

// Configuration factory
const createDateConfig = (daysBack: number) => {
  const today = new Date()
  const startDate = new Date(today)
  startDate.setDate(today.getDate() - daysBack)

  return {
    minDate: formatDateToString(startDate),
    maxDate: formatDateToString(today),
  }
}

const createMonthConfig = (monthsBack: number) => {
  const today = new Date()
  const startDate = new Date(today.getFullYear(), today.getMonth() - monthsBack, 1)

  return {
    minDate: formatDateToMonthString(startDate),
    maxDate: formatDateToMonthString(today),
  }
}

// Configurations
const dateConfig = createDateConfig(29) // Last 30 days
const monthConfig = createMonthConfig(5) // Last 5 months
const extendedConfig = createDateConfig(180) // Last 6 months

// Reactive state
const selectedDateRange = ref<SelectedDateRange>(null)
const selectedSingleDate = ref<SelectedSingleValue>(null)
const selectedMonthRange = ref<SelectedDateRange>(null)
const selectedSingleMonth = ref<SelectedSingleValue>(null)
const selectedExtendedRange = ref<SelectedDateRange>(null)
</script>
