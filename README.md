# Vue.js Custom Date & Month Picker

A modern, flexible Vue 3 component library for date and month selection, built with **TypeScript** and **Tailwind CSS v4**. Supports both single and range selection, navigation constraints, and accessibility.

## Features

- **DatePicker.vue**: Calendar-based date selection (single or range)
- **MonthPicker.vue**: Grid-based month selection (single or range)
- **Navigation Constraints**: Set min/max boundaries for selectable dates or months
- **Quick Selection**: Presets like "Today", "Yesterday", "Last 30 days", etc. (DatePicker only)

---

## Installation

```bash
pnpm install
pnpm run dev    # Start development server
pnpm run build  # Build for production
```

---

## Usage

### DatePicker

```vue
<!-- Filename: src/components/DatePicker.vue -->
<template>
  <DatePicker
    v-model="selectedDate"
    mode="single"
    placeholder="Select a date"
    :min-navigation="'2025-08-01'"
    :max-navigation="'2025-10-31'"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const selectedDate = ref<string | string[] | null>(null)
</script>
```

- **Single mode:** returns `"YYYY-MM-DD"`
- **Range mode:** returns `["YYYY-MM-DD", "YYYY-MM-DD"]`

### MonthPicker

```vue
<!-- Filename: src/components/MonthPicker.vue -->
<template>
  <MonthPicker
    v-model="selectedMonthRange"
    mode="range"
    :min-navigation="'2025-01'"
    :max-navigation="'2025-12'"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import MonthPicker from './components/MonthPicker.vue'

const selectedMonthRange = ref<string[] | string | null>(null)
</script>
```

- **Single mode:** returns `"YYYY-MM"`
- **Range mode:** returns `["YYYY-MM", "YYYY-MM"]`

---

## Props

### DatePicker

| Prop          | Type                       | Default  | Description           |
| ------------- | -------------------------- | -------- | --------------------- |
| modelValue    | string \| string[] \| null | null     | Selected date(s)      |
| mode          | 'single' \| 'range'        | 'range'  | Selection mode        |
| placeholder   | string                     | Auto-gen | Input placeholder     |
| minNavigation | string \| null             | null     | Earliest date allowed |
| maxNavigation | string \| null             | null     | Latest date allowed   |

### MonthPicker

| Prop          | Type                       | Default  | Description         |
| ------------- | -------------------------- | -------- | ------------------- |
| modelValue    | string \| string[] \| null | null     | Selected month(s)   |
| mode          | 'single' \| 'range'        | 'single' | Selection mode      |
| placeholder   | string                     | Auto-gen | Input placeholder   |
| minNavigation | string \| null             | null     | Earliest year/month |
| maxNavigation | string \| null             | null     | Latest year/month   |

---

## Events

| Component   | Event             | Payload                    | Description       |
| ----------- | ----------------- | -------------------------- | ----------------- |
| DatePicker  | update:modelValue | string \| string[] \| null | Selection changed |
| MonthPicker | update:modelValue | string \| string[] \| null | Selection changed |

---

## Data Formats

| Component   | Mode   | Return Format                | Example                      |
| ----------- | ------ | ---------------------------- | ---------------------------- |
| DatePicker  | Single | "YYYY-MM-DD"                 | "2025-09-24"                 |
| DatePicker  | Range  | ["YYYY-MM-DD", "YYYY-MM-DD"] | ["2025-09-20", "2025-09-24"] |
| MonthPicker | Single | "YYYY-MM"                    | "2025-09"                    |
| MonthPicker | Range  | ["YYYY-MM", "YYYY-MM"]       | ["2025-06", "2025-09"]       |

---

## Styling

Customize via `src/css/main.css` using Tailwind CSS v4:

```css
@theme {
  --color-primary-50: #fdf2f8;
  --color-primary-100: #fce7f3;
  --color-primary-200: #fbcfe8;
  --color-primary-300: #f9a8d4;
  --color-primary-400: #f472b6;
  --color-primary-500: #e54993; /* Base */
  --color-primary-600: #db2777;
  --color-primary-700: #be185d; /* Selected */
  --color-primary-800: #9d174d;
  --color-primary-900: #831843;
}
```

---

## Quick Selection (DatePicker)

- **Today**: Current date
- **Yesterday**: Previous day
- **Last 30 days**: 30-day period ending today
- **This month**: Month start to today
- **Last 6 months**: 180-day period ending today

---

## Project Structure

```
src/
├── components/
│   ├── DatePicker.vue
│   └── MonthPicker.vue
├── css/
│   └── main.css
├── App.vue
└── main.ts
```

---

## Browser Support

- Modern evergreen browsers (Chrome, Firefox, Safari, Edge)
- **No IE11 support**

---

## License

MIT License

---

> Built with Vue 3.5, TypeScript 5.8, and Tailwind CSS 4.1
