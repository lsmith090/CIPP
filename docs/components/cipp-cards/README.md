# CippCards

Card components for displaying information, metrics, and interactive content in a consistent, visually appealing format. All cards support loading states, theming, and responsive design.

## Components

### CippInfoCard
Simple metric display card with optional action link.

**Props:**
- `label` (string, required): Display label for the metric
- `value` (number, required): Numeric value to display
- `icon` (ReactNode): Icon component (defaults to CubeIcon)
- `actionLink` (string): Optional navigation link
- `actionText` (string): Text for action button
- `isFetching` (boolean): Shows skeleton loading state
- `cardSize` (string): Size variant

**Usage:**
```jsx
import { CippInfoCard } from '../../src/components/CippCards/CippInfoCard';
import { UserIcon } from '@heroicons/react/24/outline';

<CippInfoCard
  label="Total Users"
  value={1247}
  icon={<UserIcon />}
  actionLink="/identity/administration/users"
  actionText="View All Users"
  isFetching={isLoading}
/>
```

### CippPropertyListCard
Displays key-value property lists with optional actions.

**Props:**
- `title` (string): Card header title
- `propertyItems` (array, required): Array of property objects
- `actionItems` (array): Array of action objects
- `align` (string): 'vertical' or 'horizontal' alignment
- `layout` (string): 'single' or 'double' column layout
- `copyItems` (boolean): Enable copy-to-clipboard functionality
- `showDivider` (boolean): Show dividers between items
- `isFetching` (boolean): Loading state
- `data` (object): Data object passed to actions
- `actionButton` (ReactNode): Button in card header
- `cardButton` (ReactNode): Button in card footer

**Property Item Structure:**
```jsx
{
  label: 'Display Name',
  value: 'John Doe',
  copyValue: 'john.doe@company.com', // Optional: value to copy instead
  chip: true, // Render as chip
  link: '/users/123', // Make clickable link
}
```

**Action Item Structure:**
```jsx
{
  label: 'Reset Password',
  icon: <KeyIcon />,
  api: {
    url: '/api/ResetPassword',
    data: { userId: '{userId}' },
    method: 'POST'
  },
  condition: (data) => data.accountEnabled, // Optional: show/hide condition
  noConfirm: false, // Skip confirmation dialog
  customFunction: (item, data, result) => {}, // Custom handler
}
```

**Usage:**
```jsx
import { CippPropertyListCard } from '../../src/components/CippCards/CippPropertyListCard';

const userProperties = [
  { label: 'Name', value: user.displayName },
  { label: 'Email', value: user.mail, copyItems: true },
  { label: 'Department', value: user.department },
  { label: 'Status', value: user.accountEnabled ? 'Enabled' : 'Disabled', chip: true },
];

const userActions = [
  {
    label: 'Reset Password',
    icon: <KeyIcon />,
    api: { url: '/api/ResetPassword', method: 'POST' },
  },
  {
    label: 'Disable Account',
    icon: <XMarkIcon />,
    api: { url: '/api/DisableUser', method: 'POST' },
    condition: (data) => data.accountEnabled,
  },
];

<CippPropertyListCard
  title="User Details"
  propertyItems={userProperties}
  actionItems={userActions}
  data={user}
  layout="single"
  copyItems={true}
  isFetching={isLoading}
/>
```

### CippChartCard
Chart visualization card component using ApexCharts with Material-UI theming.

**Props:**
- `title` (string): Chart title displayed in header
- `chartSeries` (array, default: []): Chart data series array
- `labels` (array, default: []): Chart labels for x-axis or pie slices
- `chartType` (string, default: 'donut'): Chart type ('donut', 'line', 'bar', 'pie')
- `actions` (array): Action menu items for chart interactions
- `isFetching` (boolean): Loading state for skeleton display
- `onClick` (function): Optional click handler for entire card

**Usage:**
```jsx
import { CippChartCard } from '../../src/components/CippCards/CippChartCard';

// Donut/Pie Chart
const pieData = [400, 300, 500, 200];
const pieLabels = ['Active', 'Inactive', 'Suspended', 'Guest'];

<CippChartCard
  title="User Status Distribution"
  chartSeries={pieData}
  labels={pieLabels}
  chartType="donut"
  isFetching={loading}
  actions={[
    { label: 'Export Data', onClick: handleExport },
    { label: 'Refresh', onClick: handleRefresh }
  ]}
/>

// Line Chart
const lineData = [
  { name: 'This Month', data: [10, 20, 30, 40, 50] },
  { name: 'Last Month', data: [15, 25, 35, 45, 55] }
];
const lineLabels = ['Week 1', 'Week 2', 'Week 3', 'Week 4', 'Week 5'];

<CippChartCard
  title="Weekly Activity Trend"
  chartSeries={lineData}
  labels={lineLabels}
  chartType="line"
  onClick={() => router.push('/analytics/details')}
/>

// Bar Chart
const barData = [65, 45, 80, 30];
const barLabels = ['Q1', 'Q2', 'Q3', 'Q4'];

<CippChartCard
  title="Quarterly Performance"
  chartSeries={barData}
  labels={barLabels}
  chartType="bar"
  isFetching={loading}
/>
```

### CippInfoBar
Horizontal information bar displaying metrics in a grid layout with optional off-canvas details.

**Props:**
- `data` (array, required): Array of metric data objects
- `isFetching` (boolean): Loading state for skeleton display

**Data Item Structure:**
```jsx
{
  name: 'Active Users',           // Display label
  data: '1,250',                 // Value to display
  icon: <UserIcon />,            // Optional: icon component
  color: 'primary',              // Optional: icon color
  toolTip: 'Total active users', // Optional: tooltip text
  offcanvas: {                   // Optional: off-canvas details
    title: 'User Details',
    propertyItems: [
      { label: 'Active', value: '1,250' },
      { label: 'Inactive', value: '45' }
    ]
  }
}
```

**Usage:**
```jsx
import { CippInfoBar } from '../../src/components/CippCards/CippInfoBar';

const metrics = [
  {
    name: 'Total Users',
    data: '1,247',
    icon: <UserIcon />,
    color: 'primary',
    toolTip: 'All registered users'
  },
  {
    name: 'Active Licenses',
    data: '890',
    icon: <LicenseIcon />,
    color: 'success'
  },
  {
    name: 'Security Alerts',
    data: '23',
    icon: <AlertIcon />,
    color: 'warning',
    offcanvas: {
      title: 'Alert Details',
      propertyItems: [
        { label: 'Critical', value: '5' },
        { label: 'High', value: '18' }
      ]
    }
  }
];

<CippInfoBar data={metrics} isFetching={loading} />
```

### CippButtonCard
Interactive card with primary action button.

**Props:**
- `title` (string): Card title
- `description` (string): Card description
- `buttonText` (string): Button label
- `buttonIcon` (ReactNode): Button icon
- `onButtonClick` (function): Button click handler
- `disabled` (boolean): Disable button
- `loading` (boolean): Show loading state
- `variant` (string): Card variant

**Usage:**
```jsx
import { CippButtonCard } from '../../src/components/CippCards/CippButtonCard';

<CippButtonCard
  title="Create New User"
  description="Add a new user to the organization"
  buttonText="Create User"
  buttonIcon={<PlusIcon />}
  onButtonClick={() => router.push('/identity/administration/users/add')}
  disabled={!hasPermission}
/>
```

### CippImageCard
Image content card with overlays and actions.

**Props:**
- `src` (string): Image URL
- `alt` (string): Alt text
- `title` (string): Overlay title
- `subtitle` (string): Overlay subtitle
- `actions` (array): Action buttons
- `overlay` (boolean): Show overlay
- `aspectRatio` (string): Image aspect ratio

**Usage:**
```jsx
import { CippImageCard } from '../../src/components/CippCards/CippImageCard';

<CippImageCard
  src="/assets/tenant-logo.png"
  alt="Tenant Logo"
  title="Contoso Corp"
  subtitle="Premium License"
  overlay={true}
  aspectRatio="16:9"
  actions={[
    { label: 'Edit', onClick: handleEdit },
    { label: 'Download', onClick: handleDownload },
  ]}
/>
```

### CippBannerListCard
Collapsible banner component for displaying grouped information with expandable details. See [CippBannerListCard.md](./CippBannerListCard.md) for detailed documentation.

### CippRemediationCard
Business Email Compromise security remediation status display card.

**Props:**
- `title` (string): Remediation title
- `status` (string): Status ('pending', 'in-progress', 'completed', 'failed')
- `description` (string): Description text
- `priority` (string): Priority level ('low', 'medium', 'high', 'critical')
- `assignee` (string): Assigned user
- `dueDate` (Date): Due date
- `actions` (array): Available actions

**Usage:**
```jsx
import { CippRemediationCard } from '../../src/components/CippCards/CippRemediationCard';

<CippRemediationCard
  title="Enable MFA for Admin Accounts"
  status="in-progress"
  description="Configure multi-factor authentication for all administrative accounts"
  priority="high"
  assignee="security-team@company.com"
  dueDate={new Date('2024-01-15')}
  actions={[
    { label: 'Mark Complete', onClick: handleComplete },
    { label: 'Reassign', onClick: handleReassign },
  ]}
/>
```

## Common Patterns

### Loading States
All cards support loading states via the `isFetching` prop:

```jsx
<CippInfoCard
  label="Loading..."
  value={0}
  isFetching={true} // Shows skeleton animation
/>
```

### Error Handling
Cards can display error states:

```jsx
<CippPropertyListCard
  title="Error Loading Data"
  propertyItems={[
    { label: 'Error', value: 'Failed to load user data', error: true }
  ]}
/>
```

### Responsive Design
Cards adapt to screen size:

```jsx
<Grid container spacing={3}>
  <Grid item xs={12} sm={6} md={4}>
    <CippInfoCard {...props} />
  </Grid>
  <Grid item xs={12} md={8}>
    <CippPropertyListCard layout="double" {...props} />
  </Grid>
</Grid>
```

### Theming
Cards respect Material-UI theme:

```jsx
<CippInfoCard
  sx={{
    backgroundColor: 'background.paper',
    borderColor: 'divider',
    '& .MuiCardHeader-title': {
      color: 'text.primary',
    },
  }}
  {...props}
/>
```

## Best Practices

1. **Consistent Metrics**: Use the same units and formatting across similar cards
2. **Action Grouping**: Group related actions together in the same card
3. **Loading States**: Always provide loading states for async data
4. **Error Handling**: Show meaningful error messages when data fails to load
5. **Accessibility**: Include proper ARIA labels and roles
6. **Performance**: Use React.memo() for cards with expensive rendering
7. **Responsive**: Design cards to work across all screen sizes
8. **Progressive Disclosure**: Use cards to show summary info with links to detailed views