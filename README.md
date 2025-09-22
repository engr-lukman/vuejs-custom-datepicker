# Vue.js Custom DatePicker

A comprehensive Vue.js date picker component with advanced features, strict validation, and Tailwind CSS styling.

## Features

### Core Functionality

- **Single Date Selection**: Pick individual dates with 180-day validation
- **Date Range Selection**: Select date ranges with smart validation
- **Month Selection**: Single month and month range selection
- **Quick Selection Sidebar**: Preset date ranges (Today, Yesterday, Last 7/30/90 days)
- **Auto-close Behavior**: Automatically closes after quick selection

### Validation & Navigation

- **180-Day Range Limit**: Restricts selections to current date + previous 179 days
- **Smart Month Validation**: Allows current month + previous 5 months only
- **Strict Navigation Controls**: Prevents navigation to invalid date ranges
- **Real-time Validation**: Instant feedback on invalid selections

### UI/UX Excellence

- **Tailwind CSS v4**: Modern utility-first styling with custom primary color (#E54993)
- **Mobile Responsive**: Optimized for all screen sizes
- **Gradient Backgrounds**: Beautiful visual enhancements
- **Hover & Focus States**: Interactive feedback throughout
- **Accessibility**: Proper ARIA labels and keyboard navigation

### Technical Features

- **TypeScript Support**: Full type safety and IntelliSense
- **Vue 3 Composition API**: Modern reactive architecture
- **Embedded Types & Utilities**: Self-contained component with no external dependencies
- **Clean Architecture**: Production-ready code structure

## Quick Start

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Run linting
npm run lint
```

## Usage Examples

### Basic Single Date Selection

```vue
<template>
  <DatePicker v-model="selectedDate" mode="single" placeholder="Select a date" />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const selectedDate = ref<Date | null>(null)
</script>
```

### Date Range with Quick Selection

```vue
<template>
  <DatePicker v-model="dateRange" mode="range" placeholder="Select date range" />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const dateRange = ref<{ start: Date | null; end: Date | null }>({
  start: null,
  end: null,
})
</script>
```

### Month Selection

```vue
<template>
  <DatePicker v-model="selectedMonth" mode="month" placeholder="Select month" />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const selectedMonth = ref<Date | null>(null)
</script>
```

### Month Range Selection

```vue
<template>
  <DatePicker v-model="monthRange" mode="month-range" placeholder="Select month range" />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const monthRange = ref<{ start: Date | null; end: Date | null }>({
  start: null,
  end: null,
})
</script>
```

## Component API

### Props

| Prop          | Type                                              | Default         | Description            |
| ------------- | ------------------------------------------------- | --------------- | ---------------------- |
| `modelValue`  | `Date \| DateRange \| null`                       | `null`          | The selected date(s)   |
| `mode`        | `'single' \| 'range' \| 'month' \| 'month-range'` | `'single'`      | Selection mode         |
| `placeholder` | `string`                                          | `'Select date'` | Input placeholder text |
| `disabled`    | `boolean`                                         | `false`         | Disable the component  |

### Events

| Event               | Payload                     | Description                    |
| ------------------- | --------------------------- | ------------------------------ |
| `update:modelValue` | `Date \| DateRange \| null` | Emitted when selection changes |

### Types

```typescript
interface DateRange {
  start: Date | null
  end: Date | null
}

type DatePickerMode = 'single' | 'range' | 'month' | 'month-range'
```

## Styling & Theming

The component uses a custom Tailwind CSS configuration with:

- **Primary Color**: `#E54993` (vibrant pink)
- **Responsive Design**: Mobile-first approach
- **Custom Components**: Button variants, form controls, and interactive states
- **Gradient Backgrounds**: Subtle visual enhancements

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

- **180-Day Limit**: Current date + previous 179 days
- **No Future Dates**: Cannot select dates beyond today
- **Smart Range Validation**: End date must be after start date
- **Invalid Date Handling**: Graceful error handling for invalid inputs

### Month Mode (Single & Range)

- **6-Month Window**: Current month + previous 5 months
- **No Future Months**: Cannot select future months
- **Year Navigation**: Intelligent year navigation based on valid months

## Quick Selection Options

The quick selection sidebar (available in date range mode) includes:

- **Today**: Current date range
- **Yesterday**: Previous day
- **Last 7 days**: Past week
- **Last 30 days**: Past month
- **Last 90 days**: Past quarter
- **This Month**: Current month's date range
- **Last Month**: Previous month's date range

## Project Structure

```
vuejs-custom-datepicker/
├── src/
│   ├── components/
│   │   └── DatePicker.vue      # Main component (853 lines)
│   ├── css/
│   │   └── main.css           # Tailwind configuration
│   ├── App.vue                # Demo application
│   └── main.ts               # Entry point
├── public/
├── package.json
├── vite.config.ts
├── tsconfig.json
└── README.md
```

## Development Scripts

```bash
# Development
npm run dev          # Start dev server with hot reload
npm run build        # Build for production
npm run preview      # Preview production build

# Code Quality
npm run lint         # Run ESLint
npm run lint:fix     # Fix ESLint issues
npm run type-check   # TypeScript type checking

# Dependencies
npm install          # Install dependencies
npm update          # Update dependencies
```

## Testing & Quality

- **Zero Lint Errors**: Clean codebase with consistent formatting
- **TypeScript Strict Mode**: Full type safety
- **Production Ready**: Optimized build with tree-shaking
- **Performance**: Efficient reactivity with computed properties
- **Accessibility**: ARIA labels and keyboard support

## Use Cases

- **Financial Applications**: Transaction date selection with historical limits
- **Reporting Systems**: Date range selection for reports
- **Booking Systems**: Event date selection with validation
- **Analytics Dashboards**: Time period selection
- **Data Entry Forms**: Date input with strict validation

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Ensure all tests pass
5. Submit a pull request

## License

MIT License - see LICENSE file for details

## Related

- [Vue 3 Documentation](https://vuejs.org/)
- [Tailwind CSS v4](https://tailwindcss.com/)
- [TypeScript](https://www.typescriptlang.org/)
- [Vite](https://vitejs.dev/)
