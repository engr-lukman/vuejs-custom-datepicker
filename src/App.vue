<template>
  <div class="min-h-screen bg-gray-50 px-4 py-12 sm:px-6 lg:px-8">
    <div class="mx-auto max-w-6xl">
      <!-- Application Header -->
      <header class="mb-12 text-center">
        <h1 class="text-2xl font-bold text-gray-900">Custom Date/Month Picker</h1>
        <p class="mt-4 text-lg text-gray-600">
          A flexible Vue.js date/month picker component with single and range selection modes
        </p>
      </header>

      <!-- Main Content Grid -->
      <main class="grid gap-8 md:grid-cols-2 lg:grid-cols-3">
        <!-- Single Date Selection (30 days) -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Single Date (Last 30 Days)</h2>
            <p class="text-sm text-gray-600">Select a single date within the last 30 days</p>
          </header>
          <DatePicker
            mode="single"
            v-model="selectedSingleDate"
            :min-navigation="date30Config.minNavigation"
            :max-navigation="date30Config.maxNavigation"
          />
          <div v-if="selectedSingleDate" class="mt-4 rounded bg-gray-50 p-3 text-sm">
            <strong>Selected:</strong> {{ selectedSingleDate }}
          </div>
        </section>

        <!-- Date Range Selection (30 days) -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Date Range (Last 30 Days)</h2>
            <p class="text-sm text-gray-600">Select a date range within the last 30 days</p>
          </header>
          <DatePicker
            mode="range"
            v-model="selectedDateRange30"
            :min-navigation="date30Config.minNavigation"
            :max-navigation="date30Config.maxNavigation"
          />
          <div
            v-if="selectedDateRange30?.[0] || selectedDateRange30?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
          >
            <strong>Selected:</strong>
            {{ selectedDateRange30?.[0] || 'Not set' }} to
            {{ selectedDateRange30?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Date Range Selection (180 days) -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Date Range (Last 180 Days)</h2>
            <p class="text-sm text-gray-600">Select a date range within the last 180 days</p>
          </header>
          <DatePicker
            mode="range"
            v-model="selectedDateRange180"
            :min-navigation="date180Config.minNavigation"
            :max-navigation="date180Config.maxNavigation"
          />
          <div
            v-if="selectedDateRange180?.[0] || selectedDateRange180?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
          >
            <strong>Selected:</strong>
            {{ selectedDateRange180?.[0] || 'Not set' }} to
            {{ selectedDateRange180?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Single Month Selection (Last 5 months, excluding current) -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Single Month (Last 5 Months)</h2>
            <p class="text-sm text-gray-600">
              Select a single month from the last 5 months (excluding current)
            </p>
          </header>
          <MonthPicker
            v-model="selectedSingleMonth"
            :min-navigation="month5Config.minNavigation"
            :max-navigation="month5Config.maxNavigation"
          />
          <div v-if="selectedSingleMonth" class="mt-4 rounded bg-gray-50 p-3 text-sm">
            <strong>Selected:</strong> {{ selectedSingleMonth }}
          </div>
        </section>

        <!-- Month Range Selection (2 months) -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Month Range (Last 2 Months)</h2>
            <p class="text-sm text-gray-600">Select a range of months within the last 2 months</p>
          </header>
          <MonthPicker
            mode="range"
            v-model="selectedMonthRange2"
            :min-navigation="month2Config.minNavigation"
            :max-navigation="month2Config.maxNavigation"
          />
          <div
            v-if="selectedMonthRange2?.[0] || selectedMonthRange2?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
          >
            <strong>Selected:</strong>
            {{ selectedMonthRange2?.[0] || 'Not set' }} to
            {{ selectedMonthRange2?.[1] || 'Not set' }}
          </div>
        </section>

        <!-- Month Range Selection (6 months) -->
        <section class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <header class="mb-4">
            <h2 class="text-lg font-semibold text-gray-900">Month Range (Last 6 Months)</h2>
            <p class="text-sm text-gray-600">Select a range of months within the last 6 months</p>
          </header>
          <MonthPicker
            mode="range"
            v-model="selectedMonthRange6"
            :min-navigation="month6Config.minNavigation"
            :max-navigation="month6Config.maxNavigation"
          />
          <div
            v-if="selectedMonthRange6?.[0] || selectedMonthRange6?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
          >
            <strong>Selected:</strong>
            {{ selectedMonthRange6?.[0] || 'Not set' }} to
            {{ selectedMonthRange6?.[1] || 'Not set' }}
          </div>
        </section>
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

import DatePicker from '@/components/DatePicker.vue'
import MonthPicker from '@/components/MonthPicker.vue'

type SelectedDateRange = string[] | null
type SelectedSingleValue = string | null

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

// Date configs
const createDateConfig = (daysBack: number) => {
  const today = new Date()
  const startDate = new Date(today)
  startDate.setDate(today.getDate() - daysBack)
  return {
    minNavigation: formatDateToString(startDate),
    maxNavigation: formatDateToString(today),
  }
}

// Month configs
const createMonthConfig = (monthsBack: number, excludeCurrent = false) => {
  const today = new Date()
  let endMonth = today.getMonth()
  let endYear = today.getFullYear()
  if (excludeCurrent) {
    endMonth -= 1
    if (endMonth < 0) {
      endMonth = 11
      endYear -= 1
    }
  }
  const startDate = new Date(endYear, endMonth - monthsBack + 1, 1)
  const endDate = new Date(endYear, endMonth, 1)
  return {
    minNavigation: formatDateToMonthString(startDate),
    maxNavigation: formatDateToMonthString(endDate),
  }
}

// Configurations
const date30Config = createDateConfig(29) // Last 30 days
const date180Config = createDateConfig(180) // Last 180 days

const month5Config = createMonthConfig(5, true) // Last 5 months, excluding current
const month2Config = createMonthConfig(2) // Last 2 months (including current)
const month6Config = createMonthConfig(6) // Last 6 months (including current)

const selectedSingleDate = ref<SelectedSingleValue>(null)
const selectedDateRange30 = ref<SelectedDateRange>(null)
const selectedDateRange180 = ref<SelectedDateRange>(null)
const selectedSingleMonth = ref<SelectedSingleValue>(null)
const selectedMonthRange2 = ref<SelectedDateRange>(null)
const selectedMonthRange6 = ref<SelectedDateRange>(null)
</script>
