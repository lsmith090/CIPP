# Component Architecture

This document outlines the CIPP frontend component architecture, design patterns, and usage guidelines for the component library.

## Component Library Overview

The CIPP component library is organized into functional categories, each serving specific use cases within the application. This modular approach promotes reusability, consistency, and maintainability.

## Component Categories

### 1. CippCards - Dashboard and Information Display

Dashboard components for displaying metrics, status information, and interactive elements.

#### Core Card Components

**CippInfoBar**
```jsx
import { CippInfoBar } from '../components/CippCards/CippInfoBar';

// Usage: Status information with optional actions
<CippInfoBar
  title="Tenant Status"
  value="Healthy"
  color="success"
  icon={<CheckCircleIcon />}
  action={<Button>Details</Button>}
/>
```

**CippChartCard**
```jsx
import { CippChartCard } from '../components/CippCards/CippChartCard';

// Usage: Data visualization with charts
<CippChartCard
  title="User Growth"
  chartType="line"
  data={chartData}
  height="300px"
  showLegend={true}
/>
```

**CippPropertyListCard**
```jsx
import { CippPropertyListCard } from '../components/CippCards/CippPropertyListCard';

// Usage: Key-value property display
<CippPropertyListCard
  title="Tenant Information"
  properties={[
    { label: 'Domain', value: 'contoso.com' },
    { label: 'Users', value: '1,250' },
    { label: 'Licenses', value: '1,000' }
  ]}
/>
```

#### Dashboard Layout Pattern

```jsx
// Typical dashboard layout using Grid system
import { Grid } from '@mui/material';

function Dashboard() {
  return (
    <Grid container spacing={3}>
      <Grid item xs={12} md={4}>
        <CippInfoBar title="Active Users" value="1,250" />
      </Grid>
      <Grid item xs={12} md={4}>
        <CippInfoBar title="Licenses Used" value="1,000" />
      </Grid>
      <Grid item xs={12} md={4}>
        <CippInfoBar title="Security Score" value="85%" />
      </Grid>
      <Grid item xs={12} md={8}>
        <CippChartCard title="User Activity" data={activityData} />
      </Grid>
      <Grid item xs={12} md={4}>
        <CippPropertyListCard title="Quick Stats" properties={statsData} />
      </Grid>
    </Grid>
  );
}
```

### 2. CippTable - Data Tables and Lists

Comprehensive data table solution with advanced features for data management.

#### Core Table Component

**CippDataTable**
```jsx
import { CippDataTable } from '../components/CippTable/CippDataTable';

// Basic usage with API integration
<CippDataTable
  title="Users"
  api={{
    url: '/api/listUsers',
    data: { tenantFilter: selectedTenant }
  }}
  columns={[
    { accessorKey: 'displayName', header: 'Name' },
    { accessorKey: 'userPrincipalName', header: 'Email' },
    { accessorKey: 'accountEnabled', header: 'Status' }
  ]}
  actions={[
    {
      label: 'Edit User',
      icon: <EditIcon />,
      action: (row) => handleEditUser(row.original)
    }
  ]}
/>
```

#### Advanced Table Features

```jsx
// Table with custom formatting and bulk actions
<CippDataTable
  title="License Report"
  api={{ url: '/api/listLicenses' }}
  simpleColumns={['displayName', 'assignedLicenses', 'usageLocation']}
  filters={[
    { id: 'accountEnabled', value: true }
  ]}
  actions={[
    {
      label: 'Assign License',
      type: 'bulk',
      action: (selectedRows) => handleBulkLicenseAssignment(selectedRows)
    }
  ]}
  exportEnabled={true}
  cardButton={<Button onClick={handleAddUser}>Add User</Button>}
/>
```

#### Table Patterns

1. **List Pages**: Read-only data display with export capabilities
2. **Management Pages**: Interactive tables with row actions
3. **Report Pages**: Data analysis with filtering and grouping
4. **Selection Pages**: Multi-select for bulk operations

### 3. CippFormPages - Form Components

Standardized form components with validation, error handling, and submission workflows.

#### Core Form Component

**CippFormPage**
```jsx
import { CippFormPage } from '../components/CippFormPages/CippFormPage';
import { useForm } from 'react-hook-form';

function AddUserPage() {
  const formControl = useForm({
    defaultValues: {
      displayName: '',
      userPrincipalName: '',
      accountEnabled: true
    }
  });

  return (
    <CippFormPage
      title="Add User"
      formPageType="Add"
      queryKey={['users']}
      formControl={formControl}
      postUrl="/api/addUser"
    >
      <CippFormComponent
        type="textField"
        label="Display Name"
        name="displayName"
        required={true}
        formControl={formControl}
      />
      <CippFormComponent
        type="textField"
        label="Email Address"
        name="userPrincipalName"
        required={true}
        formControl={formControl}
      />
      <CippFormComponent
        type="switch"
        label="Account Enabled"
        name="accountEnabled"
        formControl={formControl}
      />
    </CippFormPage>
  );
}
```

#### Form Field Components

**CippFormComponent** - Universal form field component:
```jsx
// Text input
<CippFormComponent
  type="textField"
  label="Display Name"
  name="displayName"
  required={true}
  formControl={formControl}
/>

// Select dropdown
<CippFormComponent
  type="select"
  label="License Type"
  name="licenseType"
  options={licenseOptions}
  formControl={formControl}
/>

// Multi-select
<CippFormComponent
  type="autoComplete"
  label="Groups"
  name="memberOf"
  multiple={true}
  options={groupOptions}
  formControl={formControl}
/>

// Switch/Toggle
<CippFormComponent
  type="switch"
  label="Account Enabled"
  name="accountEnabled"
  formControl={formControl}
/>
```

#### Specialized Form Components

**CippFormTenantSelector**
```jsx
<CippFormTenantSelector
  name="tenantId"
  label="Select Tenant"
  formControl={formControl}
  multiple={false}
/>
```

**CippFormUserSelector**
```jsx
<CippFormUserSelector
  name="userId"
  label="Select User"
  formControl={formControl}
  tenantId={selectedTenant}
/>
```

### 4. CippWizard - Multi-Step Workflows

Components for complex multi-step processes with validation and progress tracking.

#### Core Wizard Component

**CippWizard**
```jsx
import { CippWizard } from '../components/CippWizard/CippWizard';

function OnboardingWizard() {
  const steps = [
    {
      title: 'Basic Information',
      component: <BasicInfoStep />
    },
    {
      title: 'License Assignment',
      component: <LicenseStep />
    },
    {
      title: 'Group Membership',
      component: <GroupStep />
    },
    {
      title: 'Confirmation',
      component: <ConfirmationStep />
    }
  ];

  return (
    <CippWizard
      title="User Onboarding Wizard"
      steps={steps}
      onComplete={handleWizardComplete}
      onCancel={handleWizardCancel}
    />
  );
}
```

#### Wizard Step Pattern

```jsx
// Individual wizard step component
function BasicInfoStep({ formControl, nextStep, previousStep }) {
  return (
    <CippWizardPage
      title="Basic Information"
      subtitle="Enter user details"
    >
      <CippFormComponent
        type="textField"
        label="First Name"
        name="firstName"
        required={true}
        formControl={formControl}
      />
      <CippFormComponent
        type="textField"
        label="Last Name"
        name="lastName"
        required={true}
        formControl={formControl}
      />
      <CippWizardStepButtons
        onNext={nextStep}
        onPrevious={previousStep}
        nextDisabled={!formControl.formState.isValid}
      />
    </CippWizardPage>
  );
}
```

### 5. CippComponents - Utility Components

General-purpose components used throughout the application.

#### Dialog Components

**CippApiDialog**
```jsx
import { CippApiDialog } from '../components/CippComponents/CippApiDialog';

// API operation with confirmation dialog
<CippApiDialog
  title="Delete User"
  content="Are you sure you want to delete this user?"
  api={{
    url: '/api/removeUser',
    data: { userId: selectedUser.id }
  }}
  onSuccess={() => showSuccess('User deleted successfully')}
/>
```

**CippComponentDialog**
```jsx
// Custom content dialog
<CippComponentDialog
  title="User Details"
  open={dialogOpen}
  onClose={() => setDialogOpen(false)}
  maxWidth="md"
>
  <UserDetailsContent user={selectedUser} />
</CippComponentDialog>
```

#### Selector Components

**CippTenantSelector**
```jsx
import { CippTenantSelector } from '../components/CippComponents/CippTenantSelector';

// Global tenant selection
<CippTenantSelector
  value={selectedTenant}
  onChange={handleTenantChange}
  allTenants={false}
  placeholder="Select a tenant"
/>
```

#### Utility Components

**CippCopyToClipboard**
```jsx
<CippCopyToClipboard
  value="user@contoso.com"
  showValue={true}
  successMessage="Email copied to clipboard"
/>
```

**CippTimeAgo**
```jsx
<CippTimeAgo
  date="2024-01-15T10:30:00Z"
  live={true}
  tooltip={true}
/>
```

## Component Design Patterns

### 1. Composition Pattern

Components are designed for composition, allowing flexible layouts:

```jsx
// Flexible card composition
<Card>
  <CardHeader
    title="User Statistics"
    action={<CippCopyToClipboard value={shareUrl} />}
  />
  <CardContent>
    <CippPropertyList properties={userStats} />
  </CardContent>
  <CardActions>
    <Button onClick={handleRefresh}>Refresh</Button>
    <Button onClick={handleExport}>Export</Button>
  </CardActions>
</Card>
```

### 2. Render Props Pattern

For components that need custom rendering:

```jsx
<CippDataTable
  title="Users"
  api={{ url: '/api/listUsers' }}
  columns={[
    {
      accessorKey: 'displayName',
      header: 'Name',
      cell: ({ row }) => (
        <Box sx={{ display: 'flex', alignItems: 'center' }}>
          <Avatar src={row.original.photo} />
          <Typography ml={2}>{row.original.displayName}</Typography>
        </Box>
      )
    }
  ]}
/>
```

### 3. Hook Integration Pattern

Components integrate with custom hooks for logic separation:

```jsx
function UserManagementPage() {
  const { users, isLoading, error } = useUsers();
  const { permissions } = usePermissions();
  const { showDialog } = useDialog();

  const actions = useMemo(() => [
    {
      label: 'Edit',
      action: (user) => showDialog('editUser', { user }),
      disabled: !permissions.canEditUsers
    }
  ], [permissions, showDialog]);

  return (
    <CippDataTable
      title="Users"
      data={users}
      isLoading={isLoading}
      error={error}
      actions={actions}
    />
  );
}
```

## Component Props Patterns

### Common Prop Interfaces

```typescript
// Base component props
interface BaseComponentProps {
  title?: string;
  loading?: boolean;
  error?: string | Error;
  sx?: SxProps;
  children?: ReactNode;
}

// API integration props
interface ApiProps {
  url: string;
  data?: Record<string, any>;
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
}

// Form component props
interface FormComponentProps {
  name: string;
  label: string;
  required?: boolean;
  disabled?: boolean;
  formControl: UseFormReturn;
  validation?: ValidationRules;
}
```

### Prop Validation

Components use TypeScript for prop validation and provide sensible defaults:

```jsx
function CippInfoCard({
  title,
  value,
  color = 'primary',
  icon,
  loading = false,
  error,
  ...props
}) {
  if (loading) return <Skeleton variant="rectangular" height={120} />;
  if (error) return <ErrorDisplay error={error} />;
  
  return (
    <Card {...props}>
      {/* Component implementation */}
    </Card>
  );
}
```

## Performance Considerations

### 1. Component Memoization

Large components use React.memo for performance:

```jsx
import { memo } from 'react';

export const CippDataTable = memo(function CippDataTable(props) {
  // Component implementation
}, (prevProps, nextProps) => {
  // Custom comparison for complex props
  return isEqual(prevProps.data, nextProps.data);
});
```

### 2. Lazy Loading

Heavy components are loaded on demand:

```jsx
import { lazy, Suspense } from 'react';

const CippChartCard = lazy(() => import('./CippChartCard'));

function Dashboard() {
  return (
    <Suspense fallback={<Skeleton height={300} />}>
      <CippChartCard data={chartData} />
    </Suspense>
  );
}
```

### 3. Virtual Scrolling

Large data sets use virtual scrolling:

```jsx
<CippDataTable
  data={largeDataset}
  enableVirtualization={true}
  rowHeight={60}
  overscan={10}
/>
```

## Testing Patterns

### Component Testing

```jsx
import { render, screen, fireEvent } from '@testing-library/react';
import { CippInfoCard } from './CippInfoCard';

describe('CippInfoCard', () => {
  it('renders title and value', () => {
    render(
      <CippInfoCard
        title="Test Title"
        value="Test Value"
      />
    );
    
    expect(screen.getByText('Test Title')).toBeInTheDocument();
    expect(screen.getByText('Test Value')).toBeInTheDocument();
  });

  it('handles click events', () => {
    const handleClick = jest.fn();
    render(
      <CippInfoCard
        title="Test"
        onClick={handleClick}
      />
    );
    
    fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalled();
  });
});
```

This component architecture provides a solid foundation for building consistent, maintainable, and scalable user interfaces in the CIPP application.