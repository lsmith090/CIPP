# CippBannerListCard

Collapsible banner component for displaying grouped information with expandable details. Ideal for showing lists of items with associated actions and detailed properties.

## Props

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| items | array | Yes | [] | Array of item objects to display |
| isCollapsible | boolean | No | false | Enable/disable collapse functionality |
| isFetching | boolean | No | false | Show loading skeleton state |
| children | ReactNode | No | - | Additional content to render |
| layout | string | No | 'dual' | Layout for property items ('single', 'dual') |

## Item Structure

Each item in the `items` array should have the following structure:

```jsx
{
  id: 'unique-identifier',           // Required: unique identifier
  cardLabelBox: 'Status',           // Required: left-side label
  // OR complex label structure:
  cardLabelBox: {
    cardLabelBoxHeader: 'Critical',
    cardLabelBoxText: 'Alert'
  },
  text: 'Main item description',     // Required: primary text
  subtext: 'Secondary information',  // Optional: subtitle
  statusColor: 'success.main',       // Optional: status indicator color
  statusText: 'Active',             // Optional: status text
  actionButton: <Button>Action</Button>, // Optional: action element
  cardLabelBoxActions: <IconButton/>, // Optional: actions in header
  
  // Expandable content (when isCollapsible is true):
  propertyItems: [                   // Optional: property list
    { label: 'Property', value: 'Value' }
  ],
  table: {                          // Optional: data table
    // CippDataTable props
  },
  children: <CustomComponent />,     // Optional: custom content
  isFetching: false                 // Optional: item-level loading
}
```

## Usage Examples

### Basic List
```jsx
import { CippBannerListCard } from '../../src/components/CippCards/CippBannerListCard';

const basicItems = [
  {
    id: '1',
    cardLabelBox: 'User',
    text: 'John Doe',
    subtext: 'john.doe@company.com',
    statusColor: 'success.main',
    statusText: 'Active'
  },
  {
    id: '2', 
    cardLabelBox: 'User',
    text: 'Jane Smith',
    subtext: 'jane.smith@company.com',
    statusColor: 'error.main',
    statusText: 'Disabled'
  }
];

<CippBannerListCard items={basicItems} />
```

### Collapsible with Details
```jsx
const detailedItems = [
  {
    id: 'security-1',
    cardLabelBox: {
      cardLabelBoxHeader: 'High',
      cardLabelBoxText: 'Priority'
    },
    text: 'Security Alert: Unusual Login Activity',
    subtext: 'Detected 15 minutes ago from unknown location',
    statusColor: 'warning.main',
    statusText: 'Under Review',
    propertyItems: [
      { label: 'Source IP', value: '192.168.1.100' },
      { label: 'Location', value: 'New York, US' },
      { label: 'User Agent', value: 'Chrome 96.0.4664.110' },
      { label: 'Risk Score', value: '8.5/10' }
    ],
    actionButton: (
      <Button variant="contained" size="small">
        Investigate
      </Button>
    )
  }
];

<CippBannerListCard 
  items={detailedItems} 
  isCollapsible={true}
  layout="single"
/>
```

### With Loading State
```jsx
<CippBannerListCard 
  items={[]} 
  isFetching={true}
/>
```

### With Actions and Custom Content
```jsx
const actionItems = [
  {
    id: 'license-1',
    cardLabelBox: 'E5',
    text: 'Microsoft 365 E5 License',
    subtext: '450 of 500 licenses used',
    statusColor: 'success.main',
    statusText: '90% Utilized',
    cardLabelBoxActions: (
      <IconButton size="small">
        <EditIcon />
      </IconButton>
    ),
    table: {
      data: licenseUsageData,
      columns: licenseColumns,
      simpleColumns: ['user', 'department', 'assignedDate']
    },
    actionButton: (
      <Button variant="outlined" size="small">
        Manage Licenses
      </Button>
    )
  }
];

<CippBannerListCard 
  items={actionItems}
  isCollapsible={true}
  layout="dual"
/>
```

### Empty State
```jsx
// When items array is empty, shows "No items available." message
<CippBannerListCard items={[]} />
```

## Integration Patterns

### With Data Fetching
```jsx
import { ApiGetCall } from '../../../api/ApiCall';

const AlertsList = () => {
  const { data: alerts = [], isLoading } = ApiGetCall({
    url: '/api/ListSecurityAlerts',
    queryKey: 'security-alerts'
  });

  const alertItems = alerts.map(alert => ({
    id: alert.id,
    cardLabelBox: alert.severity.toUpperCase(),
    text: alert.title,
    subtext: `Detected ${alert.detectedTime}`,
    statusColor: getSeverityColor(alert.severity),
    statusText: alert.status,
    propertyItems: [
      { label: 'Category', value: alert.category },
      { label: 'Source', value: alert.source },
      { label: 'Confidence', value: `${alert.confidence}%` }
    ]
  }));

  return (
    <CippBannerListCard 
      items={alertItems}
      isCollapsible={true}
      isFetching={isLoading}
    />
  );
};
```

### Common Use Cases
- **Security Alerts**: Display security incidents with expandable details
- **License Reports**: Show license utilization with detailed breakdowns
- **User Lists**: Display user information with expandable properties
- **System Status**: Show service status with detailed metrics
- **Audit Logs**: Display log entries with expandable event details

## Styling Notes

- Items automatically show hover effects when `isCollapsible` is enabled
- Status indicators use consistent color tokens from the Material-UI theme
- Responsive design adapts to mobile screens
- Loading skeletons match the expected content structure
- Dividers separate list items for better visual hierarchy

## Accessibility

- Proper keyboard navigation for collapsible items
- ARIA labels for expand/collapse functionality  
- Screen reader friendly status indicators
- Focus management for nested interactive elements