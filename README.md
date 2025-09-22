# Vue 3 Custom DateP## Features

### DatePicker Component

- ✅ **Single Date Selection** - Pick individual dates with Today button
- ✅ **Date Range Selection** - Select start and end dates
- ✅ **Date Format** - Display dates in dd/mm/yyyy format
- ✅ **Date Validation** - Configurable min/max date ranges (default: last 180 days to today)
- ✅ **Today Highlighting** - Current date highlighted with yellow border

### MonthPicker Component

- ✅ **Single Month Selection** - Pick individual months
- ✅ **Month Range Selection** - Select start and end months
- ✅ **Current Month Highlighting** - Current month highlighted like today's date
- ✅ **Month Format** - Display months in Mon YYYY format (e.g., Sep 2025)

### Shared Features

- ✅ **Accessibility** - Full keyboard navigation and ARIA support
- ✅ **Mobile Responsive** - Touch-friendly interface with pointer cursors
- ✅ **TypeScript** - Full type safety with shared composables
- ✅ **Custom Styling** - Different colors for different selection statesnthPicker

A clean, reusable, and highly customizable date and month picker component library built with Vue 3, TypeScript, and Tailwind CSS v4.

## Components

### 📅 **DatePicker**

- Single date selection with Today button
- Date range selection
- dd/mm/yyyy display format
- Date validation (configurable range)

### 📆 **MonthPicker**

- Single month selection
- Month range selection
- Current month highlighting (like today highlighting)
- Month/Year format display

## Features

## Architecture

The component library is built with a clean, modular architecture:

### Components

- **DatePicker.vue** - Handles single date and date range selection
- **MonthPicker.vue** - Handles single month and month range selection

### Shared Composables

- **picker-types.ts** - TypeScript interfaces and constants
- **picker-utils.ts** - Shared utility functions for date operations

### Features by Component

#### DatePicker

- Single date selection with Today button
- Date range selection
- dd/mm/yyyy display format
- Date validation (configurable min/max ranges)

#### MonthPicker

- Single month selection
- Month range selection
- Current month highlighting
- Mon YYYY display format

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

### DatePicker Component

```vue
<template>
  <div>
    <!-- Single Date Picker -->
    <DatePicker mode="single" @date-selected="onDateSelected" />

    <!-- Date Range Picker -->
    <DatePicker mode="range" @range-selected="onRangeSelected" />
  </div>
</template>

<script setup lang="ts">
import DatePicker from '@/components/DatePicker.vue'

const onDateSelected = (date: Date) => {
  console.log('Selected date:', date)
}

const onRangeSelected = (range: { start: Date; end: Date }) => {
  console.log('Selected range:', range)
}
</script>
```

### MonthPicker Component

```vue
<template>
  <div>
    <!-- Single Month Picker -->
    <MonthPicker mode="single" @month-selected="onMonthSelected" />

    <!-- Month Range Picker -->
    <MonthPicker mode="range" @month-range-selected="onMonthRangeSelected" />
  </div>
</template>

<script setup lang="ts">
import MonthPicker from '@/components/MonthPicker.vue'

const onMonthSelected = (date: Date) => {
  console.log('Selected month:', date)
}

const onMonthRangeSelected = (range: { start: Date; end: Date }) => {
  console.log('Selected month range:', range)
}
</script>
```

## Color Scheme

The component uses a sophisticated color system:

- **Primary Color**: Custom pink (#E54993) for selections and highlights
- **Today's Date**: Yellow border (`border-today-500`) with light yellow background
- **Current Month**: Yellow border highlighting in MonthPicker
- **Range Start/End**: Primary color background with different shades
- **Dates in Range**: Light blue background (`bg-range-100`) with blue text
- **Disabled States**: Muted colors with not-allowed cursor

## Props & Events

### DatePicker Props

- `mode: 'single' | 'range'` - Selection mode
- `minDate?: Date` - Minimum selectable date
- `maxDate?: Date` - Maximum selectable date
- `placeholder?: string` - Input placeholder text

### DatePicker Events

- `@date-selected(date: Date)` - Emitted when single date is selected
- `@range-selected(range: {start: Date, end: Date})` - Emitted when date range is selected

### MonthPicker Props

- `mode: 'single' | 'range'` - Selection mode
- `placeholder?: string` - Input placeholder text

### MonthPicker Events

- `@month-selected(date: Date)` - Emitted when single month is selected
- `@month-range-selected(range: {start: Date, end: Date})` - Emitted when month range is selected

| Prop          | Type                                   | Default         | Description             |
| ------------- | -------------------------------------- | --------------- | ----------------------- |
| `modelValue`  | `string \| (string \| null)[] \| null` | `null`          | The selected date(s)    |
| `placeholder` | `string`                               | `'Select date'` | Input placeholder text  |
| `range`       | `boolean`                              | `false`         | Enable range selection  |
| `monthPicker` | `boolean`                              | `false`         | Enable month-only mode  |
| `minDate`     | `string \| null`                       | `null`          | Minimum selectable date |
| `maxDate`     | `string \| null`                       | `null`          | Maximum selectable date |

## Technical Details

### Type Safety

- Full TypeScript implementation with shared interfaces
- Strict type checking for all props and events
- Type-safe date operations across components

### Architecture Highlights

- **Separation of Concerns**: DatePicker and MonthPicker as specialized components
- **Shared Composables**: Common types and utilities in `/composables`
- **Clean Code**: Optimized computed properties and minimal re-renders
- **Error Handling**: Graceful handling of invalid dates with try-catch blocks

### Performance

- Computed properties for efficient reactivity
- Optimized calendar grid generation
- Memory leak prevention with proper event cleanup

### Accessibility

- ARIA labels and roles for screen readers
- Full keyboard navigation support
- Touch-friendly mobile interface

## Development

### Code Quality Tools

- ESLint configuration with Vue 3 + TypeScript rules
- Prettier formatting with Tailwind CSS class sorting
- TypeScript strict mode enabled
- Comprehensive error handling throughout

### Browser Support

- Modern browsers with ES2020+ support
- Mobile Safari and Chrome optimized
- Desktop Firefox, Chrome, Safari, Edge

## License

MIT License
