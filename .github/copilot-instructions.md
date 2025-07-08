# Copilot Instructions - Copilot Metrics Viewer

## Project Overview
This is a production-ready Nuxt.js 3 application for viewing GitHub Copilot metrics across enterprises, organizations, and teams. Built with TypeScript, Vuetify UI components, and Chart.js for data visualization, it supports multiple authentication methods (GitHub OAuth, Personal Access Token) and deployment strategies (Azure Container Apps, Docker, local development). The project follows strict architectural patterns with server-side rendering, comprehensive testing, and enterprise-grade security.

## Project Structure & Conventions

### Directory Structure
```
app/
├── components/          # Vue components (PascalCase naming)
│   ├── ApiResponse.vue      # API response display component
│   ├── BreakdownComponent.vue   # Metrics breakdown visualization
│   ├── CopilotChatViewer.vue   # Chat metrics display
│   ├── MainComponent.vue       # Main dashboard component
│   ├── MetricsViewer.vue       # Primary metrics visualization
│   └── SeatsAnalysisViewer.vue # Seat utilization analysis
├── layouts/            # Nuxt layout components
│   └── default.vue         # Main application layout with Vuetify v-app
├── pages/              # File-based routing (Nuxt convention)
│   └── index.vue           # Home page with dashboard
├── types/              # TypeScript type definitions
│   ├── auth.d.ts           # Authentication-related types
│   └── metricsApiResponse.d.ts # API response type definitions
├── model/              # Data models and business logic
│   ├── Breakdown.ts        # Breakdown data model
│   ├── Copilot_Metrics.ts  # Copilot metrics model
│   ├── Metrics.ts          # General metrics model
│   ├── MetricsToUsageConverter.ts # Data transformation utilities
│   ├── MetricsValidator.ts # Data validation logic
│   └── Seat.ts            # Seat data model
├── assets/             # Static assets and SCSS files
└── plugins/            # Nuxt plugins

server/
├── api/                # Backend API routes (RESTful endpoints)
│   ├── health.ts          # Health check endpoint (/api/health)
│   ├── live.ts           # Kubernetes liveness probe (/api/live)
│   ├── metrics.ts        # Core metrics API (/api/metrics)
│   ├── ready.ts          # Kubernetes readiness probe (/api/ready)
│   └── seats.ts          # Seat management API (/api/seats)
├── middleware/         # Server middleware (authentication, CORS, etc.)
├── modules/           # Server modules
├── plugins/           # Server plugins (HTTP agent configuration)
└── routes/            # Additional server routes

infra/                 # Infrastructure as Code (Azure Bicep)
├── main.bicep            # Main infrastructure template
├── main.parameters.json  # Environment-specific parameters
├── modules/             # Reusable Bicep modules
└── shared/             # Shared infrastructure resources

azure-deploy/          # Deployment configurations
├── with-app-registration/  # GitHub OAuth deployment
├── with-token/           # Personal Access Token deployment
└── dns/                 # Custom domain configurations
```

### Naming Conventions
- **Components**: PascalCase (e.g., `MetricsViewer.vue`, `BreakdownComponent.vue`, `CopilotChatViewer.vue`)
- **Files**: PascalCase for components, kebab-case for utilities
- **API Routes**: lowercase with descriptive names (e.g., `health.ts`, `metrics.ts`, `seats.ts`)
- **Types**: PascalCase interfaces (e.g., `MetricsApiResponse`, `CopilotMetrics`, `Breakdown`)
- **Models**: PascalCase classes (e.g., `Metrics.ts`, `Breakdown.ts`, `Seat.ts`)
- **Environment Variables**: SCREAMING_SNAKE_CASE with `NUXT_` prefix for runtime config

### Configuration Patterns
- **Public Config**: Client-accessible via `NUXT_PUBLIC_*` prefix
- **Private Config**: Server-only via `NUXT_*` prefix
- **Runtime Access**: Use `useRuntimeConfig()` for accessing configuration
- **Environment Scopes**: Support for `enterprise`, `organization`, and `team` scopes

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
  const logger = console // Use structured logging in production
  
  try {
    // Validate request and extract parameters
    const query = getQuery(event)
    
    // Access scope-specific context from middleware
    const { scope, org, ent, team } = event.context
    
    // Build GitHub API URL based on scope
    let apiUrl = ''
    switch (scope) {
      case 'enterprise':
        apiUrl = `https://api.github.com/enterprises/${ent}/copilot/metrics`
        break
      case 'organization':
        apiUrl = `https://api.github.com/orgs/${org}/copilot/metrics`
        break
      case 'team':
        apiUrl = `https://api.github.com/orgs/${org}/team/${team}/copilot/metrics`
        break
    }
    
    // Handle mock data for development/testing
    if (config.public.isDataMocked) {
      const mockData = await readMockData()
      return {
        status: 'success',
        data: mockData,
        timestamp: new Date().toISOString(),
        source: 'mock'
      }
    }
    
    // Make authenticated GitHub API call
    const response = await $fetch(apiUrl, {
      headers: {
        'Authorization': `Bearer ${config.githubToken}`,
        'Accept': 'application/vnd.github.v3+json',
        'User-Agent': 'copilot-metrics-viewer'
      }
    })
    
    // Return structured response
    return {
      status: 'success',
      data: response,
      timestamp: new Date().toISOString(),
      source: 'github-api'
    }
  } catch (error) {
    logger.error('API Error:', error)
    
    // Handle specific GitHub API errors
    if (error.status === 401) {
      throw createError({
        statusCode: 401,
        statusMessage: 'GitHub authentication failed'
      })
    }
    
    if (error.status === 403) {
      throw createError({
        statusCode: 403,
        statusMessage: 'Insufficient GitHub permissions'
      })
    }
    
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

### Nuxt.js 3 Patterns
- **Configuration**: Use `useRuntimeConfig()` for accessing both public and private config
- **Data Fetching**: Leverage `$fetch` for API calls with built-in error handling
- **Navigation**: Use `navigateTo()` for programmatic navigation
- **SEO**: Implement `useHead()` for meta tags and SEO optimization
- **Server Utilities**: Place server-side logic in `/server` directory
- **SSR**: Enable server-side rendering for better performance and SEO
- **File-based Routing**: Use `/pages` directory for automatic route generation

### Authentication Patterns
```typescript
// GitHub OAuth Authentication
const { session, user, login, logout } = useAuth()

// Check authentication status
const isAuthenticated = computed(() => !!session.value)

// Handle login
const handleLogin = async () => {
  await login('github')
}

// Access user information
const userInfo = computed(() => user.value)

// Personal Access Token Authentication
const config = useRuntimeConfig()
const hasToken = computed(() => !!config.githubToken)
```

### Configuration Management
```typescript
// Runtime configuration access
const config = useRuntimeConfig()

// Public configuration (client-side accessible)
const {
  isDataMocked,
  scope,
  githubOrg,
  githubEnt,
  githubTeam,
  usingGithubAuth,
  version
} = config.public

// Private configuration (server-side only)
const {
  githubToken,
  session: { maxAge, password },
  oauth: { github: { clientId, clientSecret } }
} = config // Only available on server

// Environment-specific settings
const isDevelopment = process.env.NODE_ENV === 'development'
const isProduction = process.env.NODE_ENV === 'production'
```

### Vuetify Patterns
- Always use `v-` prefixed components
- Utilize Vuetify's responsive grid system (`v-container`, `v-row`, `v-col`)
- Use Vuetify's color system and theme variables
- Implement proper Material Design patterns

### Chart.js Integration
```typescript
// Use vue-chartjs wrapper components
import { Line, Bar, Doughnut } from 'vue-chartjs'
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js'

// Register required Chart.js components
ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
)

// Define chart options as computed properties
const chartOptions = computed(() => ({
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: {
      position: 'top' as const,
    },
    title: {
      display: true,
      text: 'Copilot Usage Metrics'
    }
  },
  scales: {
    y: {
      beginAtZero: true
    }
  }
}))

// Handle data reactivity properly
const chartData = computed(() => ({
  labels: metrics.value?.map(m => m.date) || [],
  datasets: [
    {
      label: 'Suggestions',
      data: metrics.value?.map(m => m.suggestions_count) || [],
      backgroundColor: 'rgba(53, 162, 235, 0.5)',
      borderColor: 'rgba(53, 162, 235, 1)',
      borderWidth: 1
    }
  ]
}))
```

### Health Check Patterns
```typescript
// Health check endpoint implementation
export default defineEventHandler(async (event) => {
  const config = useRuntimeConfig(event)
  
  // Check application health
  const healthChecks = {
    database: await checkDatabaseConnection(),
    githubApi: await checkGitHubApiConnection(),
    memory: process.memoryUsage(),
    uptime: process.uptime()
  }
  
  return {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    version: config.public.version,
    checks: healthChecks
  }
})

// Kubernetes probes
// Liveness probe: /api/live
// Readiness probe: /api/ready
```

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

### Code Quality & Architecture
- **Strict TypeScript**: No `any` types, use proper interface definitions
- **Component composition**: Break down large components (follow single responsibility)
- **Error boundaries**: Implement proper error handling with user feedback
- **Performance**: Use computed properties, avoid unnecessary re-renders
- **Accessibility**: Follow Vuetify's a11y guidelines and ARIA standards
- **Separation of concerns**: Keep business logic in models, UI logic in components
- **API consistency**: Follow RESTful conventions with consistent response structures

### Security Best Practices
- **Token Management**: Never expose GitHub tokens to client-side
- **Session Security**: Use encrypted sessions with proper expiration (6 hours)
- **Input Validation**: Validate all inputs on both client and server side
- **Error Handling**: Don't expose sensitive information in error messages
- **HTTPS Only**: Enforce TLS encryption for all communications
- **CORS Configuration**: Properly configure CORS for production environments

### State Management Patterns
```typescript
// Local component state
const metrics = ref<CopilotMetrics[]>([])
const loading = ref(false)
const error = ref<string | null>(null)

// Shared state across components
const globalMetrics = useState('metrics', () => [])

// Computed state derivations
const totalSuggestions = computed(() =>
  metrics.value.reduce((sum, m) => sum + m.suggestions_count, 0)
)

// Proper error state handling
const fetchMetrics = async () => {
  try {
    loading.value = true
    error.value = null
    const result = await $fetch('/api/metrics')
    metrics.value = result.data
  } catch (err) {
    error.value = 'Failed to load metrics'
    console.error('Metrics fetch error:', err)
  } finally {
    loading.value = false
  }
}
```

### Performance Optimization
- **Code Splitting**: Leverage Nuxt's automatic code splitting
- **Lazy Loading**: Use dynamic imports for heavy components
- **Caching**: Implement proper API response caching
- **Bundle Analysis**: Monitor bundle size and dependencies
- **SSR Optimization**: Use server-side rendering for better initial load

### Deployment & Environment Patterns
```typescript
// Environment-specific configuration
const config = {
  development: {
    NUXT_PUBLIC_IS_DATA_MOCKED: true,
    NUXT_PUBLIC_SCOPE: 'organization',
    NUXT_PUBLIC_USING_GITHUB_AUTH: false
  },
  production: {
    NUXT_PUBLIC_IS_DATA_MOCKED: false,
    NUXT_PUBLIC_SCOPE: 'enterprise',
    NUXT_PUBLIC_GITHUB_ENT: 'compunnel',
    NUXT_GITHUB_TOKEN: process.env.GITHUB_TOKEN,
    NUXT_SESSION_PASSWORD: process.env.SESSION_PASSWORD
  }
}

// Docker deployment patterns
// Multi-stage builds for optimization
// Non-root user execution for security
// Health check endpoints for Kubernetes

// Azure Container Apps deployment
// Infrastructure as Code with Bicep templates
// Managed identity for secure authentication
// Auto-scaling configuration
```

### API Design Patterns
- **Consistent Response Structure**: All API endpoints return standardized responses
- **Error Handling**: Proper HTTP status codes and error messages
- **Authentication**: Support both OAuth and Personal Access Token
- **Rate Limiting**: Handle GitHub API rate limits gracefully
- **Logging**: Comprehensive request/response logging for debugging

### Styling Guidelines
- **Vuetify Design System**: Use Material Design principles consistently
- **Responsive Design**: Mobile-first approach with Vuetify's grid system
- **Theme Management**: Support light/dark themes with Vuetify
- **Custom Styling**: Use scoped styles and SCSS for customizations
- **Performance**: Minimize CSS bundle size, use tree-shaking

## Anti-Patterns to Avoid

### ❌ Security Anti-Patterns
```typescript
// Don't expose GitHub tokens to client-side
const githubToken = 'ghp_...' // Never do this in frontend code

// Don't use any types for sensitive data
const authData: any = await $fetch('/api/auth') // Lacks type safety

// Don't ignore authentication in API routes
export default defineEventHandler(async (event) => {
  // Missing authentication check
  return await fetchSensitiveData()
})

// Don't store secrets in frontend code
const config = {
  clientSecret: 'secret123' // Never expose secrets
}
```

### ❌ Architecture Anti-Patterns
```typescript
// Don't mix concerns in components
<script setup>
// Don't put business logic directly in components
const processMetrics = (data) => {
  // Complex business logic should be in models/
  return data.map(item => /* complex processing */)
}

// Don't mutate props directly
props.metrics.push(newMetric) // Use events instead

// Don't use inline styles extensively
<v-card style="margin: 20px; padding: 15px; color: red;">

// Don't ignore error states
const data = await $fetch('/api/metrics') // No error handling
</script>
```

### ❌ Performance Anti-Patterns
```typescript
// Don't use reactive() for large datasets
const massiveData = reactive(millionsOfRecords) // Use ref() instead

// Don't forget to cleanup watchers
watch(someRef, () => {
  // Missing cleanup can cause memory leaks
})

// Don't use computed for side effects
const badComputed = computed(() => {
  localStorage.setItem('data', someValue) // Side effects in computed
  return processedData
})
```

### ❌ API Anti-Patterns
```typescript
// Don't ignore GitHub API rate limits
const response = await $fetch(githubUrl) // No rate limit handling

// Don't return inconsistent response structures
// Endpoint 1
return { data: metrics }
// Endpoint 2  
return { result: seats } // Inconsistent structure

// Don't expose internal errors to users
catch (error) {
  throw new Error(error.stack) // Exposes internal details
}
```

### ✅ Correct Implementations
```typescript
// Use proper TypeScript interfaces
interface MetricsApiResponse {
  data: CopilotMetrics[]
  status: 'success' | 'error'
  timestamp: string
  source: 'github-api' | 'mock'
}
const response: MetricsApiResponse = await $fetch('/api/metrics')

// Use events for parent-child communication
const emit = defineEmits<{
  'update:metrics': [value: CopilotMetrics[]]
  'metrics-error': [error: string]
}>()

emit('update:metrics', newMetrics)

// Use Vuetify classes for styling
<v-card class="ma-4 pa-4 elevation-2">
  <v-card-title class="text-h5">Metrics Dashboard</v-card-title>
</v-card>

// Proper error handling with user feedback
const { data, error, pending } = await useFetch('/api/metrics', {
  retry: 3,
  retryDelay: 1000,
  onResponseError({ response }) {
    console.error('API Error:', response.status)
  }
})

// Use Composition API with proper TypeScript
const metrics = ref<CopilotMetrics[]>([])
const loading = ref(false)

const fetchMetrics = async () => {
  try {
    loading.value = true
    const result = await $fetch<MetricsApiResponse>('/api/metrics')
    metrics.value = result.data
  } catch (err) {
    console.error('Failed to fetch metrics:', err)
  } finally {
    loading.value = false
  }
}

// Proper authentication patterns
export default defineEventHandler(async (event) => {
  const config = useRuntimeConfig(event)
  
  // Check authentication
  if (!config.githubToken && !config.public.usingGithubAuth) {
    throw createError({
      statusCode: 401,
      statusMessage: 'Authentication required'
    })
  }
  
  // Proceed with authenticated request
})

// Secure configuration management
const config = useRuntimeConfig()
const isAuthenticated = computed(() => 
  config.public.usingGithubAuth ? !!session.value : !!config.githubToken
)
```

## Development Workflow

### Local Development Setup
```bash
# Clone and setup
git clone <repository>
cd copilot-metrics-viewer
npm install

# Environment configuration
cp .env.example .env
# Configure your GitHub token and scope settings

# Development server with hot reload
npm run dev

# Access application at http://localhost:3000
```

### Code Quality & Linting
- **Pre-commit**: Run `npm run lint` before committing changes
- **Auto-fix**: Use `npm run lint:fix` to automatically fix style issues
- **TypeScript**: Run `npm run typecheck` for type validation
- **Formatting**: Follow Prettier-compatible formatting standards
- **ESLint Config**: Extends Nuxt's recommended ESLint configuration

### Testing Strategy
```bash
# Unit tests with Vitest
npm run test

# E2E tests with Playwright
npm run test:e2e

# Watch mode for development
npm run test -- --watch

# Coverage reports
npm run test -- --coverage
```

### Build & Deployment Process
```bash
# Production build
npm run build

# Preview production build locally
npm run preview

# Docker build for containerization
docker build -t copilot-metrics .

# Multi-architecture build
docker buildx build --platform linux/amd64,linux/arm64 -t copilot-metrics .

# Azure deployment with azd
azd auth login
azd up
```

### Environment Management
```bash
# Development environment
NUXT_PUBLIC_IS_DATA_MOCKED=true
NUXT_PUBLIC_SCOPE=organization
NUXT_PUBLIC_USING_GITHUB_AUTH=false

# Staging environment
NUXT_PUBLIC_IS_DATA_MOCKED=false
NUXT_PUBLIC_SCOPE=enterprise
NUXT_GITHUB_TOKEN=<your-token>

# Production environment (Azure)
# Managed through Azure Key Vault and Container Apps configuration
```

## Dependencies Management

### Core Dependencies Strategy
- **Nuxt.js Ecosystem**: Keep Nuxt, Vue, and related packages aligned
- **UI Framework**: Maintain Vuetify 3.x compatibility with Material Design
- **Charts**: Keep Chart.js and vue-chartjs versions synchronized
- **TypeScript**: Use latest stable TypeScript version for best type safety
- **Testing**: Maintain Vitest and Playwright for comprehensive test coverage

### Version Management
```json
{
  "dependencies": {
    "nuxt": "^3.17.5",
    "vue": "latest",
    "vuetify": "^3.7.3",
    "chart.js": "^4.4.7",
    "vue-chartjs": "^5.3.2",
    "typescript": "^5.6.3"
  },
  "devDependencies": {
    "@playwright/test": "^1.49.1",
    "vitest": "^3.0.5",
    "eslint": "^9.17.0"
  }
}
```

### Security Updates
- **Regular Audits**: Run `npm audit` regularly for vulnerability checks
- **Dependency Updates**: Keep security-critical packages updated
- **Lock Files**: Commit `package-lock.json` for reproducible builds
- **Vulnerability Monitoring**: Use tools like Dependabot or Snyk

### Performance Considerations
- **Bundle Analysis**: Monitor bundle size with `nuxt analyze`
- **Tree Shaking**: Ensure unused code is eliminated
- **Dynamic Imports**: Use dynamic imports for code splitting
- **CDN Optimization**: Leverage CDN for static assets

---

## Quick Reference

### Common Composables & APIs
- `useRuntimeConfig()` - Access public/private configuration
- `useFetch(url, options)` - Data fetching with SSR support and caching
- `useState(key, init)` - Shared reactive state across components
- `useHead(meta)` - SEO meta tags and document head management
- `useNuxtApp()` - Access Nuxt application context and plugins
- `useAuth()` - Authentication state and methods (nuxt-auth-utils)
- `navigateTo(path)` - Programmatic navigation
- `createError(options)` - Create and throw HTTP errors

### GitHub API Integration
```typescript
// Enterprise metrics
GET /enterprises/{enterprise}/copilot/metrics

// Organization metrics  
GET /orgs/{org}/copilot/metrics

// Team metrics
GET /orgs/{org}/team/{team}/copilot/metrics

// Seat management
GET /orgs/{org}/copilot/billing/seats
```

### Vuetify Essential Components
- `v-app` - Root application wrapper with theme support
- `v-container`, `v-row`, `v-col` - Responsive grid layout system
- `v-card`, `v-card-title`, `v-card-text` - Content containers
- `v-btn` - Material Design buttons with variants
- `v-data-table` - Advanced data tables with sorting/filtering
- `v-chart` - Chart.js integration components
- `v-dialog` - Modal dialogs and overlays
- `v-snackbar` - Toast notifications
- `v-progress-circular`, `v-progress-linear` - Loading indicators

### Environment Variables Reference
```bash
# Public (client-accessible)
NUXT_PUBLIC_IS_DATA_MOCKED=false
NUXT_PUBLIC_SCOPE=enterprise|organization|team
NUXT_PUBLIC_GITHUB_ORG=organization-name
NUXT_PUBLIC_GITHUB_ENT=enterprise-name
NUXT_PUBLIC_GITHUB_TEAM=team-name
NUXT_PUBLIC_USING_GITHUB_AUTH=true|false

# Private (server-only)
NUXT_GITHUB_TOKEN=ghp_...
NUXT_SESSION_PASSWORD=secure-session-key
NUXT_OAUTH_GITHUB_CLIENT_ID=github-app-id
NUXT_OAUTH_GITHUB_CLIENT_SECRET=github-app-secret
```

### File Structure Commands
```bash
# Component creation
touch app/components/NewMetricsComponent.vue

# API endpoint creation
touch server/api/new-endpoint.ts

# Type definition
touch app/types/newApiTypes.d.ts

# Data model
touch app/model/NewDataModel.ts

# Layout creation
touch app/layouts/custom.vue

# Page creation
touch app/pages/dashboard.vue

# Server middleware
touch server/middleware/auth.ts

# Plugin creation
touch app/plugins/chart-config.client.ts
```

### Development Commands
```bash
# Development
npm run dev              # Start development server
npm run build            # Build for production
npm run preview          # Preview production build
npm run generate         # Generate static site

# Quality Assurance
npm run lint             # Run ESLint
npm run lint:fix         # Auto-fix linting issues
npm run typecheck        # TypeScript type checking
npm run test             # Run unit tests
npm run test:e2e         # Run E2E tests

# Docker Operations
docker build -t copilot-metrics .
docker run -p 3000:80 copilot-metrics

# Azure Deployment
azd auth login
azd up                   # Deploy to Azure
azd down                 # Destroy Azure resources
```

### Debugging & Monitoring
```typescript
// Server-side logging
console.log('Server log:', data)

// Client-side debugging
console.debug('Debug info:', state)

// Error tracking
try {
  await riskyOperation()
} catch (error) {
  console.error('Operation failed:', error)
  // Send to monitoring service
}

// Performance monitoring
const start = performance.now()
await operation()
const duration = performance.now() - start
console.log(`Operation took ${duration}ms`)
```

---
*Last updated: July 8, 2025*
