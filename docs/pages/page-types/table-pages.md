# Table Pages - CippTablePage Pattern

Table pages are the most common page type in CIPP, used for displaying lists of data with filtering, sorting, and action capabilities. They provide a consistent interface for managing collections of entities like users, devices, policies, and reports.

## Core Component

The `CippTablePage` component is the foundation for all table-based pages in CIPP.

**Location**: `/src/components/CippComponents/CippTablePage.jsx`

## Basic Usage

```javascript
import { CippTablePage } from "../../../components/CippComponents/CippTablePage.jsx";
import { Layout as DashboardLayout } from "/src/layouts/index.js";

const Page = () => {
  return (
    <CippTablePage
      title="Users"
      apiUrl="/api/ListGraphRequest"
      apiData={{
        Endpoint: "users",
        $select: "id,userPrincipalName,displayName,mail",
        $orderby: "displayName"
      }}
      apiDataKey="Results"
      simpleColumns={["userPrincipalName", "displayName", "mail"]}
    />
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default Page;
```

## Component Props

### Required Props

| Prop | Type | Description |
|------|------|-------------|
| `title` | `string` | Page title displayed in header and browser tab |
| `apiUrl` | `string` | API endpoint for fetching table data |

### Core Configuration Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `apiData` | `object` | `{}` | Additional data sent with API request |
| `apiDataKey` | `string` | `"Results"` | Key in API response containing table data |
| `simpleColumns` | `array` | `[]` | Array of column names to display |
| `columns` | `array` | `[]` | Custom column definitions |
| `queryKey` | `string` | Auto-generated | React Query cache key |

### UI Enhancement Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `cardButton` | `ReactNode` | `null` | Action buttons in table header |
| `noDataButton` | `ReactNode` | `null` | Button shown when no data exists |
| `tenantInTitle` | `boolean` | `true` | Include tenant name in title |
| `filters` | `array` | `[]` | Predefined filter options |

### Advanced Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `actions` | `array` | `[]` | Row-level action definitions |
| `offCanvas` | `object` | `null` | Off-canvas panel configuration |
| `tableFilter` | `ReactNode` | `null` | Custom filter components |
| `sx` | `object` | `{flexGrow: 1, py: 4}` | Custom styling |

## Real-World Example: Users Page

Here's the actual implementation from `/src/pages/identity/administration/users/index.js`:

```javascript
import { CippTablePage } from "../../../components/CippComponents/CippTablePage.jsx";
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { Send, GroupAdd, PersonAdd } from "@mui/icons-material";
import Link from "next/link";
import { useSettings } from "/src/hooks/use-settings.js";
import { PermissionButton } from "../../../../utils/permissions";
import { CippUserActions } from "../../../components/CippComponents/CippUserActions.jsx";

const Page = () => {
  const pageTitle = "Users";
  const tenant = useSettings().currentTenant;
  const cardButtonPermissions = ["Identity.User.ReadWrite"];

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
      "onPremisesLastSyncDateTime",
      "onPremisesDistinguishedName",
      "otherMails",
    ],
    actions: CippUserActions(),
  };

  return (
    <CippTablePage
      title={pageTitle}
      apiUrl="/api/ListGraphRequest"
      cardButton={
        <>
          <PermissionButton
            requiredPermissions={cardButtonPermissions}
            component={Link}
            href="users/add"
            startIcon={<PersonAdd />}
          >
            Add User
          </PermissionButton>
          <PermissionButton
            requiredPermissions={cardButtonPermissions}
            component={Link}
            href="users/bulk-add"
            startIcon={<GroupAdd />}
          >
            Bulk Add Users
          </PermissionButton>
          <PermissionButton
            requiredPermissions={cardButtonPermissions}
            component={Link}
            href="users/invite"
            startIcon={<Send />}
          >
            Invite Guest
          </PermissionButton>
        </>
      }
      apiData={{
        Endpoint: "users",
        manualPagination: true,
        $select: "id,accountEnabled,businessPhones,city,createdDateTime,companyName,country,department,displayName,faxNumber,givenName,isResourceAccount,jobTitle,mail,mailNickname,mobilePhone,officeLocation,otherMails,postalCode,preferredDataLocation,preferredLanguage,proxyAddresses,showInAddressList,state,streetAddress,surname,usageLocation,userPrincipalName,userType,assignedLicenses,onPremisesSyncEnabled,OnPremisesImmutableId,onPremisesLastSyncDateTime,onPremisesDistinguishedName",
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
        "proxyAddresses",
        "assignedLicenses",
      ]}
      filters={filters}
    />
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default Page;
```

## Key Features

### 1. Automatic Data Fetching

The component automatically handles:
- API calls with React Query integration
- Tenant-aware requests (includes current tenant)
- Loading states and error handling
- Data caching and refetching

### 2. Column Configuration

#### Simple Columns
For basic display of API fields:

```javascript
simpleColumns={["displayName", "userPrincipalName", "mail"]}
```

#### Custom Columns
For advanced formatting and interactions:

```javascript
columns={[
  {
    id: "displayName",
    header: "Display Name",
    accessorKey: "displayName",
    cell: ({ row }) => (
      <Link href={`/users/${row.original.id}`}>
        {row.original.displayName}
      </Link>
    )
  }
]}
```

### 3. Filtering System

#### Predefined Filters
Quick filter buttons for common scenarios:

```javascript
filters={[
  {
    filterName: "Active Users",
    value: [{ id: "accountEnabled", value: "Yes" }],
    type: "column",
  },
  {
    filterName: "Disabled Users",
    value: [{ id: "accountEnabled", value: "No" }],
    type: "column",
  }
]}
```

#### Advanced Filtering
The table includes built-in filtering capabilities:
- Column-based filters
- Global search
- Date range filters
- Custom filter operators

### 4. Actions System

#### Row Actions
Actions available for individual rows:

```javascript
actions={[
  {
    label: "Edit",
    icon: <Edit />,
    handler: (row) => router.push(`/users/edit?id=${row.id}`),
    color: "primary"
  },
  {
    label: "Delete", 
    icon: <Delete />,
    handler: (row) => handleDelete(row.id),
    color: "error",
    confirmText: "Are you sure you want to delete this user?"
  }
]}
```

#### Bulk Actions
Actions for multiple selected rows:

```javascript
// Handled automatically by CippDataTable component
// Based on actions configuration with bulk: true
```

### 5. Off-Canvas Details

Detailed view panel that slides in from the right:

```javascript
offCanvas={
  extendedInfoFields: [
    "createdDateTime",
    "userPrincipalName", 
    "assignedLicenses",
    "department"
  ],
  actions: UserActions(), // Actions specific to the detail view
}
```

### 6. Permission Integration

The component integrates with CIPP's permission system:

```javascript
cardButton={
  <PermissionButton
    requiredPermissions={["Identity.User.ReadWrite"]}
    component={Link}
    href="users/add"
  >
    Add User
  </PermissionButton>
}
```

## Data Flow

1. **Component Mount**: `CippTablePage` mounts and initiates API call
2. **API Request**: Calls `apiUrl` with `apiData` and current tenant
3. **Data Processing**: Extracts data using `apiDataKey` 
4. **Table Rendering**: `CippDataTable` renders with processed data
5. **User Interaction**: Filters, sorting, actions trigger re-renders
6. **State Management**: React Query manages caching and updates

## Customization Patterns

### Custom Header Buttons

```javascript
cardButton={
  <Stack direction="row" spacing={2}>
    <Button variant="contained" onClick={handleAdd}>
      Add New
    </Button>
    <Button variant="outlined" onClick={handleImport}>
      Import Data
    </Button>
  </Stack>
}
```

### Custom No Data State

```javascript
noDataButton={
  <Box textAlign="center" py={4}>
    <Typography variant="h6">No data available</Typography>
    <Button variant="contained" onClick={handleCreateFirst}>
      Create Your First Item
    </Button>
  </Box>
}
```

### Custom Styling

```javascript
sx={{
  flexGrow: 1,
  py: 2,
  '& .MuiCard-root': {
    boxShadow: 'none',
    border: '1px solid',
    borderColor: 'divider'
  }
}}
```

## API Integration Patterns

### Microsoft Graph Integration

```javascript
apiData={{
  Endpoint: "users",
  $select: "id,displayName,userPrincipalName",
  $filter: "accountEnabled eq true",
  $orderby: "displayName",
  $top: 100
}}
```

### Custom API Endpoints

```javascript
apiUrl="/api/ListCustomData"
apiData={{
  customParam: "value",
  tenantFilter: tenant,
  additionalOptions: true
}}
```

### Manual Pagination

```javascript
apiData={{
  manualPagination: true,
  $count: true,
  $top: 999
}}
```

## Performance Considerations

### Data Optimization
- Use `$select` to limit returned fields
- Implement pagination for large datasets
- Cache frequently accessed data

### Rendering Optimization
- Use `simpleColumns` for basic displays
- Implement virtualization for very large tables
- Lazy load off-canvas details

### Memory Management
- React Query handles caching automatically
- Avoid storing large datasets in component state
- Use proper queryKey patterns for cache invalidation

## Common Patterns

### Entity List Page
```javascript
const EntityListPage = () => (
  <CippTablePage
    title="Entities"
    apiUrl="/api/ListEntities"
    simpleColumns={["name", "status", "created"]}
    actions={EntityActions()}
  />
);
```

### Report Page
```javascript
const ReportPage = () => (
  <CippTablePage
    title="Report"
    apiUrl="/api/GenerateReport"
    apiData={{ reportType: "security" }}
    tenantInTitle={false}
    filters={ReportFilters}
  />
);
```

### Audit Log Page
```javascript
const AuditLogPage = () => (
  <CippTablePage
    title="Audit Logs"
    apiUrl="/api/ListAuditLogs"
    simpleColumns={["timestamp", "user", "action", "result"]}
    offCanvas={{
      extendedInfoFields: ["details", "ipAddress", "userAgent"]
    }}
  />
);
```

## Best Practices

1. **Consistent Naming**: Use descriptive titles and consistent naming
2. **Performance**: Optimize API calls with proper `$select` and pagination
3. **Permissions**: Always implement proper permission checks
4. **User Experience**: Provide clear loading states and error messages
5. **Accessibility**: Ensure table is accessible with proper ARIA labels
6. **Responsive Design**: Tables should work on mobile devices
7. **Actions**: Group related actions logically
8. **Filtering**: Provide intuitive filter options for common use cases

## Troubleshooting

### Common Issues

**No Data Displayed**
- Check `apiDataKey` matches API response structure
- Verify API endpoint is correct and accessible
- Check network tab for API errors

**Columns Not Showing**
- Verify column names match API response fields
- Check for typos in `simpleColumns` array
- Ensure data exists in API response

**Actions Not Working**
- Check action handler functions are defined
- Verify permissions for action buttons
- Check console for JavaScript errors

**Performance Issues**
- Implement pagination for large datasets
- Use `$select` to limit returned fields
- Check React Query cache configuration

## Related Components

- [`CippDataTable`](../components/cipp-table/README.md) - Underlying table component
- [`CippUserActions`](../components/cipp-components/README.md) - User-specific actions
- [`PermissionButton`](../components/README.md) - Permission-gated buttons
- [`CippOffCanvas`](../components/cipp-components/README.md) - Detail panel component