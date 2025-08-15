# CIPP Frontend Documentation

Welcome to the comprehensive documentation for the CIPP (CyberDrain Improved Partner Portal) frontend application. This documentation is designed to help both human developers and AI agents understand the codebase architecture, patterns, and development practices.

## Table of Contents

### Getting Started
- [Architecture Overview](./getting-started/overview.md) - High-level architecture and technology stack
- [Development Setup](./getting-started/development-setup.md) - Environment setup and getting started
- [Project Structure](./getting-started/project-structure.md) - Directory organization and file patterns

### Architecture & Patterns
- [Component Architecture](./architecture/components.md) - Component library organization and patterns
- [Page Patterns](./architecture/page-patterns.md) - Common page types and their implementations
- [State Management](./architecture/state-management.md) - Comprehensive Redux, React Query, and Context patterns
- [Routing & Navigation](./architecture/routing.md) - Next.js routing and navigation patterns
- [Authentication & Authorization](./architecture/auth.md) - Azure AD integration, RBAC system, and permission components

### Components Documentation
- [Component Library](./components/README.md) - Overview of the component library
- [CippCards](./components/cipp-cards/README.md) - Dashboard card components
- [CippTable](./components/cipp-table/README.md) - Data table components and utilities
- [CippFormPages](./components/cipp-forms/README.md) - Form page components and patterns
- [CippWizard](./components/cipp-wizard/README.md) - Multi-step wizard components
- [CippComponents](./components/cipp-components/README.md) - General utility components

### Development Guides
- [Adding New Pages](./guides/adding-pages.md) - How to create new pages
- [Creating Components](./guides/creating-components.md) - Component development guidelines
- [Working with APIs](./guides/api-integration.md) - API integration patterns
- [Styling Guidelines](./guides/styling.md) - Material-UI theming and component styling
- [Testing Patterns](./guides/testing.md) - Testing strategies and examples
- [Accessibility](./guides/accessibility.md) - WCAG 2.1 compliance and accessibility patterns
- [Naming Conventions](./guides/naming-conventions.md) - Code naming standards and patterns
- [Performance Optimization](./guides/performance.md) - React performance patterns and optimization

### API Reference
- [API Utilities](./api/api-utilities.md) - API call utilities and helpers
- [Hooks Reference](./api/hooks.md) - Custom React hooks
- [Core Utilities](./api/core-utilities.md) - Utility functions and helpers

### Deployment & Configuration
- [Build & Deployment](./deployment/build.md) - Build process and deployment
- [Environment Configuration](./deployment/environment.md) - Environment variables and configuration
- [Azure Static Web Apps](./deployment/azure-swa.md) - Azure SWA specific configuration

## Key Architecture Highlights

### Technology Stack
- **Framework**: Next.js 15+ with React 19
- **UI Library**: Material-UI (MUI) v6+ with custom theming
- **State Management**: Redux Toolkit with Redux Persist
- **Data Fetching**: TanStack Query (React Query) v5 with localStorage persistence
- **Forms**: React Hook Form with Yup validation
- **Routing**: Next.js file-based routing
- **Authentication**: Azure AD via Static Web Apps authentication

### Component Organization
The application uses a structured component library organized by function:

- **CippCards** - Dashboard and information display cards
- **CippTable** - Data tables with advanced filtering and actions
- **CippFormPages** - Form-based page components
- **CippWizard** - Multi-step workflow components
- **CippComponents** - General utility and shared components

### Page Patterns
CIPP follows consistent page patterns:

1. **Table Pages** - Use `CippTablePage` for data listing with actions
2. **Form Pages** - Use `CippFormPage` for add/edit operations
3. **Wizard Pages** - Use `CippWizard` for multi-step processes
4. **Dashboard Pages** - Custom layouts with cards and charts

### Tenant-Aware Architecture
The application is built for Microsoft 365 partner management with:
- Multi-tenant support with tenant selection
- Role-based permissions system
- Azure AD integration with GDAP (Granular Delegated Admin Privileges)
- Tenant-specific data isolation

## Contributing

When contributing to the CIPP frontend:

1. Follow the established component patterns
2. Use TypeScript for type safety where applicable
3. Implement proper error handling and loading states
4. Follow Material-UI theming conventions
5. Write comprehensive tests for new components
6. Update documentation for new features

## Quick Links

- [Component Library Overview](./components/README.md)
- [Page Pattern Examples](./architecture/page-patterns.md)
- [Development Setup Guide](./getting-started/development-setup.md)
- [API Integration Guide](./guides/api-integration.md)

For questions or contributions, please refer to the main CIPP repository at https://github.com/KelvinTegelaar/CIPP.