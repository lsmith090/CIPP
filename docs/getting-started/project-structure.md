# Project Structure

This document provides a comprehensive overview of the CIPP frontend project structure, explaining the purpose and organization of each directory and key files.

## Root Directory Structure

```
CIPP/
├── docs/                    # Documentation (this directory)
├── src/                     # Source code
├── public/                  # Static assets
├── deployment/              # Deployment templates
├── Tools/                   # Development scripts
├── github_assets/           # GitHub-specific assets
├── next.config.js           # Next.js configuration
├── package.json             # Dependencies and scripts
├── staticwebapp.config.json # Azure SWA configuration
└── README.md                # Project overview
```

## Source Code Structure (`src/`)

### Core Application Directories

```
src/
├── api/                     # API integration utilities
├── components/              # Reusable UI components
├── contexts/                # React context providers
├── data/                    # Static data and configurations
├── hooks/                   # Custom React hooks
├── icons/                   # Icon components
├── layouts/                 # Page layout components
├── libs/                    # Third-party library configurations
├── pages/                   # Next.js pages (file-based routing)
├── sections/                # Page-specific component sections
├── store/                   # Redux store configuration
├── theme/                   # Material-UI theme customization
├── utils/                   # Utility functions
├── paths.js                 # Route path constants
└── index.js                 # Application entry point
```

## Detailed Directory Breakdown

### API Layer (`src/api/`)

```
api/
└── ApiCall.jsx              # Core API utilities and hooks
```

**Purpose**: Centralized API integration with standardized patterns for GET/POST operations, caching, and error handling.

**Key Components**:
- `ApiGetCall` - Hook for GET requests with TanStack Query
- `ApiPostCall` - Hook for POST requests with mutations
- `ApiGetCallWithPagination` - Paginated data fetching

### Component Library (`src/components/`)

The component library is organized by function and usage patterns:

```
components/
├── CippCards/               # Dashboard and information cards
├── CippComponents/          # General utility components
├── CippFormPages/           # Form-based page components
├── CippIntegrations/        # Integration-specific components
├── CippSettings/            # Settings and configuration components
├── CippStandards/           # Standards and compliance components
├── CippTable/               # Data table components
├── CippWizard/              # Multi-step wizard components
└── [utility-components]     # Individual utility components
```

#### CippCards Components
```
CippCards/
├── CippBannerListCard.jsx   # Banner-style list display
├── CippButtonCard.jsx       # Interactive button cards
├── CippChartCard.jsx        # Chart visualization cards
├── CippInfoBar.jsx          # Information status bars
├── CippInfoCard.jsx         # General information display
├── CippPropertyListCard.jsx # Property list display
└── CippUserInfoCard.jsx     # User-specific information
```

**Usage**: Dashboard components for displaying metrics, status, and interactive elements.

#### CippTable Components
```
CippTable/
├── CippDataTable.js         # Main data table component
├── CIPPTableToptoolbar.js   # Table toolbar with actions
├── CippDataTableButton.jsx  # Table action buttons
├── util-columnsFromAPI.js   # Dynamic column generation
├── util-handleActionsList.js # Action handling utilities
└── util-tablemode.js        # Table mode configurations
```

**Usage**: Comprehensive data table solution with pagination, filtering, sorting, and actions.

#### CippFormPages Components
```
CippFormPages/
├── CippFormPage.jsx         # Base form page wrapper
├── CippFormSection.jsx      # Form section grouping
├── CippFormSkeleton.jsx     # Loading state skeleton
├── CippAddEditUser.jsx      # User management forms
├── CippAddGroupForm.jsx     # Group creation forms
└── [specific-forms]         # Domain-specific form components
```

**Usage**: Standardized form pages with validation, error handling, and submission workflows.

#### CippWizard Components
```
CippWizard/
├── CippWizard.jsx           # Base wizard container
├── CippWizardPage.jsx       # Individual wizard steps
├── CippWizardStepButtons.jsx # Navigation controls
├── CippWizardConfirmation.jsx # Final confirmation step
└── [specific-wizards]       # Domain-specific wizard flows
```

**Usage**: Multi-step processes with validation, progress tracking, and state management.

### Page Structure (`src/pages/`)

Next.js file-based routing with domain-organized pages:

```
pages/
├── _app.js                  # App wrapper and providers
├── _document.js             # HTML document structure
├── index.js                 # Dashboard homepage
├── cipp/                    # CIPP administration
│   ├── settings/            # Application settings
│   ├── logs/                # Activity logs
│   └── scheduler/           # Task scheduling
├── email/                   # Email management
│   ├── administration/      # Email admin features
│   ├── reports/             # Email reports
│   ├── spamfilter/          # Spam filtering
│   ├── tools/               # Email tools
│   └── transport/           # Transport rules
├── endpoint/                # Endpoint management
│   ├── MEM/                 # Microsoft Endpoint Manager
│   ├── applications/        # Application management
│   └── autopilot/           # Autopilot deployment
├── identity/                # Identity management
│   ├── administration/      # User/group admin
│   └── reports/             # Identity reports
├── security/                # Security features
│   ├── defender/            # Microsoft Defender
│   ├── incidents/           # Security incidents
│   └── safelinks/           # Safe Links policies
├── teams-share/             # Teams and SharePoint
│   ├── onedrive/            # OneDrive management
│   ├── sharepoint/          # SharePoint sites
│   └── teams/               # Teams management
├── tenant/                  # Tenant management
│   ├── administration/      # Tenant admin features
│   ├── conditional/         # Conditional access
│   ├── gdap-management/     # GDAP relationships
│   ├── reports/             # Tenant reports
│   └── standards/           # Compliance standards
└── tools/                   # Utility tools
    ├── breachlookup/        # Security breach lookup
    └── templatelib/         # Template library
```

### Data Layer (`src/data/`)

Static configuration and reference data:

```
data/
├── AuditLogSchema.json      # Audit log structure definitions
├── AuditLogTemplates.json   # Audit log display templates
├── Extensions.json          # Available extensions
├── GDAPRoles.json          # GDAP role definitions
├── M365Licenses.json       # Microsoft 365 license types
├── alerts.json             # Alert configurations
├── cipp-roles.json         # CIPP role definitions
├── countryList.json        # Country/region data
├── languageList.json       # Supported languages
├── portals.json            # Microsoft portal links
├── standards.json          # Compliance standards
└── timezoneList.json       # Timezone definitions
```

### Hooks Directory (`src/hooks/`)

Custom React hooks for common functionality:

```
hooks/
├── use-auth.js              # Authentication state and methods
├── use-dialog.js            # Dialog state management
├── use-filters.js           # Table filtering logic
├── use-permissions.js       # Permission checking
├── use-settings.js          # Application settings
├── use-selection.js         # Multi-select functionality
└── [utility-hooks]         # Additional utility hooks
```

### Layouts (`src/layouts/`)

Page layout components and navigation:

```
layouts/
├── HeaderedTabbedLayout.jsx # Layout with header and tabs
├── TabbedLayout.jsx         # Simple tabbed layout
├── index.js                 # Main layout wrapper
├── side-nav.js              # Sidebar navigation
├── top-nav.js               # Top navigation bar
├── mobile-nav.js            # Mobile navigation
└── [layout-components]      # Navigation-related components
```

### Store Configuration (`src/store/`)

Redux store setup and configuration:

```
store/
├── index.js                 # Store configuration and providers
├── root-reducer.js          # Combined reducers
└── toasts.js               # Toast notification state
```

### Theme System (`src/theme/`)

Material-UI theme customization:

```
theme/
├── index.js                 # Theme provider and configuration
├── colors.js                # Color palette definitions
├── base/                    # Base theme components
│   ├── create-components.js # Component style overrides
│   ├── create-options.js    # Theme options
│   └── create-typography.js # Typography settings
├── light/                   # Light theme variations
│   ├── create-palette.js    # Light color palette
│   ├── create-shadows.js    # Light shadow definitions
│   └── create-components.js # Light-specific overrides
└── dark/                    # Dark theme variations
    ├── create-palette.js    # Dark color palette
    ├── create-shadows.js    # Dark shadow definitions
    └── create-components.js # Dark-specific overrides
```

### Utilities (`src/utils/`)

Helper functions and utilities:

```
utils/
├── apply-filters.js         # Data filtering logic
├── apply-pagination.js      # Pagination utilities
├── apply-sort.js            # Sorting algorithms
├── get-cipp-error.js        # Error handling utilities
├── get-cipp-formatting.js   # Data formatting helpers
├── get-cipp-translation.js  # Localization utilities
├── get-cipp-validator.js    # Validation helpers
├── permissions.js           # Permission checking logic
└── [utility-functions]      # Additional helpers
```

## Configuration Files

### Next.js Configuration (`next.config.js`)

```javascript
// Key configurations:
// - Static export for Azure SWA
// - Environment variable handling
// - Build optimizations
// - Webpack customizations
```

### Azure SWA Configuration (`staticwebapp.config.json`)

```json
{
  "routes": [
    // Route definitions for Azure Static Web Apps
    // Authentication requirements
    // API proxy configurations
  ],
  "auth": {
    // Azure AD authentication settings
  }
}
```

### Package Configuration (`package.json`)

Key sections:
- **dependencies**: Production dependencies
- **devDependencies**: Development tools
- **scripts**: Build and development commands
- **engines**: Node.js version requirements

## File Naming Conventions

### Components
- **React Components**: PascalCase (e.g., `CippDataTable.jsx`)
- **Utility Components**: camelCase (e.g., `property-list.js`)
- **Hook Files**: kebab-case with `use-` prefix (e.g., `use-auth.js`)

### Pages
- **Next.js Pages**: kebab-case (e.g., `list-users.js`)
- **Dynamic Routes**: Square brackets (e.g., `[id].js`)
- **Nested Routes**: Directory structure matches URL structure

### Utilities and Helpers
- **Utility Functions**: kebab-case (e.g., `apply-filters.js`)
- **Configuration Files**: kebab-case (e.g., `next.config.js`)

## Import Patterns

### Relative Imports
```javascript
// Within same directory
import { Component } from './Component';

// From parent directory
import { utility } from '../utils/utility';

// From src root
import { ApiCall } from '../../api/ApiCall';
```

### Absolute Imports
```javascript
// From src root (if configured)
import { ApiCall } from 'src/api/ApiCall';

// Third-party libraries
import { Box, Card } from '@mui/material';
import { useQuery } from '@tanstack/react-query';
```

## Directory Usage Guidelines

### When to Create New Directories

1. **Component Categories**: When you have 3+ related components
2. **Feature Modules**: For complete feature implementations
3. **Utility Groups**: For related helper functions
4. **Page Sections**: For complex page hierarchies

### Component Organization Principles

1. **Functional Grouping**: Group by what components do, not how they're built
2. **Reusability**: Shared components in `/components`, page-specific in `/sections`
3. **Size Consideration**: Large components get their own directories
4. **Domain Separation**: Business logic components separated by domain

This structure provides a scalable foundation for the CIPP frontend, enabling efficient development and maintenance while maintaining clear separation of concerns.