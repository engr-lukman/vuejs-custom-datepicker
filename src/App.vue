<template>
  <div class="min-h-screen bg-gray-50 px-4 py-12 sm:px-6 lg:px-8">
    <div class="mx-auto max-w-3xl">
      <!-- Header -->
      <div class="mb-12 text-center">
        <h1 class="text-3xl font-bold text-gray-900 sm:text-4xl">Vue 3 Date Picker</h1>
        <p class="mt-4 text-lg text-gray-600">
          A clean, reusable date picker component with multiple selection modes
        </p>
      </div>

      <!-- Examples Grid -->
      <div class="grid gap-8 md:grid-cols-2">
        <!-- Single Date with Custom Range -->
        <div class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <h2 class="mb-4 text-lg font-semibold text-gray-900">Single Date</h2>
          <DatePicker
            v-model="singleDate"
            placeholder="Select a date"
            :min-date="minDate"
            :max-date="maxDate"
          />
          <div v-if="singleDate" class="mt-4 rounded bg-gray-50 p-3 text-sm">
            <strong>Selected:</strong> {{ singleDate }}
          </div>
        </div>

        <!-- Date Range with Custom Range -->
        <div class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <h2 class="mb-4 text-lg font-semibold text-gray-900">Date Range</h2>
          <DatePicker
            v-model="dateRange"
            :range="true"
            placeholder="Select date range"
            :min-date="minDate"
            :max-date="maxDate"
          />
          <div v-if="dateRange?.[0] || dateRange?.[1]" class="mt-4 rounded bg-gray-50 p-3 text-sm">
            <strong>Selected:</strong>
            {{ dateRange?.[0] || 'Not set' }} to {{ dateRange?.[1] || 'Not set' }}
          </div>
        </div>

        <!-- Single Month -->
        <div class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <h2 class="mb-4 text-lg font-semibold text-gray-900">Single Month</h2>
          <DatePicker
            v-model="singleMonth"
            view="month"
            placeholder="Select a month"
            :min-date="minDate"
            :max-date="maxDate"
          />
          <div v-if="singleMonth" class="mt-4 rounded bg-gray-50 p-3 text-sm">
            <strong>Selected:</strong> {{ singleMonth }}
          </div>
        </div>

        <!-- Month Range -->
        <div class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
          <h2 class="mb-4 text-lg font-semibold text-gray-900">Month Range</h2>
          <DatePicker
            v-model="monthRange"
            view="month"
            :range="true"
            placeholder="Select month range"
            :min-date="minDate"
            :max-date="maxDate"
          />

          <div
            v-if="monthRange?.[0] || monthRange?.[1]"
            class="mt-4 rounded bg-gray-50 p-3 text-sm"
          >
            <strong>Selected:</strong>
            {{ monthRange?.[0] || 'Not set' }} to {{ monthRange?.[1] || 'Not set' }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from '@/components/DatePicker.vue'

// State
const singleDate = ref<string | null>(null)
const dateRange = ref<string[] | null>(null)
const singleMonth = ref<string | null>(null)
const monthRange = ref<string[] | null>(null)

// Date validation ranges
// - Date selection: 180 days
// - Month selection: 6 months (current month + previous 5 months)
const today = new Date()
const minDateObj = new Date(today)
minDateObj.setDate(today.getDate() - 180)
const maxDateObj = new Date(today)

const minDate = minDateObj.toISOString().split('T')[0]
const maxDate = maxDateObj.toISOString().split('T')[0]
</script>
