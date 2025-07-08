# Copilot Instructions - Copilot Metrics Viewer

## Project Overview
This is a modern Nuxt.js 3 application for viewing GitHub Copilot metrics with TypeScript, Vuetify UI components, and Chart.js for data visualization. The project follows strict TypeScript patterns, uses server-side rendering, and maintains a clean separation between frontend components and backend API routes.

## Project Structure & Conventions

### Directory Structure
```
app/
├── components/          # Vue components (PascalCase naming)
├── layouts/            # Nuxt layout components
├── pages/              # File-based routing
├── types/              # TypeScript type definitions
├── model/              # Data models and business logic
├── assets/             # Static assets
└── plugins/            # Nuxt plugins

server/
├── api/                # Backend API routes
├── middleware/         # Server middleware
└── routes/             # Additional server routes
```

### Naming Conventions
- **Components**: PascalCase (e.g., `MetricsViewer.vue`, `BreakdownComponent.vue`)
- **Files**: PascalCase for components, kebab-case for utilities
- **API Routes**: lowercase with descriptive names (e.g., `health.ts`, `metrics.ts`)
- **Types**: PascalCase interfaces (e.g., `MetricsApiResponse`, `CopilotMetrics`)
- **Models**: PascalCase classes (e.g., `Metrics.ts`, `Breakdown.ts`)

## Code Patterns & Best Practices

### Vue Component Structure
```vue
<template>
  <!-- Use Vuetify components with v- prefix -->
  <v-container>
    <v-card>
      <!-- Content here -->
    </v-card>
  </v-container>
</template>

<script lang="ts" setup>
// Use Composition API with TypeScript
import type { ComponentType } from '@/types/componentType'

// Define props with TypeScript
interface Props {
  data: ComponentType
  loading?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  loading: false
})

// Use Nuxt composables
const config = useRuntimeConfig()
const { $fetch } = useNuxtApp()

// Reactive state
const state = ref<ComponentType | null>(null)
const isLoading = ref(false)

// Computed properties
const formattedData = computed(() => {
  return state.value ? formatData(state.value) : null
})

// Methods
const fetchData = async () => {
  try {
    isLoading.value = true
    state.value = await $fetch('/api/endpoint')
  } catch (error) {
    console.error('Error fetching data:', error)
  } finally {
    isLoading.value = false
  }
}
</script>

<style scoped>
/* Use scoped styles */
.custom-class {
  /* Vuetify-compatible styling */
}
</style>
```

### Server API Routes
```typescript
export default defineEventHandler(async (event) => {
  const config = useRuntimeConfig(event)
  
  try {
    // Validate request
    const query = getQuery(event)
    
    // Business logic
    const result = await processData(query)
    
    // Return structured response
    return {
      status: 'success',
      data: result,
      timestamp: new Date().toISOString()
    }
  } catch (error) {
    throw createError({
      statusCode: 500,
      statusMessage: 'Internal server error'
    })
  }
})
```

### TypeScript Type Definitions
```typescript
// Use interfaces for object shapes
interface MetricsApiResponse {
  metrics: Metrics[]
  usage: CopilotMetrics[]
  timestamp: string
}

// Use types for unions and primitives
type Status = 'loading' | 'success' | 'error'
type ChartType = 'line' | 'bar' | 'doughnut'

// Export types from model files
export type { MetricsApiResponse, Status, ChartType }
```

## Framework-Specific Patterns

### Nuxt.js Patterns
- Use `useRuntimeConfig()` for configuration access
- Leverage `$fetch` for API calls instead of axios
- Use `navigateTo()` for programmatic navigation
- Implement `useHead()` for SEO meta tags
- Use server-side utilities in `/server` directory

### Vuetify Patterns
- Always use `v-` prefixed components
- Utilize Vuetify's responsive grid system (`v-container`, `v-row`, `v-col`)
- Use Vuetify's color system and theme variables
- Implement proper Material Design patterns

### Chart.js Integration
- Use `vue-chartjs` wrapper components
- Define chart options as computed properties
- Ensure responsive chart configurations
- Handle data reactivity properly

## Testing Conventions

### Unit Tests (Vitest)
```typescript
import { describe, test, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import ComponentName from '@/components/ComponentName.vue'

describe('ComponentName', () => {
  test('renders correctly', () => {
    const wrapper = mount(ComponentName, {
      props: { /* test props */ }
    })
    expect(wrapper.text()).toContain('expected text')
  })
})
```

### E2E Tests (Playwright)
```typescript
import { test, expect } from '@playwright/test'

test('metrics page loads correctly', async ({ page }) => {
  await page.goto('/')
  await expect(page.locator('h1')).toContainText('Copilot Metrics')
})
```

## Best Practices

### Code Quality
- **Always use TypeScript**: No `any` types, prefer strict typing
- **Component composition**: Break down large components into smaller, reusable ones
- **Error handling**: Implement proper try-catch blocks and user feedback
- **Performance**: Use computed properties and watch for reactive data
- **Accessibility**: Follow Vuetify's accessibility guidelines

### State Management
- Use `ref()` and `reactive()` for local component state
- Use `useState()` for shared state across components
- Implement proper error states and loading states
- Use Nuxt's built-in state management patterns

### API Design
- Follow RESTful conventions for API endpoints
- Return consistent response structures
- Implement proper HTTP status codes
- Use TypeScript for request/response types

### Styling
- Use Vuetify's design system consistently
- Implement responsive design patterns
- Use scoped styles to avoid conflicts
- Follow Material Design principles

## Anti-Patterns to Avoid

### ❌ Don't Do This
```typescript
// Don't use any types
const data: any = await $fetch('/api/metrics')

// Don't mutate props directly
props.data.push(newItem)

// Don't use inline styles extensively
<v-card style="margin: 20px; padding: 15px;">

// Don't ignore error handling
const result = await $fetch('/api/data') // No try-catch

// Don't use Vue 2 patterns
export default {
  data() { return {} }, // Use Composition API instead
  methods: {}
}
```

### ✅ Do This Instead
```typescript
// Use proper TypeScript
interface ApiResponse {
  data: MetricsData[]
  status: string
}
const data: ApiResponse = await $fetch('/api/metrics')

// Emit events to modify parent data
const emit = defineEmits<{
  update: [value: DataType]
}>()

// Use Vuetify classes
<v-card class="ma-4 pa-4">

// Proper error handling
try {
  const result = await $fetch('/api/data')
} catch (error) {
  console.error('API Error:', error)
}

// Use Composition API
const { data, pending, error } = await $fetch('/api/data')
```

## Development Workflow

### Code Style
- Run `npm run lint` before committing
- Use `npm run lint:fix` to auto-fix issues
- Follow the existing ESLint configuration
- Use consistent formatting (Prettier-compatible)

### Testing
- Write unit tests for components and utilities
- Add E2E tests for critical user flows
- Run `npm run test` for unit tests
- Run `npm run test:e2e` for E2E tests

### Build & Deployment
- Use `npm run build` for production builds
- Test with `npm run preview` before deployment
- Ensure Docker builds work with `docker build`
- Follow Azure Container Apps deployment patterns

## Dependencies Management

### Core Dependencies
- Keep Nuxt, Vue, and Vuetify versions aligned
- Use TypeScript strictly throughout
- Maintain Chart.js and vue-chartjs compatibility

### Development Dependencies
- Keep testing frameworks updated
- Maintain ESLint and Vitest configurations
- Use exact versions for critical dependencies

---

## Quick Reference

### Common Composables
- `useRuntimeConfig()` - Access configuration
- `useFetch()` - Data fetching with SSR
- `useState()` - Shared reactive state
- `useHead()` - SEO and meta tags
- `useNuxtApp()` - Access Nuxt instance

### Vuetify Components
- `v-app` - Root application component
- `v-container`, `v-row`, `v-col` - Layout grid
- `v-card` - Content containers
- `v-btn` - Buttons with Material Design
- `v-data-table` - Data tables with sorting/filtering

### File Structure Commands
```bash
# Create new component
touch app/components/NewComponent.vue

# Create new API route
touch server/api/new-endpoint.ts

# Create new type definition
touch app/types/newType.d.ts

# Create new model
touch app/model/NewModel.ts
```

---
*Last updated: July 8, 2025*
