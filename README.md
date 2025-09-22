# Vue.js Custom DatePicker

A flexible and comprehensive Vue.js date picker component built with TypeScript and Tailwind CSS. This component provides both single and range selection capabilities for dates and months, with intelligent validation and accessibility features.

## Overview

This date picker component offers a modern solution for date and month selection in Vue.js applications. It supports multiple selection modes, provides customizable date constraints, and includes preset quick selection options for common date ranges.

## Core Features

### Selection Modes and Views

The component operates in two distinct dimensions:

**View Types:**

- Date View: Displays a traditional calendar for day-level selection
- Month View: Shows a month grid for month-level selection

**Selection Modes:**

- Single Mode: Select one date or month
- Range Mode: Select a date or month range with start and end values

**Quick Selection Options:**
Available only in Date View with Range Mode, providing preset options:

- Today: Current date only
- Yesterday: Previous day
- Last 30 days: 30-day period ending today
- This month: From first day of current month to today
- 6 months: 180-day period ending today

### Data Formats

The component returns different data formats based on the view and mode combination:

- **Date View + Single Mode**: Returns string in YYYY-MM-DD format
- **Date View + Range Mode**: Returns array of two YYYY-MM-DD strings
- **Month View + Single Mode**: Returns string in YYYY-MM format
- **Month View + Range Mode**: Returns array of two YYYY-MM strings

### Validation and Constraints

**Date Constraints:**

- Configurable minimum and maximum date limits
- Automatic validation prevents selection of invalid dates
- Navigation controls are disabled when reaching date boundaries

**Smart Navigation:**

- Previous/Next buttons automatically disable when limits are reached
- Calendar navigation respects min/max date constraints
- Visual feedback for non-selectable dates

### User Interface

**Responsive Design:**

- Mobile-optimized touch targets and spacing
- Adaptive layout that works across all screen sizes
- Consistent spacing using Tailwind CSS gap utilities

**Visual States:**

- Clear visual distinction between selectable and disabled dates
- Highlighted current date indicator
- Distinct styling for selected dates and ranges
- Hover states for interactive elements

**Accessibility Features:**

- Full keyboard navigation support
- ARIA labels for screen readers
- Proper focus management
- Semantic HTML structure

### Technical Implementation

**Modern Vue.js Architecture:**

- Built with Vue 3 Composition API
- Full TypeScript support with strict typing
- Reactive state management with computed properties
- Efficient event handling and lifecycle management

**Error Handling:**

- Comprehensive error boundaries with try-catch blocks
- Optional chaining for safe property access
- Graceful degradation for invalid date inputs
- Robust validation throughout the component

**Performance Optimizations:**

- Optimized watchers with immediate flag
- Efficient DOM updates through virtual DOM
- Minimal re-renders using computed properties
- Clean event listener management

## Installation and Setup

### Prerequisites

This project requires Node.js and pnpm package manager.

### Quick Start

```bash
# Install dependencies
pnpm install

# Start development server
pnpm run dev

# Build for production
pnpm run build

# Run code linting
pnpm run lint
```

## Usage Examples

The component can be used in various configurations depending on your needs. Here are the most common usage patterns:

### Basic Single Date Selection

```vue
<template>
  <DatePicker
    v-model="selectedDate"
    mode="single"
    placeholder="Choose a date"
    aria-label="Select single date"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const selectedDate = ref<string | null>(null)

// selectedDate.value will contain: "2024-01-15"
</script>
```

### Date Range Selection with Quick Options

```vue
<template>
  <DatePicker v-model="dateRange" placeholder="Select date range" aria-label="Select date range" />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const dateRange = ref<[string, string] | null>(null)

// dateRange.value will contain: ["2024-01-15", "2024-01-20"]
</script>
```

### Single Month Selection

```vue
<template>
  <DatePicker
    v-model="selectedMonth"
    view="month"
    mode="single"
    placeholder="Choose a month"
    aria-label="Select month"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const selectedMonth = ref<string | null>(null)

// selectedMonth.value will contain: "2024-01"
</script>
```

### Month Range Selection

```vue
<template>
  <DatePicker
    v-model="monthRange"
    view="month"
    placeholder="Select month range"
    aria-label="Select month range"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const monthRange = ref<[string, string] | null>(null)

// monthRange.value will contain: ["2024-01", "2024-03"]
</script>
```

### Date Selection with Constraints

```vue
<template>
  <DatePicker
    v-model="constrainedDate"
    mode="single"
    :min-date="minDate"
    :max-date="maxDate"
    placeholder="Select date within range"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/DatePicker.vue'

const constrainedDate = ref<string | null>(null)
const minDate = ref('2024-01-01')
const maxDate = ref('2024-12-31')
</script>
```

## Component Properties and Events

### Properties (Props)

The DatePicker component accepts the following properties:

| Property      | Type                                     | Default Value  | Description                               |
| ------------- | ---------------------------------------- | -------------- | ----------------------------------------- |
| `modelValue`  | `string` or `[string, string]` or `null` | `null`         | The selected date(s) or month(s) value    |
| `mode`        | `'single'` or `'range'`                  | `'range'`      | Selection mode for single value or range  |
| `view`        | `'date'` or `'month'`                    | `'date'`       | View type for date picker or month picker |
| `placeholder` | `string`                                 | Auto-generated | Input placeholder text                    |
| `minDate`     | `string` or `null`                       | `null`         | Minimum selectable date or month          |
| `maxDate`     | `string` or `null`                       | `null`         | Maximum selectable date or month          |

#### Model Value Format Guidelines

The format of the `modelValue` property depends on the combination of `view` and `mode` settings:

**Date View Formats:**

- Single Mode: `"2024-01-15"` (YYYY-MM-DD format)
- Range Mode: `["2024-01-15", "2024-01-20"]` (Array of YYYY-MM-DD strings)

**Month View Formats:**

- Single Mode: `"2024-01"` (YYYY-MM format)
- Range Mode: `["2024-01", "2024-03"]` (Array of YYYY-MM strings)

#### Date Constraints Format

The `minDate` and `maxDate` properties accept different formats depending on the view:

**For Date View:**

- Format: `"YYYY-MM-DD"` (e.g., `"2024-01-15"`)
- Constrains selectable dates to the specified range

**For Month View:**

- Format: `"YYYY-MM"` (e.g., `"2024-01"`)
- Constrains selectable months to the specified range

### Events

The component emits the following events to communicate with parent components:

| Event Name             | Payload Type                             | Description                             |
| ---------------------- | ---------------------------------------- | --------------------------------------- |
| `update:modelValue`    | `string` or `[string, string]` or `null` | Emitted when the selection changes      |
| `date-selected`        | `Date`                                   | Emitted when a single date is selected  |
| `date-range-selected`  | `{ start: Date; end: Date }`             | Emitted when a date range is selected   |
| `month-selected`       | `Date`                                   | Emitted when a single month is selected |
| `month-range-selected` | `{ start: Date; end: Date }`             | Emitted when a month range is selected  |

### TypeScript Interface Definitions

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
  isVisible?: boolean
}

interface DayData {
  date: Date
  day: number
  month: number
  year: number
  isCurrentMonth: boolean
  isToday: boolean
  dateString: string
}

interface MonthData {
  month: number
  name: string
  shortName: string
  date: string
  isCurrentMonth: boolean
}
```

## Styling and Customization

### Design System

The component uses a modern design system built on Tailwind CSS v4 with a custom color palette:

**Primary Color Scheme:**

- Base Color: `#E54993` (vibrant pink/magenta)
- Color variants from 50 to 950 for different UI states
- Gradient backgrounds for enhanced visual appeal

**Layout and Spacing:**

- Consistent gap-2 spacing throughout the calendar grid
- Mobile-optimized touch targets (minimum 44px)
- Responsive design that adapts to different screen sizes
- Clean typography with proper font weights and sizes

### Visual States and Interactions

**Interactive Elements:**

- Hover effects with smooth color transitions
- Focus states with ring indicators for accessibility
- Clear visual distinction between enabled and disabled elements
- Animated transitions for state changes

**Selection Indicators:**

- Current date highlighted with special border styling
- Selected dates/months with solid background colors
- Range selections with gradient background between start and end
- Quick selection options with active state indicators

### CSS Architecture

The styling is organized in a modular structure:

```css
/* Primary theme colors defined in src/css/main.css */
@theme {
  --color-primary-50: #fdf2f8;
  --color-primary-100: #fce7f3;
  --color-primary-200: #fbcfe8;
  --color-primary-300: #f9a8d4;
  --color-primary-400: #f472b6;
  --color-primary-500: #e54993;
  --color-primary-600: #db2777;
  --color-primary-700: #be185d;
  --color-primary-800: #9d174d;
  --color-primary-900: #831843;
  --color-primary-950: #500724;
}
```

**Component Structure:**

- Input field with calendar icon
- Dropdown panel with optional quick selection sidebar
- Navigation header with month/year controls
- Calendar grid or month grid depending on view
- Footer with clear selection button

### Responsive Behavior

**Mobile Devices:**

- Touch-friendly button sizes and spacing
- Optimized dropdown positioning to prevent overflow
- Simplified navigation controls for smaller screens

**Desktop Devices:**

- Hover states for enhanced interactivity
- Keyboard navigation support
- Larger dropdown panels with additional features

## Behavior and Logic

### Selection Logic

**Single Mode Behavior:**

- Click any selectable date/month to select it
- Previously selected value is replaced with new selection
- Dropdown closes automatically after selection
- Clear button resets selection to null

**Range Mode Behavior:**

- First click sets the start date/month
- Second click sets the end date/month (must be after start)
- If second click is before start, it becomes the new start
- Dropdown closes automatically after range completion
- Visual feedback shows partial selection state

### Quick Selection Features

**Preset Options (Date View + Range Mode only):**

- Today: Sets range to current date only
- Yesterday: Sets range to previous day only
- Last 30 days: Sets 30-day range ending today
- This month: Sets range from month start to today
- 6 months: Sets 180-day range ending today

**Quick Selection Behavior:**

- Instant selection with immediate dropdown close
- Visual indication of currently active quick selection
- Overrides any existing selection when clicked

### Validation and Constraints

**Date Boundary Enforcement:**

- Navigation buttons disable when reaching min/max limits
- Non-selectable dates appear grayed out and non-interactive
- Range selections automatically validate start/end relationships
- Invalid selections are prevented rather than corrected

**Error Handling:**

- Graceful handling of invalid date formats
- Fallback to safe defaults when parsing fails
- Optional chaining prevents runtime errors
- Clear visual feedback for constraint violations

### Navigation Controls

**Month/Year Navigation:**

- Previous/Next buttons for temporal navigation
- Smart boundary detection based on min/max dates
- Keyboard support for arrow key navigation
- Screen reader announcements for navigation changes

**Keyboard Accessibility:**

- Tab navigation through all interactive elements
- Enter and Space keys for selection
- Escape key to close dropdown
- Arrow keys for date navigation within calendar

## Project Structure and Architecture

### File Organization

```
vuejs-custom-datepicker/
├── src/
│   ├── components/
│   │   └── DatePicker.vue          # Main component (972 lines)
│   ├── css/
│   │   └── main.css               # Tailwind CSS configuration
│   ├── App.vue                    # Demo application with examples
│   └── main.ts                   # Application entry point
├── public/
│   └── favicon.ico               # Application icon
├── package.json                  # Dependencies and scripts
├── vite.config.ts               # Vite build configuration
├── tsconfig.json                # TypeScript configuration
├── eslint.config.js             # ESLint configuration
├── postcss.config.js            # PostCSS configuration
└── README.md                    # Documentation
```

### Component Architecture

**DatePicker.vue Structure:**

- Template: Input field with dropdown calendar interface
- Script: Vue 3 Composition API with TypeScript
- Styling: Tailwind CSS utility classes with custom theme

**Key Architectural Decisions:**

- Single-file component for easy integration
- Composition API for better code organization and reusability
- Computed properties for reactive state management
- Event-driven communication with parent components

### Dependencies and Configuration

**Core Dependencies:**

- Vue 3: Modern reactive framework
- TypeScript: Type safety and development experience
- Tailwind CSS v4: Utility-first styling system

**Development Tools:**

- Vite: Fast build tool and development server
- ESLint: Code quality and consistency
- PostCSS: CSS processing and optimization

## Development Workflow

### Available Scripts

```bash
# Development Commands
pnpm run dev              # Start development server with hot reload
pnpm run build            # Build optimized production bundle
pnpm run preview          # Preview production build locally

# Code Quality Commands
pnpm run lint             # Run ESLint for code quality checks
pnpm run lint:fix         # Automatically fix ESLint issues
pnpm run type-check       # Run TypeScript type checking

# Package Management
pnpm install              # Install all project dependencies
pnpm update              # Update dependencies to latest versions
```

### Development Setup

1. **Clone and Install:**

   ```bash
   git clone <repository-url>
   cd vuejs-custom-datepicker
   pnpm install
   ```

2. **Start Development Server:**

   ```bash
   pnpm run dev
   ```

3. **View Demo Application:**
   Open browser to `http://localhost:5173` to see component examples

### Code Quality Standards

**TypeScript Configuration:**

- Strict mode enabled for maximum type safety
- Comprehensive type definitions for all component interfaces
- Import/export optimization for better tree-shaking

**ESLint Rules:**

- Vue.js specific linting rules
- TypeScript integration for type-aware linting
- Consistent code formatting and style enforcement
- Accessibility rule checks

**Error Handling Patterns:**

- Try-catch blocks around date parsing operations
- Optional chaining for safe property access
- Fallback values for edge cases
- Graceful degradation for unsupported features

## Integration Guide

### Adding to Existing Vue Projects

1. **Copy Component Files:**

   ```bash
   cp src/components/DatePicker.vue your-project/src/components/
   cp src/css/main.css your-project/src/css/
   ```

2. **Install Required Dependencies:**

   ```bash
   pnpm add tailwindcss @tailwindcss/vite
   ```

3. **Update Tailwind Configuration:**

   ```javascript
   // tailwind.config.js
   import { defineConfig } from '@tailwindcss/vite'

   export default defineConfig({
     content: ['./src/**/*.{vue,js,ts}'],
     theme: {
       extend: {
         colors: {
           primary: {
             // Add custom color palette from main.css
           },
         },
       },
     },
   })
   ```

4. **Import and Use Component:**
   ```vue
   <script setup lang="ts">
   import DatePicker from '@/components/DatePicker.vue'
   </script>
   ```

### Customization Options

**Styling Customization:**

- Modify CSS custom properties in main.css for color scheme changes
- Override Tailwind classes through CSS specificity
- Create theme variants through CSS variables

**Behavior Customization:**

- Extend quick selection options by modifying the computed property
- Adjust date constraints logic in validation functions
- Customize display formats by updating format functions

**Internationalization Support:**

- Month names array can be replaced with localized versions
- Day names array supports different locale abbreviations
- Date format functions can be modified for regional preferences

## Use Cases and Applications

### Ideal Implementation Scenarios

**Business Applications:**

- Financial reporting systems requiring historical date selection
- Analytics dashboards with time period filters
- Transaction monitoring with date range constraints
- Audit systems needing controlled date access

**User Interface Contexts:**

- Form inputs requiring date validation
- Filter components for data tables
- Booking systems with availability constraints
- Calendar applications with custom date logic

**Technical Requirements:**

- Applications requiring strict TypeScript typing
- Projects using Tailwind CSS for consistent design
- Vue 3 applications leveraging Composition API
- Systems needing accessibility compliance

### Performance Characteristics

**Optimization Features:**

- Efficient virtual DOM updates through computed properties
- Minimal re-renders using reactive state management
- Lazy loading of dropdown content
- Optimized event listener management

**Memory Management:**

- Proper cleanup of event listeners on component unmount
- Efficient state management without memory leaks
- Optimized watcher patterns for better performance
- Clean component lifecycle management

## Troubleshooting and Support

### Common Issues and Solutions

**Date Format Problems:**

- Ensure date strings match expected YYYY-MM-DD format
- Verify timezone handling for consistent display
- Check min/max date constraint formats

**Styling Conflicts:**

- Verify Tailwind CSS is properly configured
- Check for CSS specificity conflicts
- Ensure custom theme colors are defined

**TypeScript Errors:**

- Verify all required props are provided
- Check model value type matches expected format
- Ensure event handler signatures match emitted events

### Debugging Strategies

**Development Tools:**

- Use Vue DevTools for reactive state inspection
- Browser developer tools for DOM and styling analysis
- TypeScript compiler for type checking and error identification

**Logging and Monitoring:**

- Component emits detailed events for state tracking
- Console warnings for invalid prop combinations
- Error boundaries prevent application crashes

### Contributing Guidelines

**Code Contribution Process:**

1. Fork repository and create feature branch
2. Implement changes with proper TypeScript types
3. Ensure all ESLint rules pass without errors
4. Test component in all supported modes and views
5. Update documentation for any API changes
6. Submit pull request with clear description

**Testing Requirements:**

- Manual testing across all component configurations
- Verification of accessibility features
- Cross-browser compatibility testing
- Mobile device testing for responsive behavior

## License and Attribution

This project is released under the MIT License, allowing for free use in both personal and commercial applications. The component can be freely modified, distributed, and integrated into other projects without restriction.

For questions, issues, or feature requests, please refer to the project repository or create an issue with detailed information about your use case.
