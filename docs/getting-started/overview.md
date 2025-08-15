# Architecture Overview

This document provides a comprehensive overview of the CIPP frontend architecture, technology choices, and key design patterns.

## Application Overview

CIPP (CyberDrain Improved Partner Portal) is a sophisticated Microsoft 365 partner management platform built with modern web technologies. The frontend serves as a comprehensive dashboard for managing multiple Microsoft 365 tenants, providing tools for administration, monitoring, and automation.

## Technology Stack

### Core Framework
- **Next.js 15+** - React-based framework with file-based routing, static site generation, and API routes
- **React 19** - Component-based UI library with latest features including concurrent rendering
- **JavaScript/JSX** - ES6+ JavaScript with JSX for React components

### UI and Styling
- **Material-UI (MUI) v6+** - Comprehensive React component library
  - Custom theming with light/dark mode support
  - Responsive design system
  - Accessibility-first components
- **Emotion** - CSS-in-JS styling solution for component styling
- **ApexCharts** - Interactive charts and data visualizations

### State Management
- **Redux Toolkit** - Predictable state container with modern Redux patterns
- **Redux Persist** - State persistence across browser sessions
- **TanStack Query (React Query) v5** - Server state management with caching
  - localStorage persistence for offline support
  - Automatic background refetching
  - Optimistic updates

### Forms and Validation
- **React Hook Form** - Modern form library (preferred for new development)
- **Yup** - Schema-based form validation
- **Formik** - Primary form library for complex forms (legacy)

### Data Fetching and API
- **Axios** - HTTP client for API requests
- **Custom API utilities** - Standardized API call patterns
- **SWR-like patterns** - Via TanStack Query for data synchronization

### Development and Build Tools
- **ESLint** - Code linting and quality enforcement
- **Webpack** (via Next.js) - Module bundling and optimization
- **Babel** (via Next.js) - JavaScript transpilation

## Architecture Patterns

### Component-Based Architecture

The application follows a modular component architecture with clear separation of concerns:

```
src/
├── components/           # Reusable UI components
│   ├── CippCards/       # Dashboard and info cards
│   ├── CippTable/       # Data table components
│   ├── CippFormPages/   # Form-based page components
│   ├── CippWizard/      # Multi-step workflow components
│   └── CippComponents/  # General utility components
├── pages/               # Next.js pages and routes
├── hooks/               # Custom React hooks
├── utils/               # Utility functions
├── store/               # Redux store configuration
└── theme/               # Material-UI theme customization
```

### Page Patterns

CIPP implements consistent page patterns for different use cases:

1. **Table Pages (CippTablePage)**
   - Data listing with pagination, filtering, and sorting
   - Bulk actions and individual row actions
   - Export capabilities (CSV, PDF)
   - Real-time data updates

2. **Form Pages (CippFormPage)**
   - Add/edit operations with validation
   - Conditional field rendering
   - Auto-save and draft support
   - Success/error feedback

3. **Wizard Pages (CippWizard)**
   - Multi-step processes with validation
   - Progress tracking and navigation
   - Step completion validation
   - Summary and confirmation steps

4. **Dashboard Pages**
   - Card-based layouts with metrics
   - Interactive charts and graphs
   - Real-time status indicators
   - Customizable layouts

### State Management Strategy

The application uses a hybrid state management approach:

#### Global State (Redux)
- User authentication and profile
- Tenant selection and context
- Application settings and preferences
- UI state (theme, sidebar state)

#### Server State (TanStack Query)
- API data caching and synchronization
- Background refetching and updates
- Optimistic mutations
- Error handling and retry logic

#### Local State (React useState/useReducer)
- Component-specific UI state
- Form state (via React Hook Form)
- Temporary interactions and animations

### Authentication and Authorization

#### Azure AD Integration
- **Azure Static Web Apps Authentication** - Seamless Azure AD integration
- **JWT Token Management** - Automatic token refresh and validation
- **Role-Based Access Control (RBAC)** - Granular permission system

#### Multi-Tenant Support
- **Tenant Context** - Global tenant selection and switching
- **GDAP (Granular Delegated Admin Privileges)** - Partner-tenant relationship management
- **Tenant Isolation** - Data segregation and security

### API Integration Patterns

#### Standardized API Calls
```javascript
// GET requests with caching
const { data, isLoading, error } = ApiGetCall({
  url: '/api/listUsers',
  queryKey: ['users', tenantId],
  data: { tenantFilter: tenantId }
});

// POST requests with optimistic updates
const mutation = ApiPostCall({
  relatedQueryKeys: ['users'],
  onSuccess: () => showSuccess('User created successfully')
});
```

#### Error Handling
- Centralized error handling and user feedback
- Automatic retry mechanisms for transient failures
- Graceful degradation for offline scenarios

### Performance Optimization

#### Code Splitting
- Page-level code splitting via Next.js dynamic imports
- Component-level lazy loading for heavy components
- Bundle analysis and optimization

#### Data Loading
- Prefetching for anticipated user actions
- Background data updates without UI blocking
- Pagination and virtualization for large datasets

#### Caching Strategy
- Memory caching via TanStack Query
- LocalStorage persistence for offline support
- Strategic cache invalidation

## Security Considerations

### Data Protection
- Client-side input validation and sanitization
- HTTPS enforcement for all communications
- Sensitive data encryption in transit and at rest

### Authentication Security
- Token-based authentication with short expiration
- Automatic logout on token expiration
- Secure token storage practices

### Permission System
- Role-based access control at component level
- API-level permission validation
- Tenant-specific permission isolation

## Scalability and Maintainability

### Code Organization
- Modular component architecture
- Clear separation of concerns
- Consistent naming conventions
- Comprehensive type definitions

### Testing Strategy
- Component unit testing with React Testing Library
- Integration testing for critical user flows
- End-to-end testing for core functionality

### Documentation
- Component documentation with examples
- API documentation with TypeScript types
- Architecture decision records (ADRs)

## Development Workflow

### Local Development
1. **Development Server** - Hot reloading with Next.js dev server
2. **API Mocking** - Local API development with Azure Functions emulation
3. **State Debugging** - Redux DevTools and React Query DevTools
4. **Type Checking** - Real-time TypeScript validation

### Build and Deployment
1. **Static Generation** - Next.js static export for Azure Static Web Apps
2. **Bundle Optimization** - Automatic code splitting and minification
3. **Asset Optimization** - Image optimization and lazy loading
4. **Progressive Web App** - Service worker for offline functionality

## Integration Points

### Backend Integration (CIPP-API)
- PowerShell-based Azure Functions API
- Microsoft Graph API integration
- Azure resource management
- Automated workflow orchestration

### External Services
- **Microsoft 365 Services** - Graph API, Exchange Online, SharePoint
- **Security Services** - Azure Security Center, Defender
- **Monitoring Services** - Application Insights, Azure Monitor
- **Third-party Integrations** - PSA tools, RMM platforms

This architecture provides a robust, scalable, and maintainable foundation for the CIPP frontend application, enabling efficient management of Microsoft 365 environments at scale.