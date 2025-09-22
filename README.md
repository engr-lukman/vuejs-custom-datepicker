# Vue 3 Custom DatePicker

A clean, reusable, and highly customizable date picker component built with Vue 3, TypeScript, and Tailwind CSS v4.

## Features

- ✅ **Single Date Selection** - Pick individual dates with Today button
- ✅ **Date Range Selection** - Select start and end dates
- ✅ **Month Selection** - Month-only picker mode
- ✅ **Month Range Selection** - Select month ranges
- ✅ **Date Format** - Display dates in dd/mm/yyyy format
- ✅ **Date Validation** - Configurable min/max date ranges (default: last 180 days to today)
- ✅ **Accessibility** - Full keyboard navigation and ARIA support
- ✅ **Mobile Responsive** - Touch-friendly interface with pointer cursors
- ✅ **TypeScript** - Full type safety
- ✅ **Custom Styling** - Different colors for different date states## User Interface Improvements

### Date Display Format

- **Input Display**: Shows dates in `dd/mm/yyyy` format (e.g., 22/09/2025)
- **Internal Storage**: Maintains ISO format (`yyyy-mm-dd`) for data consistency
- **Range Display**: `22/09/2025 - 30/09/2025` format for date ranges

### Smart Footer

- **Today Button**: Only visible in single date picker mode (not in range or month pickers)
- **Clear Button**: Always available for resetting selections

### Enhanced Interactions

- **Pointer Cursors**: All selectable dates and months show pointer cursor
- **Disabled States**: Non-selectable dates show not-allowed cursor
- **Touch Friendly**: Optimized for mobile interactions

## Color Scheme

The component uses a sophisticated color system:

- **Today's Date**: Yellow border (`border-today-500`) with light yellow background
- **Range Start Date**: Primary color background (`bg-primary-600`)
- **Range End Date**: Darker primary color background (`bg-primary-700`)
- **Dates in Range**: Light blue background (`bg-range-100`) with blue text
- **Selected Single Date**: Primary color background (`bg-primary-600`)

## Installation

```bash
# Install dependencies
pnpm install

# Start development server
pnpm dev

# Build for production
pnpm build

# Type checking
pnpm type-check

# Linting
pnpm lint

# Format code
pnpm format:all
```

## Usage

### Basic Single Date Selection

```vue
<template>
  <DatePicker v-model="selectedDate" placeholder="Select a date" />
</template>

<script setup>
import { ref } from 'vue'
import DatePicker from '@/components/DatePicker.vue'

const selectedDate = ref(null)
</script>
```

### Date Range Selection

```vue
<template>
  <DatePicker v-model="dateRange" :range="true" placeholder="Select date range" />
</template>

<script setup>
import { ref } from 'vue'
import DatePicker from '@/components/DatePicker.vue'

const dateRange = ref([null, null])
</script>
```

### Month Selection

```vue
<template>
  <DatePicker v-model="selectedMonth" :month-picker="true" placeholder="Select a month" />
</template>

<script setup>
import { ref } from 'vue'
import DatePicker from '@/components/DatePicker.vue'

const selectedMonth = ref(null)
</script>
```

## Props

| Prop                   | Type                                   | Default         | Description                             |
| ---------------------- | -------------------------------------- | --------------- | --------------------------------------- |
| `modelValue`           | `string \| (string \| null)[] \| null` | `null`          | The selected date(s)                    |
| `placeholder`          | `string`                               | `'Select date'` | Input placeholder text                  |
| `range`                | `boolean`                              | `false`         | Enable range selection                  |
| `monthPicker`          | `boolean`                              | `false`         | Enable month-only mode                  |
| `minDate`              | `string \| null`                       | `null`          | Minimum selectable date                 |
| `maxDate`              | `string \| null`                       | `null`          | Maximum selectable date                 |
| `enableDateValidation` | `boolean`                              | `true`          | Enable date validation (180 days range) |

## Events

| Event               | Type                                   | Description                         |
| ------------------- | -------------------------------------- | ----------------------------------- |
| `update:modelValue` | `string \| (string \| null)[] \| null` | Emitted when date selection changes |

## Styling

The component uses Tailwind CSS v4 with custom color themes:

- **Primary Colors**: Pink theme (`#E54993`)
- **Range Colors**: Blue theme for date ranges
- **Today Colors**: Yellow theme for current date indication

## Architecture

### Type Safety

- Full TypeScript implementation
- Strict type checking for all props and events
- Type-safe date operations

### Error Handling

- Graceful handling of invalid dates
- Safe date parsing with try-catch blocks
- Validation for disabled dates

### Performance

- Computed properties for efficient reactivity
- Optimized calendar grid generation
- Minimal re-renders

### Accessibility

- ARIA labels and roles
- Keyboard navigation support
- Screen reader friendly

## Development

### Code Quality

- ESLint configuration with Vue 3 rules
- Prettier formatting with Tailwind CSS class sorting
- TypeScript strict mode
- Comprehensive error handling

### Architecture Highlights

- Clean separation of concerns
- Reusable helper functions
- Optimized computed properties
- Proper event handling
- Memory leak prevention (event cleanup)

## Browser Support

- Modern browsers with ES2020+ support
- Mobile Safari and Chrome
- Desktop Firefox, Chrome, Safari, Edge

## License

MIT License
