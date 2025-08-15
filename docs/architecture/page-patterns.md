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
import { CippTablePage } from '/src/components/CippComponents/CippTablePage.jsx';
import { Layout as DashboardLayout } from '/src/layouts/index.js';
import { EditIcon, LockResetIcon } from '@mui/icons-material';
import { CippUserActions } from '/src/components/CippComponents/CippUserActions.jsx';

function UsersPage() {
  return (
    <CippTablePage
      title="Users"
      apiUrl="/api/ListGraphRequest"
      apiData={{
        Endpoint: 'users',
        tenantFilter: 'AllTenants',
        $select: 'id,displayName,userPrincipalName,accountEnabled'
      }}
      apiDataKey="Results"
      simpleColumns={[
        'displayName',
        'userPrincipalName', 
        'accountEnabled'
      ]}
      actions={CippUserActions()}
    />
  );
}

UsersPage.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;

export default UsersPage;
```

#### Advanced Table Features
```jsx
import { CippTablePage } from '/src/components/CippComponents/CippTablePage.jsx';
import { Layout as DashboardLayout } from '/src/layouts/index.js';
import { useState } from 'react';
import { Button, Chip } from '@mui/material';
import { Add, PersonAdd, GroupAdd, Send } from '@mui/icons-material';
import Link from 'next/link';
import { useSettings } from '/src/hooks/use-settings.js';
import { PermissionButton } from '/src/utils/permissions';
import { CippUserActions } from '/src/components/CippComponents/CippUserActions.jsx';

function AdvancedUsersPage() {
  const tenant = useSettings().currentTenant;
  
  const filters = [
    {
      filterName: "Account Enabled",
      value: [{ id: "accountEnabled", value: "Yes" }],
      type: "column",
    },
    {
      filterName: "Account Disabled",
      value: [{ id: "accountEnabled", value: "No" }],
      type: "column",
    },
    {
      filterName: "Guest Accounts",
      value: [{ id: "userType", value: "Guest" }],
      type: "column",
    },
  ];

  const offCanvas = {
    extendedInfoFields: [
      "createdDateTime",
      "id",
      "userPrincipalName",
      "givenName",
      "surname",
      "jobTitle",
      "assignedLicenses",
      "businessPhones",
      "mobilePhone",
      "mail",
      "city",
      "department",
    ],
    actions: CippUserActions(),
  };
  
  return (
    <CippTablePage
      title="User Management"
      apiUrl="/api/ListGraphRequest"
      apiData={{
        Endpoint: "users",
        manualPagination: true,
        $select: "id,accountEnabled,businessPhones,city,createdDateTime,displayName,givenName,jobTitle,mail,mailNickname,mobilePhone,surname,usageLocation,userPrincipalName,userType,assignedLicenses",
        $count: true,
        $orderby: "displayName",
        $top: 999,
      }}
      apiDataKey="Results"
      actions={CippUserActions()}
      offCanvas={offCanvas}
      simpleColumns={[
        "accountEnabled",
        "userPrincipalName",
        "displayName",
        "mail",
        "businessPhones",
        "assignedLicenses",
      ]}
      filters={filters}
      cardButton={
        <>
          <PermissionButton
            requiredPermissions={["Identity.User.ReadWrite"]}
            component={Link}
            href="users/add"
            startIcon={<PersonAdd />}
          >
            Add User
          </PermissionButton>
          <PermissionButton
            requiredPermissions={["Identity.User.ReadWrite"]}
            component={Link}
            href="users/bulk-add"
            startIcon={<GroupAdd />}
          >
            Bulk Add Users
          </PermissionButton>
        </>
      }
    />
  );
}

AdvancedUsersPage.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;

export default AdvancedUsersPage;
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
import { Box } from '@mui/material';
import CippFormPage from '/src/components/CippFormPages/CippFormPage.jsx';
import { Layout as DashboardLayout } from '/src/layouts/index.js';
import { useForm, useWatch } from 'react-hook-form';
import { CippFormUserSelector } from '/src/components/CippComponents/CippFormUserSelector.jsx';
import { useSettings } from '/src/hooks/use-settings.js';
import { useEffect } from 'react';
import CippAddEditUser from '/src/components/CippFormPages/CippAddEditUser.jsx';

function AddUserPage() {
  const userSettingsDefaults = useSettings();

  const formControl = useForm({
    mode: 'onBlur',
    defaultValues: {
      tenantFilter: userSettingsDefaults.currentTenant,
      usageLocation: userSettingsDefaults.usageLocation,
    },
  });

  const formValues = useWatch({ control: formControl.control, name: 'userProperties' });
  
  useEffect(() => {
    if (formValues) {
      const { userPrincipalName, usageLocation, ...restFields } = formValues.addedFields || {};
      let newFields = { ...restFields };
      if (userPrincipalName) {
        const [mailNickname, domainNamePart] = userPrincipalName.split('@');
        if (mailNickname) {
          newFields.mailNickname = mailNickname;
        }
        if (domainNamePart) {
          newFields.primDomain = { label: domainNamePart, value: domainNamePart };
        }
      }
      if (usageLocation) {
        newFields.usageLocation = { label: usageLocation, value: usageLocation };
      }
      newFields.tenantFilter = userSettingsDefaults.currentTenant;

      formControl.reset(newFields);
    }
  }, [formValues]);

  return (
    <CippFormPage
      queryKey={`Users-${userSettingsDefaults.currentTenant}`}
      formControl={formControl}
      title="User"
      backButtonTitle="User Overview"
      postUrl="/api/AddUser"
    >
      <Box sx={{ my: 2 }}>
        <CippFormUserSelector
          formControl={formControl}
          name="userProperties"
          label="Copy properties from another user"
          multiple={false}
          select="id,userPrincipalName,displayName,givenName,surname,mailNickname,jobTitle,department,usageLocation"
          addedField={{
            groupType: "calculatedGroupType",
            displayName: "displayName",
            userPrincipalName: "userPrincipalName",
            id: "id",
            givenName: "givenName",
            surname: "surname",
            mailNickname: "mailNickname",
            jobTitle: "jobTitle",
            department: "department",
            usageLocation: "usageLocation",
          }}
        />
      </Box>
      <Box sx={{ my: 2 }}>
        <CippAddEditUser formControl={formControl} userSettingsDefaults={userSettingsDefaults} />
      </Box>
    </CippFormPage>
  );
}

AddUserPage.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;

export default AddUserPage;
```

#### Edit Form Page
```jsx
import { useRouter } from 'next/router';
import { useEffect } from 'react';
import { Box } from '@mui/material';
import CippFormPage from '/src/components/CippFormPages/CippFormPage.jsx';
import { Layout as DashboardLayout } from '/src/layouts/index.js';
import { useForm } from 'react-hook-form';
import { ApiGetCall } from '/src/api/ApiCall.jsx';
import { useSettings } from '/src/hooks/use-settings.js';
import CippAddEditUser from '/src/components/CippFormPages/CippAddEditUser.jsx';

function EditUserPage() {
  const router = useRouter();
  const { userId } = router.query;
  const userSettingsDefaults = useSettings();
  
  // Fetch existing user data
  const { data: userData, isLoading } = ApiGetCall({
    url: '/api/ListGraphRequest',
    data: { 
      Endpoint: `users/${userId}`,
      tenantFilter: userSettingsDefaults.currentTenant,
      $select: 'id,displayName,userPrincipalName,givenName,surname,jobTitle,department,usageLocation,accountEnabled'
    },
    queryKey: ['user', userId, userSettingsDefaults.currentTenant]
  });

  const formControl = useForm({
    mode: 'onChange',
    defaultValues: {
      tenantFilter: userSettingsDefaults.currentTenant
    }
  });

  // Reset form when data loads
  useEffect(() => {
    if (userData) {
      formControl.reset({
        ...userData,
        userId: userId,
        tenantFilter: userSettingsDefaults.currentTenant
      });
    }
  }, [userData, formControl, userId, userSettingsDefaults.currentTenant]);

  if (isLoading) return <div>Loading...</div>;

  return (
    <CippFormPage
      title="Edit User"
      formPageType="Edit"
      queryKey={['users', 'user', userId]}
      formControl={formControl}
      postUrl="/api/EditUser"
      customDataformatter={(data) => ({
        ...data,
        userId: userId
      })}
    >
      <Box sx={{ my: 2 }}>
        <CippAddEditUser 
          formControl={formControl} 
          userSettingsDefaults={userSettingsDefaults}
          editMode={true}
        />
      </Box>
    </CippFormPage>
  );
}

EditUserPage.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;

export default EditUserPage;
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
import { Layout as DashboardLayout } from '/src/layouts/index.js';
import { CippWizardConfirmation } from '/src/components/CippWizard/CippWizardConfirmation.jsx';
import { CippDeploymentStep } from '/src/components/CippWizard/CIPPDeploymentStep.jsx';
import CippWizardPage from '/src/components/CippWizard/CippWizardPage.jsx';
import { CippWizardOptionsList } from '/src/components/CippWizard/CippWizardOptionsList.jsx';
import { BuildingOfficeIcon, CloudIcon, CpuChipIcon } from '@heroicons/react/24/outline';

function UserOnboardingWizard() {
  const steps = [
    {
      title: 'Step 1',
      description: 'Basic Information',
      component: CippWizardOptionsList,
      componentProps: {
        title: 'Select user type',
        subtext: 'Choose the type of user account to create',
        valuesKey: 'UserType',
        options: [
          {
            description: 'Create a standard user account with basic permissions',
            icon: <CpuChipIcon />,
            label: 'Standard User',
            value: 'StandardUser',
          },
          {
            description: 'Create an administrator account with elevated permissions',
            icon: <CloudIcon />,
            label: 'Administrator',
            value: 'Admin',
          },
          {
            description: 'Create a guest account for external users',
            icon: <BuildingOfficeIcon />,
            label: 'Guest User',
            value: 'Guest',
          },
        ],
      },
    },
    {
      title: 'Step 2',
      description: 'Configuration',
      component: CippDeploymentStep,
    },
    {
      title: 'Step 3',
      description: 'Confirmation',
      component: CippWizardConfirmation,
    },
  ];

  return (
    <CippWizardPage
      backButton={false}
      steps={steps}
      wizardTitle="User Onboarding"
      postUrl="/api/AddUser"
    />
  );
}

UserOnboardingWizard.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;

export default UserOnboardingWizard;
```

#### Wizard Step Implementation
```jsx
import { useState } from 'react';
import { Box, Stack, Typography, Button } from '@mui/material';
import { useForm } from 'react-hook-form';
import { CippFormComponent } from '/src/components/CippComponents/CippFormComponent.jsx';
import { CippWizardStepButtons } from '/src/components/CippWizard/CippWizardStepButtons.jsx';

function BasicInfoStep({ onNextStep, onPreviousStep, formControl, currentStep, lastStep, title, subtext }) {
  const handleNext = () => {
    // Validate current step
    formControl.trigger().then((isValid) => {
      if (isValid) {
        onNextStep();
      }
    });
  };

  return (
    <Box>
      <Stack spacing={3}>
        <Box>
          <Typography variant="h5" gutterBottom>
            {title || 'Basic Information'}
          </Typography>
          <Typography variant="body2" color="text.secondary">
            {subtext || 'Enter the user\'s basic information'}
          </Typography>
        </Box>
        
        <Stack spacing={2}>
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
        </Stack>

        <CippWizardStepButtons
          onNext={handleNext}
          onPrevious={onPreviousStep}
          nextDisabled={false}
          showPrevious={currentStep > 0}
          nextLabel={currentStep === lastStep ? 'Finish' : 'Next'}
        />
      </Stack>
    </Box>
  );
}

export default BasicInfoStep;
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
import Head from 'next/head';
import { useEffect, useState } from 'react';
import { Box, Container, Button, Card, CardContent, Tooltip } from '@mui/material';
import { Grid } from '@mui/system';
import { CippInfoBar } from '/src/components/CippCards/CippInfoBar.jsx';
import { CippChartCard } from '/src/components/CippCards/CippChartCard.jsx';
import { CippPropertyListCard } from '/src/components/CippCards/CippPropertyListCard.jsx';
import { Layout as DashboardLayout } from '/src/layouts/index.js';
import { useSettings } from '/src/hooks/use-settings.js';
import { ApiGetCall } from '/src/api/ApiCall.jsx';

function TenantDashboard() {
  const { currentTenant } = useSettings();
  const [domainVisible, setDomainVisible] = useState(false);

  const organization = ApiGetCall({
    url: '/api/ListOrg',
    queryKey: `${currentTenant}-ListOrg`,
    data: { tenantFilter: currentTenant },
  });

  const dashboard = ApiGetCall({
    url: '/api/ListuserCounts',
    data: { tenantFilter: currentTenant },
    queryKey: `${currentTenant}-ListuserCounts`,
  });

  const sharepoint = ApiGetCall({
    url: '/api/ListSharepointQuota',
    queryKey: `${currentTenant}-ListSharepointQuota`,
    data: { tenantFilter: currentTenant },
  });

  const formatStorageSize = (sizeInMB) => {
    if (sizeInMB >= 1024) {
      return `${(sizeInMB / 1024).toFixed(2)}GB`;
    }
    return `${sizeInMB}MB`;
  };

  return (
    <>
      <Head>
        <title>Dashboard</title>
      </Head>
      <Box sx={{ flexGrow: 1, py: 4 }}>
        <Container maxWidth={false}>
          <Grid container spacing={3}>
            {/* User Statistics Chart */}
            <Grid size={{ md: 4, xs: 12 }}>
              <CippChartCard
                title="User Statistics"
                isFetching={dashboard.isFetching}
                chartType="pie"
                chartSeries={[
                  Number(dashboard.data?.LicUsers || 0),
                  Number(dashboard.data?.Users - dashboard.data?.LicUsers - dashboard.data?.Guests || 0),
                  Number(dashboard.data?.Guests || 0),
                ]}
                labels={['Licensed Users', 'Unlicensed Users', 'Guests']}
              />
            </Grid>

            {/* SharePoint Quota Chart */}
            <Grid size={{ md: 4, xs: 12 }}>
              <CippChartCard
                title="SharePoint Quota"
                isFetching={sharepoint.isFetching}
                chartType="donut"
                chartSeries={[
                  Number(sharepoint.data?.TenantStorageMB - sharepoint.data?.GeoUsedStorageMB) || 0,
                  Number(sharepoint.data?.GeoUsedStorageMB) || 0,
                ]}
                labels={[
                  `Free (${formatStorageSize(
                    sharepoint.data?.TenantStorageMB - sharepoint.data?.GeoUsedStorageMB
                  )})`,
                  `Used (${formatStorageSize(sharepoint.data?.GeoUsedStorageMB)})`,
                ]}
              />
            </Grid>

            {/* Domain Names Property List */}
            <Grid size={{ md: 4, xs: 12 }}>
              <CippPropertyListCard
                title="Domain Names"
                showDivider={false}
                copyItems={true}
                isFetching={organization.isFetching}
                propertyItems={organization.data?.verifiedDomains
                  ?.slice(0, domainVisible ? undefined : 3)
                  .map((domain, idx) => ({
                    label: '',
                    value: domain.name,
                  }))}
                actionButton={
                  organization.data?.verifiedDomains?.length > 3 && (
                    <Button onClick={() => setDomainVisible(!domainVisible)}>
                      {domainVisible ? 'See less' : 'See more...'}
                    </Button>
                  )
                }
              />
            </Grid>
          </Grid>
        </Container>
      </Box>
    </>
  );
}

TenantDashboard.getLayout = (page) => <DashboardLayout allTenantsSupport={false}>{page}</DashboardLayout>;

export default TenantDashboard;
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

### API Call Patterns
```jsx
import { ApiGetCall, ApiPostCall } from '/src/api/ApiCall.jsx';
import { useSettings } from '/src/hooks/use-settings.js';

// Basic GET request
const { data, isLoading, error } = ApiGetCall({
  url: '/api/ListGraphRequest',
  data: { 
    tenantFilter: currentTenant,
    Endpoint: 'users',
    $select: 'id,displayName,userPrincipalName,accountEnabled'
  },
  queryKey: ['users', currentTenant]
});

// GET request with pagination
const { data: paginatedData } = ApiGetCall({
  url: '/api/ListGraphRequest',
  data: {
    tenantFilter: currentTenant,
    Endpoint: 'users',
    manualPagination: true,
    $count: true,
    $orderby: 'displayName',
    $top: 999
  },
  queryKey: ['users-paginated', currentTenant]
});

// POST request with mutation
const postCall = ApiPostCall({
  datafromUrl: true,
  relatedQueryKeys: ['users', currentTenant]
});

// Submit data
const handleSubmit = () => {
  postCall.mutate({
    url: '/api/AddUser',
    data: formData
  });
};

// Query key patterns:
// - Use tenant in key: ['users', currentTenant]
// - Include identifying data: ['user', userId, currentTenant]
// - Use descriptive prefixes: ['ListGraphRequest', endpoint, currentTenant]
```

### Error Handling
```jsx
import { ApiGetCall } from '/src/api/ApiCall.jsx';
import { CippErrorPage } from '/src/components/CippComponents/CippErrorPage.jsx';
import { CippLoadingPage } from '/src/components/CippComponents/CippLoadingPage.jsx';

// Consistent error handling across all page types
const { data, isLoading, error } = ApiGetCall({
  url: '/api/ListGraphRequest',
  data: { tenantFilter: currentTenant, Endpoint: 'users' },
  queryKey: ['users', currentTenant]
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
import { PermissionButton } from '/src/utils/permissions.js';
import { usePermissions } from '/src/hooks/use-permissions.js';
import { CippUnauthorized } from '/src/components/CippComponents/CippUnauthorized.jsx';

// Role-based page access
function ProtectedPage() {
  const { hasPermission } = usePermissions();
  
  if (!hasPermission('Identity.User.Read')) {
    return <CippUnauthorized />;
  }
  
  return <PageContent />;
}

// Permission-based button rendering
function PageWithPermissionButton() {
  return (
    <PermissionButton
      requiredPermissions={['Identity.User.ReadWrite']}
      component={Link}
      href="/identity/administration/users/add"
      startIcon={<PersonAddIcon />}
    >
      Add User
    </PermissionButton>
  );
}
```

### Responsive Design
```jsx
import { Grid } from '@mui/system';
import { CippInfoBar } from '/src/components/CippCards/CippInfoBar.jsx';

// Mobile-responsive layouts using MUI Grid v2
<Grid container spacing={{ xs: 2, md: 3 }}>
  <Grid size={{ xs: 12, sm: 6, md: 3 }}>
    <CippInfoBar
      title="Total Users"
      value={dashboard.data?.Users || 0}
      color="primary"
    />
  </Grid>
  <Grid size={{ xs: 12, md: 8 }}>
    <CippChartCard
      title="User Activity"
      chartType="line"
      data={activityData}
    />
  </Grid>
</Grid>
```

These patterns provide a solid foundation for building consistent, maintainable pages throughout the CIPP application.