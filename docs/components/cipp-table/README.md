# CippTable

Advanced data table system built on Material React Table with comprehensive filtering, sorting, pagination, and action capabilities. Designed for displaying large datasets with tenant-aware data fetching and permission-controlled actions.

## Components

### CippDataTable
Main data table component with full functionality.

**Key Features:**
- Material React Table integration
- Automatic column generation from API data
- Advanced filtering (text, select, multi-select, regex)
- Sorting with custom sort functions
- Row actions with confirmation dialogs
- Bulk actions and selection
- Export functionality (CSV, PDF)
- Virtualization for large datasets
- Responsive design
- Loading states and error handling

**Props:**
- `queryKey` (string/array): React Query cache key
- `data` (array): Static data (alternative to API)
- `columns` (array): Column definitions
- `api` (object): API configuration
- `actions` (array): Row action definitions
- `title` (string): Table title (default: "Report")
- `simpleColumns` (array): Column visibility configuration
- `filters` (array): Initial column filters
- `offCanvas` (object/boolean): Off-canvas detail panel configuration
- `exportEnabled` (boolean): Enable export functionality (default: true)
- `refreshFunction` (function): Custom refresh handler
- `onChange` (function): Selection change handler
- `maxHeightOffset` (string): Height calculation offset (default: "380px")
- `defaultSorting` (array): Initial sort state
- `isFetching` (boolean): Override loading state
- `columnVisibility` (object): Initial column visibility settings
- `simple` (boolean): Simple table mode (default: false)
- `cardButton` (ReactNode): Button in table header
- `noCard` (boolean): Render without card wrapper (default: false)
- `hideTitle` (boolean): Hide table title (default: false)
- `incorrectDataMessage` (string): Custom error message for malformed data

**API Configuration:**
```jsx
api: {
  url: '/api/ListUsers',
  data: { tenantFilter: 'contoso.com' }, // Additional parameters
  dataKey: 'Results', // Path to data in response (e.g., response.Results)
}
```

**Column Definition:**
```jsx
columns: [
  {
    accessorKey: 'displayName',
    header: 'Display Name',
    filterFn: 'contains',
    sortingFn: 'alphanumeric',
    enableSorting: true,
    enableColumnFilter: true,
    cell: ({ getValue }) => getValue(),
  },
  {
    accessorKey: 'mail',
    header: 'Email',
    enableClickToCopy: true,
  },
  {
    accessorKey: 'accountEnabled',
    header: 'Status',
    cell: ({ getValue }) => (
      <Chip 
        label={getValue() ? 'Enabled' : 'Disabled'}
        color={getValue() ? 'success' : 'error'}
        size="small"
      />
    ),
  }
]
```

**Action Definition:**
```jsx
actions: [
  {
    label: 'Edit User',
    icon: <EditIcon />,
    customFunction: (row, action, result) => {
      router.push(`/users/edit/${row.id}`);
    },
    noConfirm: true,
  },
  {
    label: 'Reset Password',
    icon: <KeyIcon />,
    api: {
      url: '/api/ExecResetPass',
      method: 'POST',
      data: { ID: '{id}' }, // Template replacement
    },
    fields: [ // Additional input fields for action
      {
        name: 'mustChangePass',
        label: 'Must Change Password',
        type: 'checkbox',
        defaultValue: true,
      }
    ],
    condition: (row) => row.accountEnabled, // Show/hide condition
    color: 'warning',
  },
  {
    label: 'Delete User',
    icon: <DeleteIcon />,
    api: {
      url: '/api/RemoveUser',
      method: 'POST',
      data: { ID: '{id}' },
    },
    condition: (row) => !row.isAdmin,
    color: 'error',
  }
]
```

**Usage:**
```jsx
import { CippDataTable } from '/src/components/CippTable/CippDataTable.js';

<CippDataTable
  queryKey="users-list"
  title="Users"
  api={{
    url: '/api/ListUsers',
    data: { tenantFilter: currentTenant },
    dataKey: 'Results',
  }}
  columns={userColumns}
  actions={userActions}
  filters={[
    { id: 'accountEnabled', value: true },
    { id: 'department', value: 'IT' }
  ]}
  offCanvas={{
    extendedInfoFields: ['id', 'objectId', 'createdDateTime'],
    children: <UserDetailsPanel />,
  }}
  exportEnabled={true}
  defaultSorting={[{ id: 'displayName', desc: false }]}
/>
```

### CippTablePage
Pre-configured page layout with table and standard elements.

**Props:**
- `title` (string): Page title
- `apiUrl` (string): API endpoint URL
- `apiData` (object): Additional API parameters
- `apiDataKey` (string): Response data path
- `actions` (array): Row actions
- `columns` (array): Column definitions
- `offCanvas` (object): Off-canvas configuration
- `tableFilter` (ReactNode): Filter components above table
- `cardButton` (ReactNode): Button in table header
- `noDataButton` (ReactNode): Button shown when no data
- `tenantInTitle` (boolean): Include tenant name in title
- `filters` (array): Initial filters
- `queryKey` (string): React Query key

**Usage:**
```jsx
import { CippTablePage } from '/src/components/CippComponents/CippTablePage.jsx';

<CippTablePage
  title="User Management"
  apiUrl="/api/ListUsers"
  apiData={{ includeGuests: false }}
  apiDataKey="Results"
  actions={userActions}
  queryKey="users"
  tenantInTitle={true}
  cardButton={
    <Button
      variant="contained"
      onClick={() => router.push('/users/add')}
    >
      Add User
    </Button>
  }
  offCanvas={{
    extendedInfoFields: [
      'id', 'userPrincipalName', 'mail', 'displayName',
      'jobTitle', 'department', 'accountEnabled', 'createdDateTime'
    ]
  }}
/>
```

### CippDataTableButton
Custom button component for table toolbars.

**Props:**
- `variant` (string): Button variant
- `icon` (ReactNode): Button icon
- `children` (ReactNode): Button text
- `onClick` (function): Click handler
- `disabled` (boolean): Disabled state
- `loading` (boolean): Loading state

**Usage:**
```jsx
import { CippDataTableButton } from '/src/components/CippTable/CippDataTableButton.jsx';

<CippDataTableButton
  variant="contained"
  icon={<PlusIcon />}
  onClick={handleAddUser}
  disabled={!hasPermission}
>
  Add User
</CippDataTableButton>
```

## Advanced Features

### Custom Filtering
```jsx
// Custom filter functions
const customFilters = {
  isActive: (row, columnId, value) => {
    return row.getValue('accountEnabled') === value;
  },
  domainMatch: (row, columnId, value) => {
    const email = row.getValue('mail') || '';
    return email.includes(`@${value}`);
  }
};

// Use in column definition
{
  accessorKey: 'mail',
  header: 'Email',
  filterFn: 'domainMatch',
}
```

### Custom Sorting
```jsx
// Custom sort functions
const customSorts = {
  dateTimeNullsLast: (a, b, columnId) => {
    const aVal = a.original[columnId];
    const bVal = b.original[columnId];
    
    if (!aVal && !bVal) return 0;
    if (!aVal) return 1;
    if (!bVal) return -1;
    
    return new Date(aVal) - new Date(bVal);
  }
};

// Use in column definition
{
  accessorKey: 'lastSignInDateTime',
  header: 'Last Sign In',
  sortingFn: 'dateTimeNullsLast',
}
```

### Off-Canvas Detail Panel
```jsx
const offCanvasConfig = {
  extendedInfoFields: [
    'id',
    'userPrincipalName', 
    'displayName',
    'mail',
    'jobTitle',
    'department',
    'manager',
    'accountEnabled',
    'createdDateTime',
    'lastPasswordChangeDateTime'
  ],
  children: ({ data }) => (
    <Stack spacing={2}>
      <Typography variant="h6">Additional Details</Typography>
      <UserLicensesList userId={data.id} />
      <UserGroupsList userId={data.id} />
      <UserSignInActivity userId={data.id} />
    </Stack>
  ),
  customComponent: UserDetailPanel, // Alternative to children
};
```

### Conditional Actions
```jsx
const conditionalActions = [
  {
    label: 'Enable Account',
    icon: <CheckIcon />,
    api: { url: '/api/EnableUser', method: 'POST' },
    condition: (row) => !row.accountEnabled,
    color: 'success',
  },
  {
    label: 'Disable Account', 
    icon: <XMarkIcon />,
    api: { url: '/api/DisableUser', method: 'POST' },
    condition: (row) => row.accountEnabled && !row.isAdmin,
    color: 'error',
  },
  {
    label: 'Convert to Shared Mailbox',
    icon: <MailIcon />,
    api: { url: '/api/ConvertMailbox', method: 'POST' },
    condition: (row) => row.mailboxType === 'UserMailbox',
    fields: [
      {
        name: 'keepLicense',
        label: 'Keep License',
        type: 'checkbox',
        defaultValue: false,
      }
    ],
  }
];
```

### Bulk Actions
```jsx
// Enable row selection for bulk actions
<CippDataTable
  {...props}
  enableRowSelection={true}
  onChange={(selectedRows) => {
    console.log('Selected:', selectedRows);
  }}
  renderTopToolbarCustomActions={({ table }) => (
    <Box sx={{ display: 'flex', gap: 1 }}>
      <Button
        variant="contained"
        disabled={table.getSelectedRowModel().rows.length === 0}
        onClick={() => handleBulkAction(table.getSelectedRowModel().rows)}
      >
        Bulk Action ({table.getSelectedRowModel().rows.length})
      </Button>
    </Box>
  )}
/>
```

### Export Configuration
```jsx
<CippDataTable
  {...props}
  exportEnabled={true}
  exportFormats={['csv', 'pdf']}
  exportFileName="users-export"
  exportOptions={{
    csv: {
      includeHeaders: true,
      delimiter: ',',
    },
    pdf: {
      orientation: 'landscape',
      title: 'User Report',
    }
  }}
/>
```

## Performance Optimization

### Virtualization
```jsx
// Enable virtualization for large datasets
<CippDataTable
  {...props}
  enableRowVirtualization={true}
  enableColumnVirtualization={true}
  rowVirtualizerProps={{
    estimateSize: 50, // Row height estimate
    overscan: 10, // Extra rows to render
  }}
/>
```

### Memoization
```jsx
import { useMemo } from 'react';

const UserTable = () => {
  // Memoize expensive column calculations
  const columns = useMemo(() => [
    {
      accessorKey: 'displayName',
      header: 'Display Name',
      cell: ({ getValue, row }) => (
        <UserAvatarCell 
          name={getValue()}
          email={row.original.mail}
        />
      ),
    }
  ], []);

  // Memoize action definitions
  const actions = useMemo(() => [
    {
      label: 'Edit',
      icon: <EditIcon />,
      customFunction: handleEdit,
      noConfirm: true,
    }
  ], [handleEdit]);

  return (
    <CippDataTable
      columns={columns}
      actions={actions}
      {...otherProps}
    />
  );
};
```

## Best Practices

1. **Query Keys**: Use descriptive, hierarchical query keys for proper caching
2. **Column Definitions**: Define columns outside component to prevent re-renders
3. **Action Conditions**: Use specific conditions to show/hide actions appropriately
4. **Error Handling**: Always handle API errors gracefully
5. **Loading States**: Provide clear loading indicators
6. **Responsive Design**: Test tables on various screen sizes
7. **Performance**: Use virtualization for large datasets (>1000 rows)
8. **Accessibility**: Ensure proper ARIA labels and keyboard navigation
9. **Export**: Only enable export when users need to work with data offline
10. **Filters**: Provide sensible default filters for common use cases