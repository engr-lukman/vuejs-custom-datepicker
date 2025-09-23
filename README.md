# Vue.js Custom DatePicker

A flexible and modern Vue.js date picker component built with TypeScript and Tailwind CSS. Supports both single and range selection for dates and months with built-in validation and accessibility features.

## Features

- **Multiple Selection Modes**: Single date/month or range selection
- **Two View Types**: Calendar view for dates or grid view for months
- **Quick Selection**: Preset options like "Today", "Last 30 days", "This month"

## Quick Start

```bash
# Install dependencies
pnpm install

# Start development server
pnpm run dev

# Build for production
pnpm run build
```

## Usage

### Basic Examples

**Single Date Selection:**

```vue
<template>
  <DatePicker v-model="selectedDate" mode="single" placeholder="Choose a date" />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const selectedDate = ref<string | null>(null)
// Result: "2024-01-15"
</script>
```

**Date Range Selection:**

```vue
<template>
  <DatePicker v-model="dateRange" />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const dateRange = ref<string[] | null>(null)
// Result: ["2024-01-15", "2024-01-20"]
</script>
```

**Month Selection:**

```vue
<template>
  <DatePicker v-model="selectedMonth" view="month" mode="single" />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const selectedMonth = ref<string | null>(null)
// Result: "2024-01"
</script>
```

**With Date Constraints:**

```vue
<template>
  <DatePicker
    v-model="constrainedDate"
    mode="single"
    :min-date="'2024-01-01'"
    :max-date="'2024-12-31'"
  />
</template>
```

## API Reference

### Props

| Property      | Type                         | Default        | Description                  |
| ------------- | ---------------------------- | -------------- | ---------------------------- |
| `modelValue`  | `string \| string[] \| null` | `null`         | Selected date(s) or month(s) |
| `mode`        | `'single' \| 'range'`        | `'range'`      | Selection mode               |
| `view`        | `'date' \| 'month'`          | `'date'`       | View type                    |
| `placeholder` | `string`                     | Auto-generated | Input placeholder            |
| `minDate`     | `string \| null`             | `null`         | Minimum selectable date      |
| `maxDate`     | `string \| null`             | `null`         | Maximum selectable date      |

### Events

| Event                  | Payload                      | Description           |
| ---------------------- | ---------------------------- | --------------------- |
| `update:modelValue`    | `string \| string[] \| null` | Selection changed     |
| `date-selected`        | `Date`                       | Single date selected  |
| `date-range-selected`  | `{start: Date, end: Date}`   | Date range selected   |
| `month-selected`       | `Date`                       | Single month selected |
| `month-range-selected` | `{start: Date, end: Date}`   | Month range selected  |

### Data Formats

| View + Mode    | Return Format                  | Example                        |
| -------------- | ------------------------------ | ------------------------------ |
| Date + Single  | `"YYYY-MM-DD"`                 | `"2024-01-15"`                 |
| Date + Range   | `["YYYY-MM-DD", "YYYY-MM-DD"]` | `["2024-01-15", "2024-01-20"]` |
| Month + Single | `"YYYY-MM"`                    | `"2024-01"`                    |
| Month + Range  | `["YYYY-MM", "YYYY-MM"]`       | `["2024-01", "2024-03"]`       |

## Customization

### Styling

The component uses Tailwind CSS with a custom pink/magenta color scheme. Colors can be customized in `src/css/main.css`:

```css
@theme {
  --color-primary-500: #e54993; /* Base color */
  --color-primary-700: #be185d; /* Selected states */
  --color-primary-300: #f9a8d4; /* Hover states */
  /* ... other variants */
}
```

### Quick Selection Options

Available in date range mode only:

- **Today**: Current date
- **Yesterday**: Previous day
- **Last 30 days**: 30-day period ending today
- **This month**: From month start to today
- **6 months**: 180-day period ending today

## Project Structure

```
src/
├── components/
│   └── DatePicker.vue     # Main component (928 lines)
├── css/
│   └── main.css          # Tailwind configuration
├── App.vue               # Demo application
└── main.ts              # Entry point
```

## Common Issues

**Date Format Issues:**

- Ensure dates follow `YYYY-MM-DD` format for date view
- Use `YYYY-MM` format for month view constraints
- Check timezone handling for consistent behavior

## License

MIT License - free for personal and commercial use.

---

_Built with Vue 3, TypeScript, and Tailwind CSS_
