# Page Patterns

This document outlines the standardized page patterns used throughout the CIPP frontend application. These patterns ensure consistency, maintainability, and a predictable user experience.

## Overview

CIPP implements four primary page patterns, each optimized for specific use cases:

1. **Table Pages** - Data listing and management
2. **Form Pages** - Data entry and editing
3. **Wizard Pages** - Multi-step processes
4. **Dashboard Pages** - Metrics and overview displays

## 1. Table Page Pattern

### Purpose
Table pages are used for displaying, filtering, and managing collections of data. They provide comprehensive data management capabilities including pagination, sorting, filtering, bulk actions, and exports.

### Implementation

#### Basic Table Page
```jsx
import { CippTablePage } from '../components/CippComponents/CippTablePage';

function UsersPage() {
  return (
    <CippTablePage
      title="Users"
      api={{
        url: '/api/listUsers',
        data: { tenantFilter: 'AllTenants' }
      }}
      columns={[
        { accessorKey: 'displayName', header: 'Display Name' },
        { accessorKey: 'userPrincipalName', header: 'Email' },
        { accessorKey: 'accountEnabled', header: 'Status' }
      ]}
      actions={[
        {
          label: 'Edit User',
          icon: <EditIcon />,
          action: (row) => router.push(`/identity/administration/users/edit?userId=${row.id}`)
        },
        {
          label: 'Reset Password',
          icon: <LockResetIcon />,
          action: (row) => handlePasswordReset(row)
        }
      ]}
    />
  );
}
```

#### Advanced Table Features
```jsx
function AdvancedUsersPage() {
  const [selectedTenant, setSelectedTenant] = useState('');
  
  return (
    <CippTablePage
      title="User Management"
      subtitle="Manage users across all tenants"
      
      // API configuration
      api={{
        url: '/api/listUsers',
        data: { 
          tenantFilter: selectedTenant,
          includeApplications: true 
        }
      }}
      queryKey={['users', selectedTenant]}
      
      // Column configuration
      columns={[
        { accessorKey: 'displayName', header: 'Name', size: 200 },
        { accessorKey: 'userPrincipalName', header: 'Email', size: 250 },
        { 
          accessorKey: 'accountEnabled', 
          header: 'Status',
          cell: ({ row }) => (
            <Chip 
              label={row.original.accountEnabled ? 'Enabled' : 'Disabled'}
              color={row.original.accountEnabled ? 'success' : 'error'}
            />
          )
        },
        {
          accessorKey: 'lastSignInDateTime',
          header: 'Last Sign In',
          cell: ({ row }) => <CippTimeAgo date={row.original.lastSignInDateTime} />
        }
      ]}
      
      // Actions configuration
      actions={[
        {
          label: 'Edit User',
          icon: <EditIcon />,
          action: (row) => handleEditUser(row.original),
          permission: 'users.edit'
        },
        {
          label: 'Disable User',
          icon: <BlockIcon />,
          action: (row) => handleDisableUser(row.original),
          condition: (row) => row.original.accountEnabled,
          confirmation: true
        }
      ]}
      
      // Bulk actions
      bulkActions={[
        {
          label: 'Enable Selected',
          icon: <CheckIcon />,
          action: (selectedRows) => handleBulkEnable(selectedRows)
        },
        {
          label: 'Export Selected',
          icon: <ExportIcon />,
          action: (selectedRows) => handleBulkExport(selectedRows)
        }
      ]}
      
      // Additional features
      exportEnabled={true}
      filterPresets={[
        { label: 'Active Users', filters: [{ id: 'accountEnabled', value: true }] },
        { label: 'Inactive Users', filters: [{ id: 'accountEnabled', value: false }] }
      ]}
      
      // Custom toolbar
      cardButton={
        <Button 
          variant="contained" 
          startIcon={<AddIcon />}
          onClick={() => router.push('/identity/administration/users/add')}
        >
          Add User
        </Button>
      }
      
      // Page-specific filters
      topToolbar={
        <CippTenantSelector
          value={selectedTenant}
          onChange={setSelectedTenant}
          allTenants={true}
        />
      }
    />
  );
}
```

### Table Page Structure
```
┌─────────────────────────────────────────┐
│ Page Title                   [Add Button]│
│ Subtitle (optional)                     │
├─────────────────────────────────────────┤
│ [Tenant Selector] [Filters] [Export]    │
├─────────────────────────────────────────┤
│ ┌─────────────────────────────────────┐ │
│ │ Data Table                          │ │
│ │ ├─ Search/Filter bar               │ │
│ │ ├─ Column headers (sortable)       │ │
│ │ ├─ Data rows with actions          │ │
│ │ └─ Pagination controls             │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

## 2. Form Page Pattern

### Purpose
Form pages handle data entry and editing operations. They provide validation, error handling, and standardized submission workflows.

### Implementation

#### Basic Form Page
```jsx
import { CippFormPage } from '../components/CippFormPages/CippFormPage';
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';

const schema = yup.object({
  displayName: yup.string().required('Display name is required'),
  userPrincipalName: yup.string().email('Invalid email').required('Email is required'),
  accountEnabled: yup.boolean()
});

function AddUserPage() {
  const formControl = useForm({
    resolver: yupResolver(schema),
    defaultValues: {
      displayName: '',
      userPrincipalName: '',
      accountEnabled: true,
      passwordProfile: {
        forceChangePasswordNextSignIn: true
      }
    }
  });

  return (
    <CippFormPage
      title="Add User"
      backButtonTitle="Back to Users"
      formPageType="Add"
      queryKey={['users']}
      formControl={formControl}
      postUrl="/api/addUser"
      customDataformatter={(data) => ({
        ...data,
        userPrincipalName: `${data.userPrincipalName}@${selectedTenant}`,
        tenantId: selectedTenant
      })}
    >
      <CippFormSection title="Basic Information">
        <CippFormComponent
          type="textField"
          label="Display Name"
          name="displayName"
          validators={{ required: "Display name is required" }}
          formControl={formControl}
        />
        
        <CippFormComponent
          type="textField"
          label="Username"
          name="userPrincipalName"
          validators={{ required: "Username is required" }}
          formControl={formControl}
          helperText="Enter username without domain"
        />
        
        <CippFormComponent
          type="switch"
          label="Account Enabled"
          name="accountEnabled"
          formControl={formControl}
        />
      </CippFormSection>

      <CippFormSection title="Password Settings">
        <CippFormComponent
          type="switch"
          label="Force Password Change on First Login"
          name="passwordProfile.forceChangePasswordNextSignIn"
          formControl={formControl}
        />
      </CippFormSection>

      <CippFormSection title="License Assignment">
        <CippFormLicenseSelector
          name="assignedLicenses"
          label="Assign Licenses"
          formControl={formControl}
          tenantId={selectedTenant}
          multiple={true}
        />
      </CippFormSection>
    </CippFormPage>
  );
}
```

#### Edit Form Page
```jsx
function EditUserPage() {
  const router = useRouter();
  const { userId } = router.query;
  
  // Fetch existing user data
  const { data: userData, isLoading } = ApiGetCall({
    url: '/api/getUser',
    data: { userId },
    queryKey: ['user', userId]
  });

  const formControl = useForm({
    mode: 'onChange',
    defaultValues: userData || {}
  });

  // Reset form when data loads
  useEffect(() => {
    if (userData) {
      formControl.reset(userData);
    }
  }, [userData, formControl]);

  if (isLoading) return <CippFormSkeleton />;

  return (
    <CippFormPage
      title="Edit User"
      formPageType="Edit"
      queryKey={['users', 'user', userId]}
      formControl={formControl}
      postUrl="/api/editUser"
      customDataformatter={(data) => ({
        ...data,
        userId: userId
      })}
    >
      {/* Form sections same as Add form */}
    </CippFormPage>
  );
}
```

### Form Page Structure
```
┌─────────────────────────────────────────┐
│ [← Back] Page Title                     │
│ Form Type (Add/Edit)                    │
├─────────────────────────────────────────┤
│ ┌─────────────────────────────────────┐ │
│ │ Section Title                       │ │
│ │ ├─ Form Field 1                    │ │
│ │ ├─ Form Field 2                    │ │
│ │ └─ Form Field 3                    │ │
│ └─────────────────────────────────────┘ │
│ ┌─────────────────────────────────────┐ │
│ │ Section Title 2                     │ │
│ │ ├─ Form Field 4                    │ │
│ │ └─ Form Field 5                    │ │
│ └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│              [Cancel] [Submit]          │
└─────────────────────────────────────────┘
```

## 3. Wizard Page Pattern

### Purpose
Wizard pages guide users through complex multi-step processes, breaking them into manageable steps with validation and progress tracking.

### Implementation

#### Basic Wizard
```jsx
import { CippWizard } from '../components/CippWizard/CippWizard';

function UserOnboardingWizard() {
  const [wizardData, setWizardData] = useState({});
  
  const steps = [
    {
      title: 'Basic Information',
      component: <BasicInfoStep />,
      validation: basicInfoSchema
    },
    {
      title: 'License Assignment',
      component: <LicenseStep />,
      validation: licenseSchema
    },
    {
      title: 'Group Membership',
      component: <GroupStep />,
      validation: groupSchema
    },
    {
      title: 'Review & Confirm',
      component: <ConfirmationStep />
    }
  ];

  const handleWizardComplete = async (data) => {
    try {
      await ApiPostCall({
        url: '/api/onboardUser',
        data: data
      });
      
      showSuccess('User onboarded successfully');
      router.push('/identity/administration/users');
    } catch (error) {
      showError('Failed to onboard user');
    }
  };

  return (
    <CippWizard
      title="User Onboarding Wizard"
      subtitle="Create and configure a new user account"
      steps={steps}
      data={wizardData}
      onDataChange={setWizardData}
      onComplete={handleWizardComplete}
      onCancel={() => router.back()}
    />
  );
}
```

#### Wizard Step Implementation
```jsx
function BasicInfoStep({ data, onDataChange, onNext, onPrevious }) {
  const formControl = useForm({
    mode: 'onChange',
    defaultValues: data.basicInfo || {}
  });

  const handleNext = () => {
    if (formControl.formState.isValid) {
      onDataChange({
        ...data,
        basicInfo: formControl.getValues()
      });
      onNext();
    }
  };

  return (
    <CippWizardPage
      title="Basic Information"
      subtitle="Enter the user's basic information"
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
      
      <CippFormComponent
        type="textField"
        label="Email"
        name="userPrincipalName"
        required={true}
        formControl={formControl}
      />

      <CippWizardStepButtons
        onNext={handleNext}
        onPrevious={onPrevious}
        nextDisabled={!formControl.formState.isValid}
        showPrevious={false} // First step
      />
    </CippWizardPage>
  );
}
```

### Wizard Structure
```
┌─────────────────────────────────────────┐
│ Wizard Title                            │
│ Subtitle                                │
├─────────────────────────────────────────┤
│ Step 1 ●━━○━━○━━○ Step 4               │
├─────────────────────────────────────────┤
│ ┌─────────────────────────────────────┐ │
│ │ Step Title                          │ │
│ │ Step Subtitle                       │ │
│ │                                     │ │
│ │ Step Content                        │ │
│ │ (Form fields, instructions, etc.)   │ │
│ │                                     │ │
│ └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│         [Previous] [Cancel] [Next]      │
└─────────────────────────────────────────┘
```

## 4. Dashboard Page Pattern

### Purpose
Dashboard pages provide overview information, metrics, and quick access to key functionality through cards, charts, and summary widgets.

### Implementation

#### Basic Dashboard
```jsx
import { Grid } from '@mui/material';
import { CippInfoBar, CippChartCard, CippPropertyListCard } from '../components/CippCards';

function TenantDashboard() {
  const { data: tenantStats } = ApiGetCall({
    url: '/api/getTenantStats',
    queryKey: ['tenantStats']
  });

  const { data: userActivity } = ApiGetCall({
    url: '/api/getUserActivity',
    queryKey: ['userActivity']
  });

  return (
    <Box sx={{ flexGrow: 1, py: 4 }}>
      <Container maxWidth="xl">
        <Typography variant="h4" gutterBottom>
          Dashboard
        </Typography>
        
        <Grid container spacing={3}>
          {/* Key Metrics */}
          <Grid item xs={12} sm={6} md={3}>
            <CippInfoBar
              title="Total Users"
              value={tenantStats?.totalUsers || 0}
              color="primary"
              icon={<PeopleIcon />}
            />
          </Grid>
          
          <Grid item xs={12} sm={6} md={3}>
            <CippInfoBar
              title="Active Licenses"
              value={tenantStats?.activeLicenses || 0}
              color="success"
              icon={<LicenseIcon />}
            />
          </Grid>
          
          <Grid item xs={12} sm={6} md={3}>
            <CippInfoBar
              title="Security Score"
              value={`${tenantStats?.securityScore || 0}%`}
              color="warning"
              icon={<SecurityIcon />}
            />
          </Grid>
          
          <Grid item xs={12} sm={6} md={3}>
            <CippInfoBar
              title="Active Alerts"
              value={tenantStats?.activeAlerts || 0}
              color="error"
              icon={<AlertIcon />}
            />
          </Grid>

          {/* Charts */}
          <Grid item xs={12} md={8}>
            <CippChartCard
              title="User Activity (Last 30 Days)"
              chartType="line"
              data={userActivity}
              height="400px"
            />
          </Grid>

          {/* Quick Stats */}
          <Grid item xs={12} md={4}>
            <CippPropertyListCard
              title="Quick Stats"
              properties={[
                { label: 'Sign-ins Today', value: tenantStats?.signinsToday },
                { label: 'Failed Sign-ins', value: tenantStats?.failedSignins },
                { label: 'New Users (7d)', value: tenantStats?.newUsers7d },
                { label: 'License Utilization', value: `${tenantStats?.licenseUtilization}%` }
              ]}
            />
          </Grid>

          {/* Recent Activity */}
          <Grid item xs={12}>
            <CippRecentActivityCard
              title="Recent Activity"
              activities={tenantStats?.recentActivities}
              maxItems={10}
            />
          </Grid>
        </Grid>
      </Container>
    </Box>
  );
}
```

#### Advanced Dashboard with Interactive Elements
```jsx
function AdvancedDashboard() {
  const [dateRange, setDateRange] = useState('30d');
  const [selectedTenant, setSelectedTenant] = useState('');

  return (
    <Box sx={{ flexGrow: 1, py: 4 }}>
      <Container maxWidth="xl">
        {/* Dashboard Header */}
        <Box display="flex" justifyContent="space-between" alignItems="center" mb={3}>
          <Typography variant="h4">
            Tenant Overview
          </Typography>
          
          <Box display="flex" gap={2}>
            <CippTenantSelector
              value={selectedTenant}
              onChange={setSelectedTenant}
              allTenants={true}
            />
            
            <Select
              value={dateRange}
              onChange={(e) => setDateRange(e.target.value)}
              size="small"
            >
              <MenuItem value="7d">Last 7 days</MenuItem>
              <MenuItem value="30d">Last 30 days</MenuItem>
              <MenuItem value="90d">Last 90 days</MenuItem>
            </Select>
          </Box>
        </Box>

        {/* Responsive Grid Layout */}
        <Grid container spacing={3}>
          {/* Dashboard content */}
        </Grid>
      </Container>
    </Box>
  );
}
```

### Dashboard Structure
```
┌─────────────────────────────────────────┐
│ Dashboard Title    [Filters] [Controls] │
├─────────────────────────────────────────┤
│ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐        │
│ │Info │ │Info │ │Info │ │Info │        │
│ │Card │ │Card │ │Card │ │Card │        │
│ └─────┘ └─────┘ └─────┘ └─────┘        │
├─────────────────────────────────────────┤
│ ┌─────────────────┐ ┌─────────────────┐ │
│ │                 │ │                 │ │
│ │   Chart Card    │ │ Property List   │ │
│ │                 │ │    Card         │ │
│ └─────────────────┘ └─────────────────┘ │
├─────────────────────────────────────────┤
│ ┌─────────────────────────────────────┐ │
│ │         Recent Activity             │ │
│ │         Table/List                  │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

## Pattern Selection Guidelines

### When to Use Each Pattern

#### Table Pages
- **Use for**: Data listing, bulk operations, reporting
- **Examples**: User lists, license reports, audit logs
- **Key features**: Pagination, filtering, sorting, export

#### Form Pages  
- **Use for**: Data entry, editing, configuration
- **Examples**: Add user, edit settings, create policy
- **Key features**: Validation, error handling, auto-save

#### Wizard Pages
- **Use for**: Complex multi-step processes
- **Examples**: User onboarding, tenant setup, policy creation
- **Key features**: Step validation, progress tracking, data persistence

#### Dashboard Pages
- **Use for**: Overview information, metrics, quick access
- **Examples**: Tenant dashboard, security overview, reports summary
- **Key features**: Real-time data, interactive charts, responsive layout

## Common Implementation Patterns

### Error Handling
```jsx
// Consistent error handling across all page types
const { data, isLoading, error } = ApiGetCall({
  url: '/api/getData',
  queryKey: ['data']
});

if (error) {
  return <CippErrorPage error={error} />;
}

if (isLoading) {
  return <CippLoadingPage />;
}
```

### Permission Checking
```jsx
// Role-based page access
function ProtectedPage() {
  const { hasPermission } = usePermissions();
  
  if (!hasPermission('users.view')) {
    return <CippUnauthorized />;
  }
  
  return <PageContent />;
}
```

### Responsive Design
```jsx
// Mobile-responsive layouts
<Grid container spacing={{ xs: 2, md: 3 }}>
  <Grid item xs={12} sm={6} md={3}>
    <CippInfoCard />
  </Grid>
</Grid>
```

These patterns provide a solid foundation for building consistent, maintainable pages throughout the CIPP application.