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
Advanced confirmation dialog for API actions with form inputs, validation, and comprehensive action processing.

**Key Features:**
- Dynamic form generation with validation
- Single and bulk row operations
- Automatic data replacement and processing
- Real-time form validation
- API result display with progress tracking
- Mobile-responsive full-screen mode
- Custom data formatters and functions
- Query cache invalidation

**Props:**
- `createDialog` (object, required): Dialog control object from useDialog hook
- `title` (string): Dialog title
- `api` (object, required): API action configuration
- `row` (object/array): Data row(s) for action - supports single objects or arrays for bulk
- `fields` (array): Additional input fields for user data collection
- `relatedQueryKeys` (array): Query keys to invalidate after successful operation
- `dialogAfterEffect` (function): Callback function after successful API call
- `allowResubmit` (boolean): Allow form resubmission after initial submit - default: false
- `children` (ReactNode/function): Custom content or render function
- `defaultvalues` (object): Default form values

**API Configuration:**
```jsx
{
  url: '/api/ActionEndpoint',              // API endpoint
  method: 'POST',                          // HTTP method (POST/GET)
  type: 'POST',                            // Action type (legacy)
  data: { param: '{rowField}' },           // Data template with row field substitution
  multiPost: false,                        // Send individual requests vs bulk
  confirmText: 'Are you sure?',            // Confirmation message
  customFunction: (row, action, data) => {}, // Custom handler instead of API call
  customDataformatter: (row, action, data) => {}, // Transform data before API call
  dataFunction: (row, data) => {},         // Process data with row context
  postEntireRow: false,                    // Send entire row as data
  setDefaultValues: true,                  // Pre-populate form from row data
  noConfirm: false,                        // Skip dialog and execute immediately
  link: '/path/[field]',                   // Navigate to URL instead of API call
  external: false,                         // Open link in new tab
  target: '_blank',                        // Link target
  onSuccess: (result) => {},               // Success callback
  relatedQueryKeys: ['cache-key'],         // Query keys to invalidate
  replacementBehaviour: 'removeNulls',     // Data processing behavior
}
```

**Field Configuration:**
```jsx
{
  name: 'reason',                          // Form field name
  label: 'Reason for Action',             // Field label
  type: 'textarea',                       // Field type (matches CippFormComponent types)
  required: true,                         // Required validation
  defaultValue: '',                       // Default value
  validators: {                           // React Hook Form validation rules
    required: 'Reason is required',
    minLength: { value: 10, message: 'At least 10 characters' }
  },
  // Additional CippFormComponent props supported
}
```

**Data Template Replacement:**
- `{fieldName}` - Replaced with row[fieldName] value
- `!value` - Literal value (removes ! prefix)
- Works with nested objects and arrays

**Usage Examples:**

**Basic Confirmation Dialog:**
```jsx
import { CippApiDialog } from '../../../src/components/CippComponents/CippApiDialog';
import { useDialog } from '../../../src/hooks/use-dialog';

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

  const deleteAction = {
    url: '/api/RemoveUser',
    method: 'POST',
    data: { userID: '{id}' },
    confirmText: 'Are you sure you want to delete [displayName]?'
  };

  return (
    <>
      <Button onClick={() => handleAction(userData, deleteAction)}>
        Delete User
      </Button>

      {actionData.ready && (
        <CippApiDialog
          createDialog={createDialog}
          title="Confirm Deletion"
          api={actionData.action}
          row={actionData.data}
          relatedQueryKeys={['users']}
        />
      )}
    </>
  );
};
```

**Dialog with Form Fields:**
```jsx
const resetPasswordAction = {
  url: '/api/ExecResetPass',
  method: 'POST',
  data: { ID: '{id}', mustChangePass: '{mustChangePass}' },
  confirmText: 'Reset password for [displayName]?'
};

const resetPasswordFields = [
  {
    name: 'mustChangePass',
    label: 'User must change password at next logon',
    type: 'switch',
    defaultValue: true,
  },
  {
    name: 'sendEmail',
    label: 'Send new password via email',
    type: 'switch',
    defaultValue: false,
  },
  {
    name: 'newPassword',
    label: 'Custom Password (optional)',
    type: 'password',
    validators: {
      minLength: { value: 8, message: 'Minimum 8 characters' }
    }
  }
];

<CippApiDialog
  createDialog={createDialog}
  title="Reset Password"
  api={resetPasswordAction}
  row={selectedUser}
  fields={resetPasswordFields}
  relatedQueryKeys={['users', 'user-details']}
/>
```

**Bulk Operations:**
```jsx
const bulkDisableAction = {
  url: '/api/ExecDisableUser',
  method: 'POST',
  data: { ID: '{id}', AccountEnabled: false },
  multiPost: true, // Send individual requests
  confirmText: 'Disable [length] selected users?'
};

<CippApiDialog
  createDialog={createDialog}
  title="Bulk Disable Users"
  api={bulkDisableAction}
  row={selectedUsers} // Array of user objects
  relatedQueryKeys={['users']}
/>
```

**Custom Data Processing:**
```jsx
const customAction = {
  url: '/api/UpdateUser',
  method: 'POST',
  customDataformatter: (row, action, formData) => ({
    userPrincipalName: row.userPrincipalName,
    changes: {
      department: formData.newDepartment,
      manager: formData.selectedManager?.value,
      updatedBy: currentUser.mail,
      updatedAt: new Date().toISOString()
    }
  }),
  confirmText: 'Update user [displayName]?'
};
```

**Features:**
- **Template Replacement**: Field names in `{}` are replaced with row data
- **Bulk Support**: Handles single objects or arrays of objects
- **Form Validation**: Real-time validation with React Hook Form
- **Mobile Responsive**: Full-screen on mobile devices
- **Progress Tracking**: Shows API results and progress for bulk operations
- **Query Invalidation**: Automatically refreshes related data after success

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
Advanced code display component with syntax highlighting, editing capabilities, and copy functionality.

**Key Features:**
- Dual display modes: syntax highlighting and full editor
- Monaco Editor integration for advanced editing
- Automatic theme switching (light/dark)
- Built-in copy-to-clipboard functionality
- Line number support
- Word wrapping and text overflow handling
- Multiple language support

**Props:**
- `code` (string, required): Code content to display
- `language` (string, default: 'json'): Programming language for syntax highlighting
- `showLineNumbers` (boolean, default: false): Display line numbers
- `startingLineNumber` (number, default: 1): Starting line number for syntax view
- `wrapLongLines` (boolean, default: true): Enable word wrapping
- `type` (string, default: 'syntax'): Display type ('syntax' or 'editor')
- `editorHeight` (string, default: '500px'): Height for editor mode

**Display Types:**
- **syntax**: Read-only syntax highlighting using react-syntax-highlighter
- **editor**: Interactive Monaco editor with full editing capabilities

**Usage Examples:**

**Basic Syntax Highlighting:**
```jsx
import { CippCodeBlock } from '../CippComponents/CippCodeBlock';

const jsonData = {
  "user": "john.doe@example.com",
  "permissions": ["read", "write"],
  "lastLogin": "2024-01-15T10:30:00Z"
};

<CippCodeBlock
  code={JSON.stringify(jsonData, null, 2)}
  language="json"
  showLineNumbers={true}
/>
```

**Monaco Editor Mode:**
```jsx
<CippCodeBlock
  code={editableCode}
  language="json"
  type="editor"
  editorHeight="400px"
  showLineNumbers={true}
/>
```

**PowerShell Script Display:**
```jsx
const powershellScript = `Get-MgUser -UserId $UserId | Select-Object DisplayName, Mail`;

<CippCodeBlock
  code={powershellScript}
  language="powershell"
  showLineNumbers={true}
/>
```

### CippCopyToClipBoard
Copy-to-clipboard functionality with visual feedback.

**Props:**
- `text` (string, required): Text to copy to clipboard
- `type` (string): Display type ('button', 'chip') - default: 'button'
- `visible` (boolean): Show/hide component - default: true

**Usage:**
```jsx
import { CippCopyToClipBoard } from '../CippComponents/CippCopyToClipboard';

// Button type
<CippCopyToClipBoard
  text={user.userPrincipalName}
  type="button"
/>

// Chip type
<CippCopyToClipBoard
  text={secretValue}
  type="chip"
/>
```

### CippDropzone
File upload component with drag-and-drop functionality.

**Props:**
- `onDrop` (function, required): File drop handler
- `accept` (object): Accepted file types
- `maxFiles` (number): Maximum number of files
- `multiple` (boolean): Allow multiple files
- `disabled` (boolean): Disable dropzone

**Usage:**
```jsx
import { CippDropzone } from '../CippComponents/CippDropzone';

const handleFileDrop = (acceptedFiles) => {
  console.log('Uploaded files:', acceptedFiles);
};

<CippDropzone
  onDrop={handleFileDrop}
  accept={{
    'text/csv': ['.csv'],
    'application/json': ['.json']
  }}
  maxFiles={1}
  multiple={false}
/>
```

### CippAutocomplete
Enhanced autocomplete component with API integration.

**Props:**
- `value` (any): Selected value
- `onChange` (function): Change handler
- `label` (string): Field label
- `placeholder` (string): Placeholder text
- `options` (array): Static options array
- `url` (string): API endpoint for dynamic options
- `data` (object): Additional API parameters
- `optionKey` (string): Property name for option value
- `optionLabel` (string): Property name for option display
- `loading` (boolean): Loading state
- `disabled` (boolean): Disable component
- `multiple` (boolean): Multiple selection
- `freeSolo` (boolean): Allow custom values

**Usage:**
```jsx
import { CippAutocomplete } from '../CippComponents/CippAutocomplete';

// Static options
const departments = [
  { id: 'it', name: 'Information Technology' },
  { id: 'hr', name: 'Human Resources' },
];

<CippAutocomplete
  value={selectedDepartment}
  onChange={setSelectedDepartment}
  label="Department"
  options={departments}
  optionKey="id"
  optionLabel="name"
/>

// API-driven options
<CippAutocomplete
  value={selectedUser}
  onChange={setSelectedUser}
  label="Select User"
  url="/api/ListUsers"
  data={{ tenantFilter: tenantId }}
  optionKey="userPrincipalName"
  optionLabel="displayName"
  multiple={false}
/>
```

### CippCsvExportButton
Button component for exporting table data to CSV format.

**Props:**
- `data` (array, required): Data to export
- `filename` (string): Export filename - default: 'export.csv'
- `headers` (array): Custom column headers
- `disabled` (boolean): Disable export
- `variant` (string): Button variant

**Usage:**
```jsx
import { CippCsvExportButton } from '../CippComponents/CippCsvExportButton';

<CippCsvExportButton
  data={tableData}
  filename="users-export.csv"
  headers={['Name', 'Email', 'Department']}
  variant="outlined"
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

### PropertyList
Basic list container for property display components.

**Props:**
- `children` (ReactNode, required): PropertyListItem components

**Usage:**
```jsx
import { PropertyList } from '../property-list';
import { PropertyListItem } from '../property-list-item';

<PropertyList>
  <PropertyListItem label="Name" value={user.displayName} />
  <PropertyListItem label="Email" value={user.mail} />
</PropertyList>
```

### ActionList
List container for action items with standardized styling.

**Props:**
- `children` (ReactNode, required): ActionListItem components

**Usage:**
```jsx
import { ActionList } from '../action-list';
import { ActionListItem } from '../action-list-item';

<ActionList>
  <ActionListItem 
    icon={<EditIcon />} 
    label="Edit User" 
    onClick={handleEdit} 
  />
  <ActionListItem 
    icon={<DeleteIcon />} 
    label="Delete User" 
    onClick={handleDelete} 
  />
</ActionList>
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