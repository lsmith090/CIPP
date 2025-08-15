# CippComponents

Utility components that provide common functionality across the CIPP application. These components handle cross-cutting concerns like API interactions, tenant management, dialogs, and layout helpers.

## Core Utility Components

### CippTablePage
Pre-built page layout optimized for data tables with standard elements.

**Key Features:**
- Automatic tenant context handling
- Integrated table with CippDataTable
- Standard page layout with container
- Built-in tenant warning alerts
- Consistent spacing and styling

**Props:**
- `title` (string): Page title
- `apiUrl` (string): API endpoint URL
- `apiData` (object): Additional API parameters
- `apiDataKey` (string): Response data path
- `actions` (array): Row actions for table
- `columns` (array): Table column definitions
- `offCanvas` (object): Off-canvas panel config
- `tableFilter` (ReactNode): Custom filter components
- `cardButton` (ReactNode): Header action button
- `noDataButton` (ReactNode): Empty state button
- `tenantInTitle` (boolean): Include tenant in title
- `filters` (array): Initial table filters
- `queryKey` (string): React Query cache key

**Usage:**
```jsx
import { CippTablePage } from '../CippComponents/CippTablePage';

<CippTablePage
  title="User Management"
  apiUrl="/api/ListUsers"
  apiData={{ includeGuests: false }}
  actions={userActions}
  queryKey="users"
  cardButton={
    <Button variant="contained" onClick={() => router.push('/users/add')}>
      Add User
    </Button>
  }
  offCanvas={{
    extendedInfoFields: ['id', 'mail', 'displayName', 'department']
  }}
/>
```

### CippOffCanvas
Slide-out panel for displaying detailed information and additional actions.

**Props:**
- `visible` (boolean): Panel visibility state
- `onClose` (function): Close handler
- `extendedData` (object): Data object to display
- `extendedInfoFields` (array): Fields to show in detail view
- `actions` (array): Actions available in panel
- `children` (ReactNode): Custom content
- `customComponent` (ReactNode): Custom component to render
- `isFetching` (boolean): Loading state
- `width` (number): Panel width in pixels

**Usage:**
```jsx
import { CippOffCanvas } from '../CippComponents/CippOffCanvas';

const [offCanvasVisible, setOffCanvasVisible] = useState(false);
const [selectedData, setSelectedData] = useState({});

<CippOffCanvas
  visible={offCanvasVisible}
  onClose={() => setOffCanvasVisible(false)}
  extendedData={selectedData}
  extendedInfoFields={[
    'id', 'displayName', 'mail', 'userPrincipalName',
    'department', 'jobTitle', 'accountEnabled'
  ]}
  actions={[
    {
      label: 'Edit User',
      icon: <EditIcon />,
      customFunction: (data) => router.push(`/users/edit/${data.id}`),
      noConfirm: true,
    }
  ]}
  children={
    <UserAdditionalDetails userId={selectedData.id} />
  }
/>
```

### CippApiDialog
Confirmation dialog for API actions with form inputs and validation.

**Props:**
- `createDialog` (object): Dialog control object from useDialog
- `title` (string): Dialog title
- `api` (object): API configuration
- `row` (object): Data row for action
- `fields` (array): Additional input fields
- `relatedQueryKeys` (array): Query keys to invalidate

**Field Configuration:**
```jsx
{
  name: 'reason',
  label: 'Reason for Action',
  type: 'textarea',
  required: true,
  defaultValue: '',
  validation: yup.string().required('Reason is required'),
}
```

**Usage:**
```jsx
import { CippApiDialog } from '../CippComponents/CippApiDialog';
import { useDialog } from '../../hooks/use-dialog';

const MyComponent = () => {
  const createDialog = useDialog();
  const [actionData, setActionData] = useState({ ready: false });

  const handleAction = (rowData, action) => {
    setActionData({
      data: rowData,
      action: action,
      ready: true,
    });
    createDialog.handleOpen();
  };

  return (
    <>
      {/* Trigger button */}
      <Button onClick={() => handleAction(userData, deleteAction)}>
        Delete User
      </Button>

      {/* Dialog */}
      {actionData.ready && (
        <CippApiDialog
          createDialog={createDialog}
          title="Confirm Action"
          api={actionData.action}
          row={actionData.data}
          fields={[
            {
              name: 'confirmationText',
              label: 'Type "DELETE" to confirm',
              type: 'textField',
              required: true,
              validation: yup.string().oneOf(['DELETE'], 'Must type DELETE'),
            }
          ]}
          relatedQueryKeys={['users']}
        />
      )}
    </>
  );
};
```

### CippTenantSelector
Global tenant selection component for multi-tenant operations.

**Props:**
- `value` (string): Selected tenant ID
- `onChange` (function): Selection change handler
- `label` (string): Selector label
- `disabled` (boolean): Disable selection
- `includeAllTenants` (boolean): Include "All Tenants" option
- `size` (string): Component size
- `variant` (string): Visual variant

**Usage:**
```jsx
import { CippTenantSelector } from '../CippComponents/CippTenantSelector';

const [selectedTenant, setSelectedTenant] = useState('');

<CippTenantSelector
  value={selectedTenant}
  onChange={setSelectedTenant}
  label="Select Tenant"
  includeAllTenants={true}
  size="small"
/>
```

### CippApiResults
Component for displaying API operation results with success/error states.

**Props:**
- `apiObject` (object): API call object from React Query
- `showSuccess` (boolean): Show success messages
- `showError` (boolean): Show error messages
- `successMessage` (string): Custom success message
- `errorMessage` (string): Custom error message

**Usage:**
```jsx
import { CippApiResults } from '../CippComponents/CippApiResults';
import { ApiPostCall } from '../../api/ApiCall';

const MyComponent = () => {
  const postCall = ApiPostCall({
    datafromUrl: true,
    relatedQueryKeys: ['users'],
  });

  const handleSubmit = () => {
    postCall.mutate({
      url: '/api/AddUser',
      data: formData,
    });
  };

  return (
    <>
      <Button onClick={handleSubmit}>Submit</Button>
      <CippApiResults 
        apiObject={postCall}
        successMessage="User created successfully!"
      />
    </>
  );
};
```

### CippHead
Page head component for managing document title and metadata.

**Props:**
- `title` (string): Page title
- `description` (string): Page description
- `keywords` (string): SEO keywords
- `noIndex` (boolean): Prevent search indexing

**Usage:**
```jsx
import { CippHead } from '../CippComponents/CippHead';

<CippHead 
  title="User Management"
  description="Manage users and their permissions"
/>
```

## Specialized Components

### CippFormTenantSelector
Tenant selection component specifically for forms.

**Props:**
- `name` (string): Form field name
- `label` (string): Field label
- `formControl` (object): React Hook Form control
- `required` (boolean): Required validation
- `multiple` (boolean): Allow multiple selection
- `excludeCurrentTenant` (boolean): Exclude current tenant

**Usage:**
```jsx
import { CippFormTenantSelector } from '../CippComponents/CippFormTenantSelector';

<CippFormTenantSelector
  name="targetTenant"
  label="Target Tenant"
  formControl={formControl}
  required
/>
```

### CippFormUserSelector
User selection component with search and filtering.

**Props:**
- `name` (string): Form field name
- `label` (string): Field label
- `formControl` (object): React Hook Form control
- `multiple` (boolean): Multiple selection
- `userType` (string): Filter by user type
- `includeGroups` (boolean): Include groups in results

**Usage:**
```jsx
import { CippFormUserSelector } from '../CippComponents/CippFormUserSelector';

<CippFormUserSelector
  name="assignedUsers"
  label="Assigned Users"
  formControl={formControl}
  multiple={true}
  userType="users"
/>
```

### CippFormGroupSelector
Group selection component with type filtering.

**Props:**
- `name` (string): Form field name
- `label` (string): Field label
- `formControl` (object): React Hook Form control
- `multiple` (boolean): Multiple selection
- `groupTypes` (array): Allowed group types
- `includeDistributionGroups` (boolean): Include distribution groups

**Usage:**
```jsx
import { CippFormGroupSelector } from '../CippComponents/CippFormGroupSelector';

<CippFormGroupSelector
  name="memberOf"
  label="Group Membership"
  formControl={formControl}
  multiple={true}
  groupTypes={['security', 'office365']}
/>
```

### CippTimeAgo
Relative time display component with automatic updates.

**Props:**
- `date` (Date/string): Date to display
- `updateInterval` (number): Update interval in seconds
- `showTitle` (boolean): Show full date on hover

**Usage:**
```jsx
import { CippTimeAgo } from '../CippComponents/CippTimeAgo';

<CippTimeAgo 
  date={user.lastSignInDateTime}
  updateInterval={60}
  showTitle={true}
/>
```

### CippCodeBlock
Syntax-highlighted code display component.

**Props:**
- `code` (string): Code content
- `language` (string): Programming language
- `showLineNumbers` (boolean): Show line numbers
- `copyable` (boolean): Enable copy functionality
- `maxHeight` (number): Maximum height in pixels

**Usage:**
```jsx
import { CippCodeBlock } from '../CippComponents/CippCodeBlock';

<CippCodeBlock
  code={jsonData}
  language="json"
  copyable={true}
  showLineNumbers={true}
  maxHeight={400}
/>
```

## Dialog Components

### CippComponentDialog
Reusable dialog wrapper for components.

**Props:**
- `open` (boolean): Dialog open state
- `onClose` (function): Close handler
- `title` (string): Dialog title
- `maxWidth` (string): Maximum width
- `fullScreen` (boolean): Full screen on mobile
- `actions` (ReactNode): Custom action buttons
- `children` (ReactNode): Dialog content

**Usage:**
```jsx
import { CippComponentDialog } from '../CippComponents/CippComponentDialog';

<CippComponentDialog
  open={dialogOpen}
  onClose={() => setDialogOpen(false)}
  title="User Details"
  maxWidth="md"
  actions={
    <Button onClick={handleSave} variant="contained">
      Save Changes
    </Button>
  }
>
  <UserDetailsForm />
</CippComponentDialog>
```

### CippTableDialog
Dialog specifically for displaying table data.

**Props:**
- `open` (boolean): Dialog open state
- `onClose` (function): Close handler
- `title` (string): Dialog title
- `data` (array): Table data
- `columns` (array): Table columns
- `actions` (array): Row actions

**Usage:**
```jsx
import { CippTableDialog } from '../CippComponents/CippTableDialog';

<CippTableDialog
  open={tableDialogOpen}
  onClose={() => setTableDialogOpen(false)}
  title="User Licenses"
  data={userLicenses}
  columns={licenseColumns}
  actions={licenseActions}
/>
```

## Layout Components

### CippSpeedDial
Floating action button with expandable options.

**Props:**
- `actions` (array): Action configurations
- `icon` (ReactNode): Main button icon
- `position` (object): Position configuration
- `hidden` (boolean): Hide speed dial

**Action Configuration:**
```jsx
{
  name: 'Add User',
  icon: <UserPlusIcon />,
  onClick: () => router.push('/users/add'),
}
```

**Usage:**
```jsx
import { CippSpeedDial } from '../CippComponents/CippSpeedDial';

const speedDialActions = [
  {
    name: 'Add User',
    icon: <UserPlusIcon />,
    onClick: () => router.push('/users/add'),
  },
  {
    name: 'Bulk Import',
    icon: <DocumentArrowUpIcon />,
    onClick: () => router.push('/users/bulk-import'),
  },
];

<CippSpeedDial
  actions={speedDialActions}
  icon={<PlusIcon />}
  position={{ bottom: 16, right: 16 }}
/>
```

### CippPropertyList
Generic property list component for key-value displays.

**Props:**
- `items` (array): Property items
- `copyable` (boolean): Enable copy functionality
- `divider` (boolean): Show dividers
- `align` (string): Alignment ('horizontal', 'vertical')

**Item Structure:**
```jsx
{
  label: 'Property Name',
  value: 'Property Value',
  copyValue: 'Value to copy', // Optional
  chip: true, // Render as chip
  link: '/path/to/details', // Make clickable
}
```

**Usage:**
```jsx
import { CippPropertyList } from '../CippComponents/CippPropertyList';

const userProperties = [
  { label: 'Name', value: user.displayName },
  { label: 'Email', value: user.mail, copyable: true },
  { label: 'Status', value: user.accountEnabled ? 'Enabled' : 'Disabled', chip: true },
];

<CippPropertyList
  items={userProperties}
  copyable={true}
  divider={true}
  align="vertical"
/>
```

## Best Practices

1. **Component Reuse**: Use utility components consistently across the application
2. **Error Handling**: Always handle loading and error states appropriately
3. **Accessibility**: Ensure all components support keyboard navigation and screen readers
4. **Performance**: Use React.memo() for components that receive stable props
5. **Tenant Context**: Always consider multi-tenant scenarios when building components
6. **API Integration**: Use consistent patterns for API calls and caching
7. **Form Integration**: Prefer form-specific components when building forms
8. **Dialog Management**: Use useDialog hook for consistent dialog state management
9. **Code Splitting**: Lazy load heavy components when appropriate
10. **Documentation**: Document prop interfaces and usage patterns clearly