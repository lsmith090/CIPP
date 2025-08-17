# Page Construction Patterns

This documentation covers the page construction patterns used in CIPP (CyberDrain Improved Partner Portal). Understanding these patterns is essential for building consistent, maintainable pages and components within the application.

## Overview

CIPP uses a structured approach to page construction based on common patterns that provide consistent user experiences and developer workflows. The system is built on Next.js with Material-UI components and follows a component-driven architecture.

## Page Types

CIPP uses four primary page construction patterns:

### 1. Table Pages (`CippTablePage`)
Data-driven pages that display information in tabular format with filtering, sorting, and action capabilities.

- **Primary Component**: `CippTablePage`
- **Use Cases**: User lists, device inventories, reports, logs
- **Key Features**: Filtering, sorting, bulk actions, off-canvas details
- **Example**: `/identity/administration/users/index.js`

### 2. Form Pages (`CippFormPage`)
Form-based pages for creating and editing entities with validation and submission handling.

- **Primary Component**: `CippFormPage`
- **Use Cases**: Add/edit users, groups, policies, settings
- **Key Features**: React Hook Form integration, validation, API submission
- **Example**: `/identity/administration/users/add.jsx`

### 3. Wizard Pages (`CippWizard`)
Multi-step guided processes for complex operations requiring sequential input.

- **Primary Component**: `CippWizard`
- **Use Cases**: User onboarding, bulk operations, complex configurations
- **Key Features**: Step navigation, form state management, progress tracking
- **Example**: `/identity/administration/offboarding-wizard/index.js`

### 4. Dashboard Pages
Custom layout pages with card-based information displays and interactive elements.

- **Primary Components**: Various card components (`CippInfoBar`, `CippChartCard`, etc.)
- **Use Cases**: Main dashboard, overview pages, summary views
- **Key Features**: Responsive grid layout, interactive charts, real-time data
- **Example**: `/index.js` (main dashboard)

## Layout System

CIPP uses a hierarchical layout system that provides consistent navigation and structure:

### Main Layout (`DashboardLayout`)
The base application layout that wraps all pages:
- Side navigation with role-based menu filtering
- Top navigation with tenant selection
- Authentication and permission handling
- Responsive design for mobile/desktop

### Tabbed Layouts
For pages requiring multiple views or sections:
- **`HeaderedTabbedLayout`**: Full-featured layout with title, subtitle, breadcrumbs, and actions
- **`TabbedLayout`**: Simple tabbed interface for basic multi-section pages

## Core Patterns

### getLayout Pattern
Every page uses the `getLayout` pattern to specify its layout:

```javascript
Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
```

### API Integration Pattern
All pages use consistent API patterns:
- `ApiGetCall` for data fetching with React Query
- `ApiPostCall` for form submissions and actions
- Tenant-aware API calls with automatic tenant filtering

### Permission-Based Rendering
Pages and components use permission-based rendering:
- Navigation items filtered by user permissions
- Action buttons gated by required permissions
- Component-level permission checks

### Responsive Design
All patterns follow mobile-first responsive design:
- Grid-based layouts using Material-UI Grid v2
- Responsive breakpoints for different screen sizes
- Mobile-optimized navigation and interactions

## File Structure Conventions

### Page Files
```
/src/pages/
  /{category}/
    /{section}/
      index.js           # List/table page
      add.jsx           # Add form page
      edit.jsx          # Edit form page
      {entity}/
        index.jsx       # Detail page
        edit.jsx        # Edit page
        tabOptions.json # Tab configuration
```

### Component Organization
```
/src/components/
  CippComponents/       # Core CIPP components
  CippCards/           # Card-based components
  CippFormPages/       # Form-specific components
  CippWizard/          # Wizard components
  CippTable/           # Table components
```

## Next.js Integration

### File-Based Routing
- Automatic routing based on file structure
- Dynamic routes using `[param].js` syntax
- Nested routes for detailed entity views

### Static Generation
- Pages optimized for static generation where possible
- Dynamic imports for client-side only components
- Optimized bundle splitting

## Quick Start Guide

### Creating a New Table Page

```javascript
import { CippTablePage } from "../../components/CippComponents/CippTablePage.jsx";
import { Layout as DashboardLayout } from "/src/layouts/index.js";

const Page = () => {
  return (
    <CippTablePage
      title="My Data"
      apiUrl="/api/ListMyData"
      apiData={{ tenantFilter: "current" }}
      apiDataKey="Results"
      simpleColumns={["name", "status", "created"]}
    />
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default Page;
```

### Creating a New Form Page

```javascript
import CippFormPage from "/src/components/CippFormPages/CippFormPage";
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { useForm } from "react-hook-form";

const Page = () => {
  const formControl = useForm({
    mode: "onBlur",
    defaultValues: {}
  });

  return (
    <CippFormPage
      title="My Entity"
      formControl={formControl}
      postUrl="/api/AddMyEntity"
    >
      {/* Form fields here */}
    </CippFormPage>
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default Page;
```

## Best Practices

1. **Consistent Patterns**: Always use the established patterns for similar functionality
2. **Permission Integration**: Implement proper permission checks for all actions
3. **Responsive Design**: Ensure all pages work across device sizes
4. **API Consistency**: Use standard API patterns and error handling
5. **Loading States**: Implement proper loading and error states
6. **Accessibility**: Follow WCAG guidelines for accessible interfaces
7. **Performance**: Optimize for Core Web Vitals and loading performance

## Related Documentation

- [Table Pages](./page-types/table-pages.md) - Detailed CippTablePage documentation
- [Form Pages](./page-types/form-pages.md) - Comprehensive form patterns
- [Wizard Pages](./page-types/wizard-pages.md) - Multi-step process patterns
- [Dashboard Pages](./page-types/dashboard-pages.md) - Dashboard and card layouts
- [Layout System](./layouts/README.md) - Layout components and patterns
- [Routing](../guides/routing.md) - Next.js routing patterns and navigation

## Contributing

When adding new pages or modifying existing patterns:

1. Follow the established patterns documented here
2. Update documentation for any new patterns introduced
3. Ensure responsive design and accessibility compliance
4. Test across different user permission levels
5. Validate API integration and error handling