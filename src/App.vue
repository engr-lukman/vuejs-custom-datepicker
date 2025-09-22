<template>
  <div class="min-h-screen bg-gray-50 py-12 px-4 sm:px-6 lg:px-8">
    <div class="max-w-3xl mx-auto">
      <!-- Header -->
      <div class="text-center mb-12">
        <h1 class="text-3xl font-bold text-gray-900 sm:text-4xl">Vue 3 Date Picker</h1>
        <p class="mt-4 text-lg text-gray-600">
          A clean, reusable date picker component with multiple selection modes
        </p>
      </div>

      <!-- Examples Grid -->
      <div class="grid gap-8 md:grid-cols-2">
        <!-- Single Date with Validation -->
        <div class="bg-white rounded-lg shadow-sm border border-gray-200 p-6">
          <h2 class="text-lg font-semibold text-gray-900 mb-4">Single Date (Validated)</h2>
          <DatePicker
            v-model="singleDate"
            placeholder="Select a date (last 180 days)"
            :enable-date-validation="true"
          />
          <div v-if="singleDate" class="mt-4 p-3 bg-gray-50 rounded text-sm">
            <strong>Selected:</strong> {{ singleDate }}
          </div>
          <div class="mt-2 text-xs text-gray-500">Valid range: Last 180 days to today</div>
        </div>

        <!-- Date Range with Validation -->
        <div class="bg-white rounded-lg shadow-sm border border-gray-200 p-6">
          <h2 class="text-lg font-semibold text-gray-900 mb-4">Date Range (Validated)</h2>
          <DatePicker
            v-model="dateRange"
            :range="true"
            placeholder="Select date range (last 180 days)"
            :enable-date-validation="true"
          />
          <div v-if="dateRange?.[0] || dateRange?.[1]" class="mt-4 p-3 bg-gray-50 rounded text-sm">
            <strong>Selected:</strong>
            {{ dateRange?.[0] || 'Not set' }} to {{ dateRange?.[1] || 'Not set' }}
          </div>
          <div class="mt-2 text-xs text-gray-500">Valid range: Last 180 days to today</div>
        </div>

        <!-- Single Month -->
        <div class="bg-white rounded-lg shadow-sm border border-gray-200 p-6">
          <h2 class="text-lg font-semibold text-gray-900 mb-4">Single Month</h2>
          <DatePicker v-model="singleMonth" :month-picker="true" placeholder="Select a month" />
          <div v-if="singleMonth" class="mt-4 p-3 bg-gray-50 rounded text-sm">
            <strong>Selected:</strong> {{ singleMonth }}
          </div>
        </div>

        <!-- Month Range -->
        <div class="bg-white rounded-lg shadow-sm border border-gray-200 p-6">
          <h2 class="text-lg font-semibold text-gray-900 mb-4">Month Range</h2>
          <DatePicker
            v-model="monthRange"
            :range="true"
            :month-picker="true"
            placeholder="Select month range"
          />
          <div
            v-if="monthRange?.[0] || monthRange?.[1]"
            class="mt-4 p-3 bg-gray-50 rounded text-sm"
          >
            <strong>Selected:</strong>
            {{ monthRange?.[0] || 'Not set' }} to {{ monthRange?.[1] || 'Not set' }}
          </div>
        </div>
      </div>

      <!-- Features -->
      <div class="mt-12 bg-white rounded-lg shadow-sm border border-gray-200 p-6">
        <h2 class="text-xl font-semibold text-gray-900 mb-6">Features</h2>
        <div class="grid gap-4 sm:grid-cols-2">
          <div class="flex items-center">
            <CheckIcon class="w-5 h-5 text-green-500 mr-3 flex-shrink-0" />
            <span class="text-gray-700">Date validation (180 days range)</span>
          </div>
          <div class="flex items-center">
            <CheckIcon class="w-5 h-5 text-green-500 mr-3 flex-shrink-0" />
            <span class="text-gray-700">Custom primary color (#E54993)</span>
          </div>
          <div class="flex items-center">
            <CheckIcon class="w-5 h-5 text-green-500 mr-3 flex-shrink-0" />
            <span class="text-gray-700">Single date selection</span>
          </div>
          <div class="flex items-center">
            <CheckIcon class="w-5 h-5 text-green-500 mr-3 flex-shrink-0" />
            <span class="text-gray-700">Date range selection</span>
          </div>
          <div class="flex items-center">
            <CheckIcon class="w-5 h-5 text-green-500 mr-3 flex-shrink-0" />
            <span class="text-gray-700">Month selection</span>
          </div>
          <div class="flex items-center">
            <CheckIcon class="w-5 h-5 text-green-500 mr-3 flex-shrink-0" />
            <span class="text-gray-700">Month range selection</span>
          </div>
          <div class="flex items-center">
            <CheckIcon class="w-5 h-5 text-green-500 mr-3 flex-shrink-0" />
            <span class="text-gray-700">Mobile responsive</span>
          </div>
          <div class="flex items-center">
            <CheckIcon class="w-5 h-5 text-green-500 mr-3 flex-shrink-0" />
            <span class="text-gray-700">Accessible design</span>
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

// Icon Component
const CheckIcon = {
  template: `
    <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
      <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd"></path>
    </svg>
  `,
}
</script>
