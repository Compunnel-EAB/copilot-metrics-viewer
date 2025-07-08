# Architecture Documentation - Copilot Metrics Viewer

## Overview

The Copilot Metrics Viewer is a full-stack web application built with Nuxt.js 3 that provides comprehensive visualization and analysis of GitHub Copilot usage metrics. The application supports multiple deployment scenarios and authentication methods, making it suitable for both enterprise and organization-level deployments.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                             │
├─────────────────────────────────────────────────────────────────┤
│  Vue.js 3 + Nuxt.js 3 Frontend                                │
│  ├── Vuetify UI Components                                    │
│  ├── Chart.js Data Visualization                              │
│  ├── TypeScript Type Safety                                   │
│  └── SSR/SSG Capabilities                                     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     APPLICATION LAYER                           │
├─────────────────────────────────────────────────────────────────┤
│  Nuxt.js 3 Server Layer                                       │
│  ├── Server-Side API Routes (/server/api)                     │
│  ├── Authentication Middleware                                │
│  ├── Data Processing & Validation                             │
│  └── Runtime Configuration                                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    INTEGRATION LAYER                            │
├─────────────────────────────────────────────────────────────────┤
│  GitHub API Integration                                        │
│  ├── Enterprise Metrics API                                   │
│  ├── Organization Metrics API                                 │
│  ├── Team Metrics API                                         │
│  └── GitHub OAuth Integration                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT LAYER                             │
├─────────────────────────────────────────────────────────────────┤
│  Multi-Platform Deployment Options                            │
│  ├── Azure Container Apps                                     │
│  ├── Docker Containers                                        │
│  ├── Local Development                                        │
│  └── Self-Hosted Solutions                                    │
└─────────────────────────────────────────────────────────────────┘
```

## Application Architecture

### Frontend Architecture (Client Layer)

#### Component Structure
```
app/
├── components/           # Reusable Vue components
│   ├── ApiResponse.vue      # API response handling
│   ├── BreakdownComponent.vue   # Metrics breakdown visualization
│   ├── CopilotChatViewer.vue   # Chat metrics display
│   ├── MainComponent.vue       # Main dashboard component
│   ├── MetricsViewer.vue       # Primary metrics visualization
│   └── SeatsAnalysisViewer.vue # Seat utilization analysis
├── layouts/             # Application layouts
│   └── default.vue         # Main application layout
├── pages/              # File-based routing
│   └── index.vue          # Home page
├── types/              # TypeScript definitions
│   ├── auth.d.ts          # Authentication types
│   └── metricsApiResponse.d.ts # API response types
└── model/              # Data models and business logic
    ├── Breakdown.ts       # Breakdown data model
    ├── Copilot_Metrics.ts # Copilot metrics model
    ├── Metrics.ts         # General metrics model
    ├── MetricsToUsageConverter.ts # Data transformation
    ├── MetricsValidator.ts # Data validation
    └── Seat.ts           # Seat data model
```

#### Technology Stack Integration
- **Vue.js 3 Composition API**: Modern reactive framework with TypeScript support
- **Nuxt.js 3**: Meta-framework providing SSR, file-based routing, and auto-imports
- **Vuetify 3**: Material Design component library with responsive grid system
- **Chart.js + Vue-ChartJS**: Data visualization with reactive chart components
- **TypeScript**: Static type checking across the entire application

### Backend Architecture (Application Layer)

#### Server Structure
```
server/
├── api/                # RESTful API endpoints
│   ├── health.ts          # Health check endpoint
│   ├── live.ts           # Liveness probe
│   ├── metrics.ts        # Core metrics API
│   ├── ready.ts          # Readiness probe
│   └── seats.ts          # Seat management API
├── middleware/         # Server middleware
├── modules/           # Server modules
├── plugins/           # Server plugins
└── routes/            # Additional routing
```

#### API Design Patterns
- **RESTful Architecture**: Consistent HTTP methods and status codes
- **Error Handling**: Centralized error management with proper HTTP responses
- **Validation**: Input validation and type checking
- **Configuration**: Runtime configuration management

## Data Flow Architecture

### Authentication Flow

#### GitHub OAuth Flow
```
1. User Access → Frontend Application
2. Redirect → GitHub OAuth (if NUXT_PUBLIC_USING_GITHUB_AUTH=true)
3. Authorization Code → GitHub
4. Access Token Exchange → Application
5. Session Management → nuxt-auth-utils
6. API Calls → GitHub APIs with OAuth token
```

#### Personal Access Token Flow
```
1. User Access → Frontend Application
2. Direct API Calls → GitHub APIs with PAT
3. Token Validation → GitHub
4. Metrics Retrieval → Application
```

### Data Processing Flow

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   GitHub API    │    │  Server API     │    │   Frontend      │
│                 │    │                 │    │                 │
│ Metrics Data    │──▶ │ Data Processing │──▶ │ Visualization   │
│ Seat Data       │    │ Validation      │    │ User Interface  │
│ Usage Stats     │    │ Transformation  │    │ Charts/Tables   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Scope-Based Data Retrieval

The application supports three different scopes for data retrieval:

#### Enterprise Scope
- **API Endpoint**: `https://api.github.com/enterprises/{enterprise}/copilot/metrics`
- **Configuration**: `NUXT_PUBLIC_SCOPE=enterprise`
- **Use Case**: Organization-wide metrics across multiple GitHub organizations

#### Organization Scope
- **API Endpoint**: `https://api.github.com/orgs/{organization}/copilot/metrics`
- **Configuration**: `NUXT_PUBLIC_SCOPE=organization`
- **Use Case**: Single organization metrics

#### Team Scope
- **API Endpoint**: `https://api.github.com/orgs/{organization}/team/{team}/copilot/metrics`
- **Configuration**: `NUXT_PUBLIC_SCOPE=team`
- **Use Case**: Specific team within an organization

## Deployment Architecture

### Container Strategy

#### Multi-Stage Docker Build
```dockerfile
# Stage 1: Build Stage (Node.js 24 Alpine)
- npm ci (clean install)
- npm run build (Nuxt.js build)
- Output: .output directory

# Stage 2: Production Stage (Node.js 23 Alpine)
- Copy built application
- Configure environment variables
- Expose port 80
- Non-root user execution

# Stage 3: Testing Stage (Playwright)
- Include E2E testing capabilities
- Development dependencies
- Testing configuration
```

#### Environment Variable Mapping
The Dockerfile includes backward compatibility mapping:
```bash
VUE_APP_* → NUXT_PUBLIC_*     # Frontend variables
SESSION_SECRET → NUXT_SESSION_PASSWORD  # Session management
GITHUB_CLIENT_* → NUXT_OAUTH_GITHUB_*   # OAuth configuration
```

### Azure Container Apps Deployment

#### Infrastructure as Code (Bicep)
```
infra/
├── main.bicep              # Main infrastructure template
├── main.parameters.json    # Environment-specific parameters
├── modules/               # Reusable Bicep modules
└── shared/               # Shared resources
```

#### Azure Resources
- **Container App**: Hosts the Nuxt.js application
- **Container Registry**: Stores Docker images
- **Log Analytics**: Application monitoring and logging
- **Key Vault**: Secure secret management
- **Managed Identity**: Azure service authentication

#### Deployment Options

##### Option 1: GitHub OAuth Authentication
```bash
# Required Environment Variables
NUXT_OAUTH_GITHUB_CLIENT_ID=<client-id>
NUXT_OAUTH_GITHUB_CLIENT_SECRET=<client-secret>
NUXT_PUBLIC_USING_GITHUB_AUTH=true
```

##### Option 2: Personal Access Token
```bash
# Required Environment Variables
NUXT_GITHUB_TOKEN=<github-pat>
NUXT_PUBLIC_USING_GITHUB_AUTH=false
```

##### Option 3: Mock Data (Development/Demo)
```bash
# Required Environment Variables
NUXT_PUBLIC_IS_DATA_MOCKED=true
```

### Local Development Architecture

#### Development Server
- **Hot Module Replacement**: Live code reloading
- **TypeScript Compilation**: Real-time type checking
- **ESLint Integration**: Code quality enforcement
- **Vuetify Development**: Component inspection

#### Testing Architecture
```
Testing Strategy:
├── Unit Tests (Vitest)
│   ├── Component testing with Vue Test Utils
│   ├── Model/utility function testing
│   └── API endpoint testing
├── E2E Tests (Playwright)
│   ├── Cross-browser testing
│   ├── User workflow validation
│   └── Visual regression testing
└── Linting (ESLint)
    ├── Code style enforcement
    ├── TypeScript error detection
    └── Vue.js best practices
```

## Configuration Management

### Runtime Configuration Hierarchy

#### Public Configuration (Client-side accessible)
```typescript
public: {
  isDataMocked: boolean,      // Mock data usage
  scope: string,              // API scope (enterprise/org/team)
  githubOrg: string,          // GitHub organization
  githubEnt: string,          // GitHub enterprise
  githubTeam: string,         // GitHub team
  usingGithubAuth: boolean,   // Authentication method
  version: string,            // Application version
  isPublicApp: boolean        // Public deployment flag
}
```

#### Private Configuration (Server-side only)
```typescript
private: {
  githubToken: string,        // GitHub PAT
  session: {
    maxAge: number,           // Session duration (6 hours)
    password: string          // Session encryption key
  },
  oauth: {
    github: {
      clientId: string,       // OAuth client ID
      clientSecret: string    // OAuth client secret
    }
  }
}
```

### Environment-Specific Configuration

#### Development Environment
```bash
# .env.development
NUXT_PUBLIC_IS_DATA_MOCKED=true
NUXT_PUBLIC_SCOPE=organization
NUXT_PUBLIC_USING_GITHUB_AUTH=false
```

#### Production Environment
```bash
# Azure Container Apps
NUXT_PUBLIC_IS_DATA_MOCKED=false
NUXT_PUBLIC_SCOPE=enterprise
NUXT_PUBLIC_GITHUB_ENT=compunnel
NUXT_GITHUB_TOKEN=<encrypted>
NUXT_SESSION_PASSWORD=<encrypted>
```

## Security Architecture

### Authentication Security
- **OAuth 2.0 Flow**: Secure GitHub OAuth implementation
- **Session Management**: Encrypted session cookies with 6-hour expiration
- **Token Storage**: Server-side token storage (no client exposure)
- **CSRF Protection**: Built-in Nuxt.js CSRF protection

### API Security
- **Rate Limiting**: GitHub API rate limit handling
- **Input Validation**: TypeScript-based input validation
- **Error Handling**: Secure error responses (no sensitive data exposure)
- **HTTPS Enforcement**: TLS encryption for all communications

### Container Security
- **Non-root User**: Container runs as non-privileged user (node:1000)
- **Minimal Base Image**: Alpine Linux for reduced attack surface
- **Dependency Scanning**: npm audit integration
- **Secret Management**: Environment-based secret injection

## Performance Architecture

### Frontend Performance
- **Server-Side Rendering**: Improved initial page load
- **Static Generation**: Pre-built pages where applicable
- **Code Splitting**: Automatic route-based code splitting
- **Tree Shaking**: Unused code elimination
- **Font Optimization**: Nuxt Fonts module for performance

### Backend Performance
- **Nitro Engine**: High-performance server engine
- **API Caching**: Intelligent response caching
- **Compression**: Built-in gzip/brotli compression
- **Connection Pooling**: Efficient HTTP connection management

### Monitoring and Observability

#### Health Checks
- **Liveness Probe**: `/api/live` - Container health
- **Readiness Probe**: `/api/ready` - Application readiness
- **Health Check**: `/api/health` - Detailed system status

#### Logging Architecture
```
Application Logs:
├── Request/Response Logging
├── Error Tracking
├── Performance Metrics
├── Security Events
└── GitHub API Interaction Logs
```

## Scalability Considerations

### Horizontal Scaling
- **Stateless Design**: No server-side state storage
- **Session Management**: External session storage capability
- **Load Balancing**: Azure Container Apps automatic scaling
- **CDN Integration**: Static asset delivery optimization

### Vertical Scaling
- **Resource Configuration**: Configurable CPU/memory limits
- **Database Considerations**: Future database integration points
- **Cache Strategies**: Redis integration capability

## Future Architecture Enhancements

### Planned Improvements
1. **Database Integration**: Persistent storage for historical metrics
2. **Real-time Updates**: WebSocket integration for live data
3. **Advanced Analytics**: Machine learning integration
4. **Multi-tenant Support**: Organization isolation
5. **API Rate Limiting**: Enhanced rate limiting strategies

### Extension Points
- **Plugin Architecture**: Modular plugin system
- **Custom Visualizations**: Extensible chart components
- **Export Capabilities**: PDF/Excel export functionality
- **Notification System**: Alert and notification framework

---

## Deployment Quick Reference

### Azure Container Apps Deployment
```bash
# Using Azure Developer CLI
azd auth login
azd up

# Using Docker
docker build -t copilot-metrics .
docker run -p 3000:80 copilot-metrics
```

### Local Development
```bash
# Install dependencies
npm install

# Development server
npm run dev

# Build for production
npm run build

# Run tests
npm run test
npm run test:e2e
```

### Environment Configuration Checklist
- [ ] GitHub scope configuration (enterprise/org/team)
- [ ] Authentication method (OAuth vs PAT)
- [ ] GitHub organization/enterprise/team names
- [ ] Session security configuration
- [ ] SSL/TLS certificate setup
- [ ] Health check endpoints configuration
- [ ] Logging and monitoring setup

---
*Last updated: July 8, 2025*
