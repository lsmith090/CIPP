# CIPP Component Library

The CIPP component library provides a comprehensive set of reusable React components built on Material-UI v6+ for building Microsoft 365 partner management interfaces. All components are tenant-aware, permission-controlled, and follow consistent design patterns.

> **Architecture Overview**: For high-level architectural patterns, design principles, and integration guidelines, see the [Component Architecture](../architecture/components.md) documentation.

## Quick Start Guide

1. **Choose Component Category**: Identify your use case and select the appropriate component category
2. **Review Component Documentation**: Read the detailed README for your chosen category
3. **Check Examples**: Review real-world examples in the `examples.md` files
4. **Follow Import Patterns**: Use absolute import paths with `.jsx` extensions
5. **Implement with Props**: Follow the component-specific prop interfaces and validation

### Navigation

- **[Architecture Patterns](../architecture/components.md)** - Design principles and architectural guidance
- **[CippCards](./cipp-cards/README.md)** - Information display and visualization components ([Examples](./cipp-cards/examples.md))
- **[CippTable](./cipp-table/README.md)** - Data management and table components ([Examples](./cipp-table/examples.md))
- **[CippForms](./cipp-forms/README.md)** - Form and input components ([Examples](./cipp-forms/examples.md))
- **[CippWizard](./cipp-wizard/README.md)** - Multi-step workflow components ([Examples](./cipp-wizard/examples.md))
- **[CippComponents](./cipp-components/README.md)** - Utility and helper components

## Component Categories

The component library is organized into five main categories, each serving specific use cases in Microsoft 365 partner management interfaces:

### 1. [CippCards](./cipp-cards/README.md) - Information Display
**Purpose**: Dashboard and data visualization components for presenting metrics, status, and interactive content.

**Core Components**:
- **CippInfoCard**: Single metric display with optional action links
- **CippPropertyListCard**: Key-value property lists with conditional actions
- **CippChartCard**: ApexCharts integration with Material-UI theming
- **CippInfoBar**: Horizontal metrics bar with expandable details
- **CippBannerListCard**: Collapsible grouped information display
- **CippButtonCard**: Action-focused interactive cards
- **CippImageCard**: Image content with overlay functionality
- **CippRemediationCard**: Security remediation workflow cards

**Common Use Cases**: Dashboards, status displays, metric visualization, quick actions

#### Additional Card Components

- **CippDomainCards**: Domain analysis and security information display
- **CippExchangeInfoCard**: Exchange-specific information and configuration
- **CippUserInfoCard**: User-specific information with contextual actions
- **CippListitemCard**: Individual list item component with actions
- **CippPageCard**: Page-level card component for consistent layouts

### 2. [CippTable](./cipp-table/README.md) - Data Management
**Purpose**: Advanced data table system for managing large datasets with comprehensive filtering, sorting, and action capabilities.

**Core Components**:
- **CippDataTable**: Full-featured table with Material React Table integration
- **CippTablePage**: Pre-configured page layout with integrated table
- **CippDataTableButton**: Custom action buttons for table toolbars

**Key Features**: Sorting, filtering, pagination, bulk actions, export, virtualization, off-canvas details
**Common Use Cases**: User management, license reporting, audit logs, data analysis

### 3. [CippFormPages](./cipp-forms/README.md) - Form Management
**Purpose**: Comprehensive form components with React Hook Form integration, providing validation, submission handling, and consistent styling.

**Core Components**:
- **CippFormPage**: Complete form page with validation and submission workflow
- **CippFormSection**: Collapsible sections for organizing complex forms
- **CippFormComponent**: Universal field component supporting all input types
- **CippFormTenantSelector**: Multi-tenant selection for forms
- **CippFormUserSelector**: User and group selection with search

**Key Features**: Validation, error handling, tenant-aware operations, conditional fields, file uploads
**Common Use Cases**: User creation, configuration forms, bulk operations, settings management

### 4. [CippWizard](./cipp-wizard/README.md) - Multi-Step Workflows
**Purpose**: Guided multi-step processes with validation, conditional logic, and progress tracking for complex operations.

**Core Components**:
- **CippWizard**: Main wizard container with step management and state persistence
- **CippWizardPage**: Individual wizard page wrapper with navigation
- **CippWizardStepButtons**: Navigation controls with validation checks
- **CippWizardConfirmation**: Final review and confirmation step

**Key Features**: Step validation, conditional navigation, progress tracking, state persistence, responsive design
**Common Use Cases**: User onboarding, complex configurations, bulk operations, guided setups

### 5. [CippComponents](./cipp-components/README.md) - Utility Components
**Purpose**: Cross-cutting utility components providing common functionality across the application.

**Core Components**:
- **CippTablePage**: Complete table page template with standard layout
- **CippOffCanvas**: Slide-out detail panels with action integration
- **CippApiDialog**: Confirmation dialogs with form inputs and API integration
- **CippTenantSelector**: Global tenant selection with context management
- **CippApiResults**: API response display with success/error handling
- **CippCodeBlock**: Syntax highlighting and code editing capabilities

**Key Features**: Dialog management, tenant context, API integration, layout consistency
**Common Use Cases**: Page layouts, data operations, tenant switching, code display

## Key Features

### Material-UI v6+ Integration
All components use Material-UI components and theming system:
- Consistent design tokens and spacing
- Dark/light mode support
- Responsive breakpoints
- Custom theme extensions

### Tenant Awareness
Components automatically handle multi-tenant scenarios:
- Current tenant context from settings
- Tenant filtering in API calls
- Tenant-specific data display
- Permission-based access control

### Permission Control
Role-based access control throughout:
- Component-level permission checks
- Action-level restrictions
- Conditional rendering based on roles
- Integration with CIPP permission system

### API Integration
Seamless backend integration:
- React Query for data fetching
- Automatic caching and invalidation
- Error handling and retry logic
- Loading states and skeletons

## Design Patterns

> **Architectural Patterns**: For comprehensive information on architectural design patterns, composition strategies, and system integration, see the [Component Architecture documentation](../architecture/components.md#architectural-design-patterns).

### Dashboard Pattern
```jsx
import { Grid, Container } from '@mui/system';
import { CippInfoBar } from '/src/components/CippCards/CippInfoBar.jsx';
import { CippChartCard } from '/src/components/CippCards/CippChartCard.jsx';
import { CippPropertyListCard } from '/src/components/CippCards/CippPropertyListCard.jsx';

const Dashboard = () => (
  <Container maxWidth="xl">
    <Grid container spacing={3}>
      <Grid size={{ xs: 12 }}>
        <CippInfoBar items={metrics} />
      </Grid>
      <Grid size={{ xs: 12, md: 6 }}>
        <CippChartCard title="Usage Chart" chartSeries={chartData} />
      </Grid>
      <Grid size={{ xs: 12, md: 6 }}>
        <CippPropertyListCard title="Details" propertyItems={properties} />
      </Grid>
    </Grid>
  </Container>
);
```

### Table Pattern
```jsx
import { CippTablePage } from '/src/components/CippComponents/CippTablePage.jsx';

const UsersTable = () => (
  <CippTablePage
    title="Users"
    apiUrl="/api/ListUsers"
    actions={userActions}
    offCanvas={{ extendedInfoFields: userFields }}
    queryKey="users"
  />
);
```

### Form Pattern
```jsx
import { CippFormPage } from '/src/components/CippFormPages/CippFormPage.jsx';
import { CippFormComponent } from '/src/components/CippComponents/CippFormComponent.jsx';
import { useForm } from 'react-hook-form';

const AddUser = () => {
  const formControl = useForm({
    mode: 'onChange',
    defaultValues: {
      displayName: '',
      email: ''
    }
  });
  
  return (
    <CippFormPage
      title="Add User"
      formControl={formControl}
      postUrl="/api/AddUser"
      queryKey={['users']}
    >
      <CippFormComponent
        type="textField"
        name="displayName"
        label="Display Name"
        formControl={formControl}
        validators={{ required: "Display Name is required" }}
      />
      <CippFormComponent
        type="textField"
        name="email"
        label="Email"
        formControl={formControl}
        validators={{
          required: "Email is required",
          pattern: {
            value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
            message: "Invalid email format"
          }
        }}
      />
    </CippFormPage>
  );
};
```

### Wizard Pattern
```jsx
import { CippWizard } from '/src/components/CippWizard/CippWizard.jsx';

const SetupWizard = () => (
  <CippWizard
    steps={wizardSteps}
    postUrl="/api/CompleteSetup"
    initialState={defaultValues}
  />
);
```

## Styling Guidelines

### Theme Integration
```jsx
import { useTheme } from '@mui/material/styles';

const MyComponent = () => {
  const theme = useTheme();
  
  return (
    <Box sx={{
      backgroundColor: theme.palette.background.paper,
      color: theme.palette.text.primary,
      padding: theme.spacing(2),
    }}>
      Content
    </Box>
  );
};
```

### Responsive Design
```jsx
// Use Material-UI Grid system
<Grid container spacing={3}>
  <Grid size={{ xs: 12, md: 6, lg: 4 }}>
    <CippInfoCard />
  </Grid>
</Grid>

// Use sx prop for responsive styling
<Box sx={{
  display: { xs: 'block', md: 'flex' },
  flexDirection: { md: 'row', lg: 'column' }
}}>
```

### Color Usage
```jsx
// Use semantic color tokens
sx={{
  color: 'primary.main',
  backgroundColor: 'background.paper',
  borderColor: 'divider',
}}

// For status indicators
sx={{
  color: 'success.main', // Green for success
  color: 'error.main',   // Red for errors
  color: 'warning.main', // Orange for warnings
  color: 'info.main',    // Blue for information
}}
```

## Data Fetching Patterns

### React Query Integration
```jsx
import { ApiGetCall } from '/src/api/ApiCall.jsx';

const MyComponent = () => {
  const { data, isLoading, error } = ApiGetCall({
    url: '/api/ListData',
    queryKey: 'my-data',
  });

  if (isLoading) return <Skeleton />;
  if (error) return <Alert severity="error">{error.message}</Alert>;
  
  return <DataDisplay data={data} />;
};
```

### Tenant-Aware Queries
```jsx
import { useSettings } from '/src/hooks/use-settings.js';

const TenantDataComponent = () => {
  const { currentTenant } = useSettings();
  
  const { data } = ApiGetCall({
    url: '/api/ListTenantData',
    data: { tenantFilter: currentTenant },
    queryKey: ['tenant-data', currentTenant],
  });
};
```

## Best Practices

1. **Component Composition**: Build complex UIs by composing smaller components
2. **Props Interface**: Use TypeScript-like prop validation with PropTypes
3. **Error Boundaries**: Wrap components in error boundaries for graceful failures
4. **Accessibility**: Include proper ARIA labels and keyboard navigation
5. **Performance**: Use React.memo() for expensive re-renders
6. **Testing**: Write unit tests for component logic and integration tests for workflows

## Getting Started

1. Import the required components from their respective directories
2. Ensure Material-UI theme is properly configured
3. Set up React Query provider for data fetching
4. Configure tenant context provider
5. Implement permission checking hooks

For detailed examples and API documentation, see the individual component category documentation.