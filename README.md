# Vue.js Custom DatePicker

A comprehensive Vue.js date picker component with advanced features, flexible selection modes, strict validation, and modern Tailwind CSS styling.

## Features

### Core Functionality

- **Flexible Selection Modes**: Single or range selection for both dates and months
- **Two View Types**: Date view (YYYY-MM-DD) and Month view (YYYY-MM)
- **Mode Support**:
  - `mode="range"` (default): Select date/month ranges
  - `mode="single"`: Select individual dates/months
- **Quick Selection Sidebar**: Preset date ranges (Today, Yesterday, Last 30/180 days) - available only in range mode
- **Auto-close Behavior**: Automatically closes after selection completion

### Data Format

- **Date View + Single Mode**: Returns `YYYY-MM-DD` string
- **Date View + Range Mode**: Returns `[YYYY-MM-DD, YYYY-MM-DD]` array
- **Month View + Single Mode**: Returns `YYYY-MM` string
- **Month View + Range Mode**: Returns `[YYYY-MM, YYYY-MM]` array

### Validation & Navigation

- **Configurable Date Limits**: Set min/max dates with `minDate` and `maxDate` props
- **Smart Month Validation**: Intelligent month range validation based on configured limits
- **Intelligent Navigation**: Prevents navigation to invalid date ranges with visual feedback
- **Real-time Validation**: Instant feedback on selectable vs non-selectable dates

### UI/UX Excellence

- **Tailwind CSS v4**: Modern utility-first styling with custom primary color (#E54993)
- **Mobile Responsive**: Fully optimized for all screen sizes and devices
- **Enhanced Spacing**: Improved calendar layout with standard spacing (gap-2)
- **Visual States**: Clear hover, focus, and selection states
- **Accessibility**: Proper ARIA labels and keyboard navigation support

### Technical Features

- **Vue 3 Composition API**: Modern reactive architecture with TypeScript
- **Optional Chaining**: Robust error handling throughout the component
- **Self-contained**: Embedded types and utilities - no external dependencies
- **Clean Code**: Production-ready with human-readable comments
- **Timezone Safe**: Local date formatting to prevent timezone-related display issues

## Quick Start

```bash
# Install dependencies
pnpm install

# Start development server
pnpm run dev

# Build for production
pnpm run build

# Run linting
pnpm run lint
```

## Usage Examples

### Single Date Selection

```vue
<template>
  <DatePicker
    v-model="selectedDate"
    mode="single"
    placeholder="Select a date"
    @date-selected="handleDateSelection"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const selectedDate = ref<string | null>(null)

const handleDateSelection = (date: Date) => {
  console.log('Selected date:', date)
  // selectedDate.value will be in format: "2024-01-15"
}
</script>
```

### Date Range Selection (Default Mode)

```vue
<template>
  <DatePicker
    v-model="dateRange"
    placeholder="Select date range"
    @range-selected="handleRangeSelection"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const dateRange = ref<[string, string] | null>(null)

const handleRangeSelection = (range: { start: Date; end: Date }) => {
  console.log('Selected range:', range)
  // dateRange.value will be in format: ["2024-01-15", "2024-01-20"]
}
</script>
```

### Single Month Selection

```vue
<template>
  <DatePicker
    v-model="selectedMonth"
    view="month"
    mode="single"
    placeholder="Select month"
    @month-selected="handleMonthSelection"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const selectedMonth = ref<string | null>(null)

const handleMonthSelection = (date: Date) => {
  console.log('Selected month:', date)
  // selectedMonth.value will be in format: "2024-01"
}
</script>
```

### Month Range Selection

```vue
<template>
  <DatePicker
    v-model="monthRange"
    view="month"
    placeholder="Select month range"
    @month-range-selected="handleMonthRangeSelection"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const monthRange = ref<[string, string] | null>(null)

const handleMonthRangeSelection = (range: { start: Date; end: Date }) => {
  console.log('Selected month range:', range)
  // monthRange.value will be in format: ["2024-01", "2024-03"]
}
</script>
```

## Component API

### Props

| Prop          | Type                                 | Default   | Description                             |
| ------------- | ------------------------------------ | --------- | --------------------------------------- |
| `modelValue`  | `string \| [string, string] \| null` | `null`    | The selected date(s) or month(s)        |
| `mode`        | `'single' \| 'range'`                | `'range'` | Selection mode (single value or range)  |
| `view`        | `'date' \| 'month'`                  | `'date'`  | View type (date picker or month picker) |
| `placeholder` | `string`                             | Auto      | Input placeholder text (auto-generated) |
| `minDate`     | `string \| null`                     | `null`    | Minimum selectable date/month           |
| `maxDate`     | `string \| null`                     | `null`    | Maximum selectable date/month           |

#### Model Value Format

The format of `modelValue` depends on the `view` and `mode` combination:

- **Date + Single**: `"2024-01-15"` (YYYY-MM-DD)
- **Date + Range**: `["2024-01-15", "2024-01-20"]` (YYYY-MM-DD array)
- **Month + Single**: `"2024-01"` (YYYY-MM)
- **Month + Range**: `["2024-01", "2024-03"]` (YYYY-MM array)

### Events

| Event                  | Payload                              | Description                       |
| ---------------------- | ------------------------------------ | --------------------------------- |
| `update:modelValue`    | `string \| [string, string] \| null` | Emitted when selection changes    |
| `date-selected`        | `Date`                               | Emitted when single date selected |
| `range-selected`       | `{ start: Date; end: Date }`         | Emitted when date range selected  |
| `month-selected`       | `Date`                               | Emitted when month selected       |
| `month-range-selected` | `{ start: Date; end: Date }`         | Emitted when month range selected |

### Types

```typescript
interface DatePickerProps {
  modelValue?: (string | null)[] | string | null
  placeholder?: string
  view?: 'date' | 'month'
  mode?: 'range' | 'single'
  minDate?: string | null
  maxDate?: string | null
}

interface DateRange {
  start: Date
  end: Date
}

interface QuickSelectionOption {
  key: string
  label: string
  getValue: () => string[]
}
```

## Styling & Theming

The component uses a custom Tailwind CSS configuration with:

- **Primary Color**: `#E54993` (vibrant pink/magenta)
- **Enhanced Spacing**: Standard gap-2 spacing for better visual clarity
- **Responsive Design**: Mobile-first approach with optimized touch targets
- **Interactive States**: Smooth hover, focus, and active state transitions
- **Gradient Backgrounds**: Subtle visual enhancements throughout

### CSS Structure

```css
/* Main theme colors in src/css/main.css */
:root {
  --primary: #e54993;
  --primary-hover: #d63384;
  --primary-light: #f8d7da;
}
```

## Validation Rules

### Date Mode (Single & Range)

- **180-Day Limit**: Current date + previous 180 days (excluding current date)
- **No Future Dates**: Cannot select dates beyond today
- **Smart Range Validation**: End date must be after or equal to start date
- **Timezone Safe**: Local date formatting prevents timezone-related issues

### Month Mode (Single & Range)

- **6-Month Window**: Current month + previous 5 months only
- **No Future Months**: Cannot select future months beyond current
- **Year Navigation**: Intelligent year navigation based on valid month ranges

## Quick Selection Options

The quick selection sidebar (available in date range mode) includes:

- **Today**: Current date only
- **Yesterday**: Previous day
- **Last 7 days**: Past week including today
- **Last 30 days**: Past month including today
- **Last 90 days**: Past quarter including today
- **This Month**: Current month's full date range

> **Note**: "Last month" option has been removed for cleaner UX

## Project Structure

```
vuejs-custom-datepicker/
├── src/
│   ├── components/
│   │   └── DatePicker.vue      # Main component (848 lines)
│   ├── css/
│   │   └── main.css           # Tailwind configuration
│   ├── App.vue                # Demo application with examples
│   └── main.ts               # Entry point
├── public/
│   └── favicon.ico
├── package.json              # pnpm dependencies
├── vite.config.ts           # Vite configuration
├── tsconfig.json            # TypeScript configuration
└── README.md               # This documentation
```

## Development Scripts

```bash
# Development
pnpm run dev          # Start dev server with hot reload
pnpm run build        # Build for production
pnpm run preview      # Preview production build

# Code Quality
pnpm run lint         # Run ESLint
pnpm run lint:fix     # Fix ESLint issues
pnpm run type-check   # TypeScript type checking

# Dependencies
pnpm install          # Install dependencies
pnpm update          # Update dependencies
```

## Testing & Quality Assurance

- **Zero Lint Errors**: Clean codebase with consistent formatting using ESLint + Prettier
- **TypeScript Strict Mode**: Full type safety with comprehensive error handling
- **Optional Chaining**: Robust error prevention throughout the component
- **Production Ready**: Optimized build with tree-shaking and code splitting
- **Performance**: Efficient reactivity using Vue 3 Composition API
- **Accessibility**: ARIA labels, keyboard navigation, and screen reader support

## Use Cases

Perfect for applications requiring strict date validation:

- **Financial Applications**: Transaction date selection with 180-day historical limits
- **Reporting Systems**: Date range selection for business reports and analytics
- **Booking Systems**: Event/appointment date selection with validation rules
- **Analytics Dashboards**: Time period selection for data visualization
- **Data Entry Forms**: Date input with comprehensive validation and error handling
- **Audit Systems**: Historical date selection with controlled time ranges

## Known Issues & Solutions

- **Timezone Display Mismatch**: **Fixed** - Now uses local date formatting instead of `toISOString()`
- **180-Day Calculation**: **Fixed** - Correctly excludes current date from range calculation
- **Mobile Touch Targets**: **Optimized** - Enhanced spacing (gap-2) for better mobile interaction

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes with proper TypeScript types
4. Ensure zero lint errors (`pnpm run lint`)
5. Test across all component modes
6. Submit a pull request with clear description

## License

MIT License - feel free to use in personal and commercial projects

## Related Resources

- [Vue 3 Documentation](https://vuejs.org/) - Official Vue.js guide
- [Tailwind CSS v4](https://tailwindcss.com/) - Utility-first CSS framework
- [TypeScript](https://www.typescriptlang.org/) - Typed JavaScript at scale
- [Vite](https://vitejs.dev/) - Next generation frontend tooling
- [TypeScript](https://www.typescriptlang.org/)
- [Vite](https://vitejs.dev/)
