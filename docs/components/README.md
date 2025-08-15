# CIPP Component Library

The CIPP component library provides a comprehensive set of reusable React components built on Material-UI v6+ for building Microsoft 365 partner management interfaces. All components are tenant-aware, permission-controlled, and follow consistent design patterns.

## Component Categories

### 1. [CippCards](./cipp-cards/README.md)
Data display cards for presenting information, metrics, and interactive content.

- **CippInfoCard**: Simple metric display with optional action link
- **CippPropertyListCard**: Key-value property lists with actions
- **CippChartCard**: ApexCharts visualization with Material-UI theming
- **CippInfoBar**: Horizontal metrics display with off-canvas details
- **CippBannerListCard**: Collapsible banner with grouped information
- **CippButtonCard**: Interactive card with primary action button
- **CippImageCard**: Image content with overlays
- **CippRemediationCard**: Business Email Compromise security remediation

#### Additional Card Components

- **CippDomainCards**: Domain analysis and security information display
- **CippExchangeInfoCard**: Exchange-specific information and configuration
- **CippUserInfoCard**: User-specific information with contextual actions
- **CippListitemCard**: Individual list item component with actions
- **CippPageCard**: Page-level card component for consistent layouts

### 2. [CippTable](./cipp-table/README.md)
Advanced data table system with filtering, sorting, and actions.

- **CippDataTable**: Main table component with Material React Table
- **CippTablePage**: Pre-configured page layout with table
- **CippDataTableButton**: Custom table action buttons

### 3. [CippFormPages](./cipp-forms/README.md)
Form components with React Hook Form integration.

- **CippFormPage**: Complete form page layout with validation
- **CippFormSection**: Collapsible form sections
- **CippFormComponent**: Form field components
- **CippFormTenantSelector**: Tenant selection in forms
- **CippFormUserSelector**: User selection components

### 4. [CippWizard](./cipp-wizard/README.md)
Multi-step wizard system for complex workflows.

- **CippWizard**: Main wizard container with step management
- **CippWizardPage**: Individual wizard page wrapper
- **CippWizardStepButtons**: Navigation controls
- **CippWizardConfirmation**: Final confirmation step

### 5. [CippComponents](./cipp-components/README.md)
Utility components for common functionality.

- **CippTablePage**: Complete table page template
- **CippOffCanvas**: Slide-out panel for details
- **CippApiDialog**: API action confirmation dialogs
- **CippTenantSelector**: Global tenant selection
- **CippApiResults**: API response display
- **CippHead**: Page head metadata management

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

### Dashboard Pattern
```jsx
import { Grid, Container } from '@mui/material';
import { CippInfoBar, CippChartCard, CippPropertyListCard } from '../../src/components/CippCards';

const Dashboard = () => (
  <Container maxWidth="xl">
    <Grid container spacing={3}>
      <Grid item xs={12}>
        <CippInfoBar items={metrics} />
      </Grid>
      <Grid item xs={12} md={6}>
        <CippChartCard title="Usage Chart" data={chartData} />
      </Grid>
      <Grid item xs={12} md={6}>
        <CippPropertyListCard title="Details" propertyItems={properties} />
      </Grid>
    </Grid>
  </Container>
);
```

### Table Pattern
```jsx
import { CippTablePage } from '../../src/components/CippComponents';

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
import { CippFormPage } from '../../src/components/CippFormPages';
import { useForm } from 'react-hook-form';

const AddUser = () => {
  const formControl = useForm();
  
  return (
    <CippFormPage
      title="Add User"
      formControl={formControl}
      postUrl="/api/AddUser"
      queryKey={['users']}
    >
      {/* Form fields */}
    </CippFormPage>
  );
};
```

### Wizard Pattern
```jsx
import { CippWizard } from '../../src/components/CippWizard';

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
  <Grid item xs={12} md={6} lg={4}>
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
import { ApiGetCall } from '../../src/api/ApiCall';

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
import { useSettings } from '../../src/hooks/use-settings';

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