# CippCards Examples

Real-world examples of using CIPP card components in different scenarios.

## Dashboard Overview

A typical dashboard using multiple card types:

```jsx
import { Grid, Container } from '@mui/system';
import {
  CippInfoBar,
  CippInfoCard,
  CippChartCard,
  CippPropertyListCard,
  CippButtonCard
} from '../CippCards';
import { UserIcon, BuildingIcon, ShieldIcon } from '@heroicons/react/24/outline';
import { getCippFormatting } from '/src/utils/get-cipp-formatting';
import { hasPermission, PermissionCheck } from '/src/utils/permissions';

const TenantDashboard = () => {
  const { data: overview, isLoading } = ApiGetCall({
    url: '/api/ExecDashboard',
    queryKey: 'tenant-overview',
  });

  // Top metrics bar
  const metrics = [
    {
      label: 'Total Users',
      value: overview?.totalUsers || 0,
      icon: <UserIcon />,
      color: 'primary',
      change: '+5.2%',
      trend: 'up'
    },
    {
      label: 'Active Licenses',
      value: overview?.activeLicenses || 0,
      icon: <BuildingIcon />,
      color: 'success'
    },
    {
      label: 'Security Score',
      value: overview?.securityScore || 0,
      icon: <ShieldIcon />,
      color: overview?.securityScore > 70 ? 'success' : 'warning'
    }
  ];

  return (
    <Container maxWidth="xl">
      <Grid container spacing={3}>
        {/* Top metrics */}
        <Grid size={{ xs: 12 }}>
          <CippInfoBar items={metrics} />
        </Grid>

        {/* Key metrics cards */}
        <Grid size={{ xs: 12, sm: 6, md: 3 }}>
          <CippInfoCard
            label="Licensed Users"
            value={overview?.licensedUsers || 0}
            icon={<UserIcon />}
            actionLink="/identity/administration/users"
            actionText="Manage Users"
            isFetching={isLoading}
          />
        </Grid>

        <Grid size={{ xs: 12, sm: 6, md: 3 }}>
          <CippInfoCard
            label="Security Alerts"
            value={overview?.securityAlerts || 0}
            icon={<ShieldIcon />}
            actionLink="/security/incidents/list-alerts"
            actionText="View Alerts"
            isFetching={isLoading}
          />
        </Grid>

        {/* Chart visualization */}
        <Grid size={{ xs: 12, md: 6 }}>
          <CippChartCard
            title="User Login Activity"
            data={overview?.loginActivity || []}
            type="line"
            height={300}
            isFetching={isLoading}
          />
        </Grid>

        {/* Tenant details */}
        <Grid size={{ xs: 12, md: 6 }}>
          <CippPropertyListCard
            title="Tenant Information"
            propertyItems={[
              { label: 'Domain', value: overview?.primaryDomain, copyItems: true },
              { label: 'Region', value: overview?.region },
              { label: 'Created', value: getCippFormatting(overview?.createdDate, 'createdDate', 'text') },
              { label: 'License Type', value: overview?.licenseType, chip: true },
            ]}
            isFetching={isLoading}
          />
        </Grid>

        {/* Quick actions - with permission checks */}
        <PermissionCheck requiredPermissions={['users.write']}>
          <Grid size={{ xs: 12, sm: 6, md: 4 }}>
            <CippButtonCard
              title="Add New User"
              description="Create a new user account"
              buttonText="Create User"
              onButtonClick={() => router.push('/identity/administration/users/add')}
            />
          </Grid>
        </PermissionCheck>

        <PermissionCheck requiredPermissions={['security.scan']}>
          <Grid size={{ xs: 12, sm: 6, md: 4 }}>
            <CippButtonCard
              title="Run Security Scan"
              description="Perform security assessment"
              buttonText="Start Scan"
              onButtonClick={handleSecurityScan}
              loading={scanInProgress}
            />
          </Grid>
        </PermissionCheck>
      </Grid>
    </Container>
  );
};
```

## User Detail View

Detailed user information using property cards:

```jsx
import { CippPropertyListCard } from '/src/components/CippCards/CippPropertyListCard.jsx';
import { KeyIcon, XMarkIcon, UserIcon } from '@heroicons/react/24/outline';

const UserDetailView = ({ userId }) => {
  const { data: user, isLoading } = ApiGetCall({
    url: '/api/ListUsers',
    data: { userId },
    queryKey: ['user', userId],
  });

  const basicInfo = [
    { label: 'Display Name', value: user?.displayName },
    { label: 'Email', value: user?.mail, copyItems: true },
    { label: 'UPN', value: user?.userPrincipalName, copyItems: true },
    { label: 'Department', value: user?.department },
    { label: 'Job Title', value: user?.jobTitle },
    { label: 'Manager', value: user?.manager?.displayName, link: `/users/${user?.manager?.id}` },
  ];

  const accountInfo = [
    { 
      label: 'Status', 
      value: user?.accountEnabled ? 'Enabled' : 'Disabled',
      chip: true,
      chipColor: user?.accountEnabled ? 'success' : 'error'
    },
    { label: 'Created', value: getCippFormatting(user?.createdDateTime, 'createdDateTime', 'component') },
    { label: 'Last Sign In', value: getCippFormatting(user?.signInActivity?.lastSignInDateTime, 'lastSignInDateTime', 'component') },
    { 
      label: 'MFA Status', 
      value: user?.mfaEnabled ? 'Enabled' : 'Disabled',
      chip: true,
      chipColor: user?.mfaEnabled ? 'success' : 'warning'
    },
  ];

  const userActions = [
    {
      label: 'Reset Password',
      icon: <KeyIcon />,
      api: {
        url: '/api/ExecResetPass',
        method: 'POST',
        data: { ID: user?.id }
      }
    },
    {
      label: 'Enable Account',
      icon: <UserIcon />,
      api: {
        url: '/api/ExecDisableUser',
        method: 'POST',
        data: { ID: user?.id, AccountEnabled: true }
      },
      condition: (data) => !data.accountEnabled
    },
    {
      label: 'Disable Account',
      icon: <XMarkIcon />,
      api: {
        url: '/api/ExecDisableUser',
        method: 'POST',
        data: { ID: user?.id, AccountEnabled: false }
      },
      condition: (data) => data.accountEnabled
    },
  ];

  return (
    <Grid container spacing={3}>
      <Grid size={{ xs: 12, md: 6 }}>
        <CippPropertyListCard
          title="Basic Information"
          propertyItems={basicInfo}
          actionItems={userActions}
          data={user}
          copyItems={true}
          isFetching={isLoading}
        />
      </Grid>

      <Grid size={{ xs: 12, md: 6 }}>
        <CippPropertyListCard
          title="Account Details"
          propertyItems={accountInfo}
          isFetching={isLoading}
        />
      </Grid>
    </Grid>
  );
};
```

## Security Dashboard

Security-focused cards with status indicators:

```jsx
import { CippRemediationCard } from '/src/components/CippCards/CippRemediationCard.jsx';
import { CippInfoCard } from '/src/components/CippCards/CippInfoCard.jsx';
import { CippChartCard } from '/src/components/CippCards/CippChartCard.jsx';
import { ShieldIcon, ExclamationTriangleIcon } from '@heroicons/react/24/outline';

const SecurityDashboard = () => {
  const { data: securityData, isLoading } = ApiGetCall({
    url: '/api/ListSecurityDashboard',
    queryKey: 'security-dashboard',
  });

  const remediationItems = securityData?.remediationItems || [];
  const threats = securityData?.threats || [];

  return (
    <Grid container spacing={3}>
      {/* Security metrics */}
      <Grid size={{ xs: 12, sm: 6, md: 3 }}>
        <CippInfoCard
          label="Security Score"
          value={securityData?.securityScore || 0}
          icon={<ShieldIcon />}
          isFetching={isLoading}
        />
      </Grid>

      <Grid size={{ xs: 12, sm: 6, md: 3 }}>
        <CippInfoCard
          label="Active Threats"
          value={threats.length}
          icon={<ExclamationTriangleIcon />}
          actionLink="/security/incidents/list-alerts"
          actionText="View Threats"
          isFetching={isLoading}
        />
      </Grid>

      {/* Threat analysis chart */}
      <Grid size={{ xs: 12, md: 6 }}>
        <CippChartCard
          title="Threat Trends"
          data={securityData?.threatTrends || []}
          type="area"
          height={300}
          options={{
            colors: ['#f44336', '#ff9800', '#4caf50'],
            chart: { stacked: true }
          }}
          isFetching={isLoading}
        />
      </Grid>

      {/* Remediation items */}
      <Grid size={{ xs: 12 }}>
        <Typography variant="h6" gutterBottom>
          Security Remediations
        </Typography>
      </Grid>

      {remediationItems.map((item, index) => (
        <Grid size={{ xs: 12, sm: 6, md: 4 }} key={index}>
          <CippRemediationCard
            title={item.title}
            status={item.status}
            description={item.description}
            priority={item.priority}
            assignee={item.assignee}
            dueDate={getCippFormatting(item.dueDate, 'dueDate', 'text')}
            actions={[
              {
                label: 'Mark Complete',
                onClick: () => handleMarkComplete(item.id)
              },
              {
                label: 'Reassign',
                onClick: () => handleReassign(item.id)
              }
            ]}
          />
        </Grid>
      ))}
    </Grid>
  );
};
```

## License Management

License overview with actionable cards:

```jsx
import { CippPropertyListCard } from '/src/components/CippCards/CippPropertyListCard.jsx';
import { CippChartCard } from '/src/components/CippCards/CippChartCard.jsx';
import { CippButtonCard } from '/src/components/CippCards/CippButtonCard.jsx';

const LicenseManagement = () => {
  const { data: licenses, isLoading } = ApiGetCall({
    url: '/api/ListLicenses',
    queryKey: 'licenses',
  });

  const licenseActions = [
    {
      label: 'Assign License',
      icon: <PlusIcon />,
      customFunction: () => setAssignDialogOpen(true),
      noConfirm: true
    },
    {
      label: 'Remove License',
      icon: <MinusIcon />,
      api: {
        url: '/api/RemoveLicense',
        method: 'POST'
      }
    }
  ];

  return (
    <Grid container spacing={3}>
      {/* License overview */}
      <Grid size={{ xs: 12, md: 8 }}>
        <CippChartCard
          title="License Utilization"
          data={licenses?.utilizationChart || []}
          type="donut"
          height={400}
          options={{
            labels: licenses?.licenseTypes || [],
            colors: ['#2196f3', '#4caf50', '#ff9800', '#f44336']
          }}
          isFetching={isLoading}
        />
      </Grid>

      {/* License details */}
      <Grid size={{ xs: 12, md: 4 }}>
        <CippPropertyListCard
          title="License Summary"
          propertyItems={[
            { label: 'Total Licenses', value: licenses?.total || 0 },
            { label: 'Assigned', value: licenses?.assigned || 0 },
            { label: 'Available', value: licenses?.available || 0 },
            { label: 'Utilization', value: `${licenses?.utilizationPercent || 0}%`, chip: true }
          ]}
          actionItems={licenseActions}
          data={licenses}
          isFetching={isLoading}
        />
      </Grid>

      {/* Individual license types */}
      {licenses?.licenseDetails?.map((license, index) => (
        <Grid size={{ xs: 12, sm: 6, md: 4 }} key={index}>
          <CippPropertyListCard
            title={license.skuPartNumber}
            propertyItems={[
              { label: 'Product', value: license.skuId },
              { label: 'Total', value: license.prepaidUnits.enabled },
              { label: 'Consumed', value: license.consumedUnits },
              { label: 'Available', value: license.prepaidUnits.enabled - license.consumedUnits },
            ]}
            layout="single"
            isFetching={isLoading}
          />
        </Grid>
      ))}

      {/* Quick actions */}
      <Grid size={{ xs: 12, sm: 6, md: 4 }}>
        <CippButtonCard
          title="Bulk License Assignment"
          description="Assign licenses to multiple users"
          buttonText="Start Wizard"
          onButtonClick={() => router.push('/identity/administration/licensing-wizard')}
        />
      </Grid>
    </Grid>
  );
};
```

## Mobile Responsive Design

Cards that adapt to mobile screens:

```jsx
import { useMediaQuery, useTheme } from '@mui/material';

const ResponsiveDashboard = () => {
  const theme = useTheme();
  const isMobile = useMediaQuery(theme.breakpoints.down('md'));

  return (
    <Grid container spacing={isMobile ? 2 : 3}>
      {/* Full width on mobile */}
      <Grid size={{ xs: 12 }}>
        <CippInfoBar 
          items={metrics}
          spacing={isMobile ? 1 : 3}
          variant={isMobile ? 'compact' : 'normal'}
        />
      </Grid>

      {/* Stack cards vertically on mobile */}
      <Grid size={{ xs: 12, md: 6 }}>
        <CippPropertyListCard
          title="Details"
          propertyItems={properties}
          layout={isMobile ? 'single' : 'double'}
          align={isMobile ? 'vertical' : 'horizontal'}
        />
      </Grid>

      {/* Smaller chart on mobile */}
      <Grid size={{ xs: 12, md: 6 }}>
        <CippChartCard
          title="Analytics"
          data={chartData}
          height={isMobile ? 200 : 300}
          options={{
            chart: {
              toolbar: { show: !isMobile }
            }
          }}
        />
      </Grid>
    </Grid>
  );
};
```

## Advanced Customization

Custom styled cards with enhanced functionality:

```jsx
import { styled } from '@mui/material/styles';
import { CippPropertyListCard } from '/src/components/CippCards/CippPropertyListCard.jsx';

const CustomCard = styled(CippPropertyListCard)(({ theme, priority }) => ({
  borderLeft: `4px solid ${
    priority === 'high' 
      ? theme.palette.error.main 
      : priority === 'medium'
      ? theme.palette.warning.main
      : theme.palette.success.main
  }`,
  '& .MuiCardHeader-title': {
    fontSize: '1.1rem',
    fontWeight: 600,
  },
  '& .MuiCardContent-root': {
    paddingBottom: theme.spacing(2),
  }
}));

const AlertCard = ({ alert }) => {
  const severity = alert.severity?.toLowerCase();
  
  return (
    <CustomCard
      title={alert.title}
      priority={severity}
      propertyItems={[
        { 
          label: 'Severity', 
          value: alert.severity,
          chip: true,
          chipColor: severity === 'high' ? 'error' : severity === 'medium' ? 'warning' : 'info'
        },
        { label: 'Source', value: alert.source },
        { label: 'Detected', value: getCippFormatting(alert.detectedTime, 'detectedTime', 'component') },
        { label: 'Status', value: alert.status, chip: true },
      ]}
      actionItems={[
        {
          label: 'Investigate',
          icon: <MagnifyingGlassIcon />,
          customFunction: () => handleInvestigate(alert.id),
          noConfirm: true
        },
        {
          label: 'Dismiss',
          icon: <XMarkIcon />,
          api: {
            url: '/api/DismissAlert',
            method: 'POST',
            data: { alertId: alert.id }
          }
        }
      ]}
      data={alert}
    />
  );
};
```