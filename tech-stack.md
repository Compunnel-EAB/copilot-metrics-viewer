# Tech Stack - Copilot Metrics Viewer

## Project Overview
A modern, production-ready Vue.js application designed for viewing GitHub Copilot metrics, with a focus on data visualization, performance, and developer experience.

## Frontend Framework & Core
- **Nuxt.js 3** (v3.17.5) - Vue.js meta-framework with SSR enabled
- **Vue.js** (latest) - JavaScript framework
- **TypeScript** (v5.6.3) - Type-safe JavaScript

## UI & Styling
- **Vuetify** (v3.7.3) - Vue Material Design component framework
- **Material Design Icons** (@mdi/font v7.4.47)
- **Roboto Font** (roboto-fontface v0.10.0)
- **Sass** (sass-embedded v1.80.3) - CSS preprocessor
- **WebFont Loader** (v1.6.28) - Font loading optimization

## Data Visualization
- **Chart.js** (v4.4.7) - JavaScript charting library
- **Vue-ChartJS** (v5.3.2) - Vue.js wrapper for Chart.js

## Authentication & Security
- **nuxt-auth-utils** (v0.5.7) - Authentication utilities for Nuxt

## Development Tools & Build
- **Vite** - Build tool (bundled with Nuxt)
- **esbuild** (>=0.25.0) - JavaScript bundler
- **ESLint** (v9.17.0) - Code linting with Nuxt ESLint config
- **Node.js** (v24 Alpine for build, v23 Alpine for production)

## Testing Framework
- **Vitest** (v3.0.5) - Unit testing framework
- **Vue Test Utils** (v2.4.6) - Vue component testing utilities
- **Playwright** (v1.49.1) - End-to-end testing
- **Happy DOM** (v16.5.3) - DOM implementation for testing
- **Nuxt Test Utils** (v3.15.4) - Testing utilities for Nuxt

## Deployment & Infrastructure
- **Docker** - Containerization (multi-stage builds)
- **Azure Container Apps** - Cloud hosting platform
- **Azure Developer CLI** - Deployment tooling

## Development Features
- **Server-Side Rendering (SSR)** - Enabled for better SEO and performance
- **Hot Module Replacement** - Development server with live reloading
- **TypeScript** - Full type safety across the application
- **Code Linting** - ESLint with auto-fix capabilities
- **Font Optimization** - Nuxt Fonts module for performance

## Project Architecture
- **Full-stack application** with both frontend and backend API routes
- **Modular structure** with organized components, pages, layouts, and server API
- **Container-ready** with production-optimized Docker builds
- **CI/CD ready** with comprehensive testing setup
- **Modern ES modules** - Using ES module syntax throughout

## Key Dependencies
```json
{
  "nuxt": "^3.17.5",
  "vue": "latest",
  "vuetify": "^3.7.3",
  "chart.js": "^4.4.7",
  "vue-chartjs": "^5.3.2",
  "typescript": "^5.6.3",
  "@playwright/test": "^1.49.1",
  "vitest": "^3.0.5"
}
```

## Scripts Available
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run generate` - Generate static site
- `npm run preview` - Preview production build
- `npm run test` - Run unit tests with Vitest
- `npm run test:e2e` - Run end-to-end tests with Playwright
- `npm run lint` - Lint code with ESLint
- `npm run lint:fix` - Auto-fix linting issues

---
*Generated on: July 8, 2025*
