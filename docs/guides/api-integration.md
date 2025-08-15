# API Integration Guide

This guide covers how to integrate with CIPP's API system using the standardized hooks and utilities. CIPP uses TanStack Query (React Query) for data fetching with custom wrappers that provide consistent error handling, caching, and loading states.

## Core API Functions

### ApiGetCall - Standard Data Fetching

The `ApiGetCall` hook is used for GET requests with automatic caching, error handling, and retry logic.

```javascript
import { ApiGetCall } from "/src/api/ApiCall";

// Basic usage
const users = ApiGetCall({
  url: "/api/ListUsers",
  queryKey: "users",
  data: { tenantFilter: "contoso.com" }
});

// Access data and loading states
if (users.isLoading) return <div>Loading...</div>;
if (users.isError) return <div>Error: {users.error?.message}</div>;
if (users.isSuccess) return <div>Users: {users.data?.length}</div>;
```

#### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `url` | string | required | API endpoint to call |
| `queryKey` | string | required | Unique key for caching |
| `data` | object | undefined | Query parameters |
| `waiting` | boolean | true | Enable/disable the query |
| `retry` | number | 3 | Number of retry attempts |
| `bulkRequest` | boolean | false | Execute multiple requests with data array |
| `toast` | boolean | false | Show error toasts |
| `onResult` | function | undefined | Callback for each result |
| `staleTime` | number | 300000 | Cache staleness time (5 min) |
| `refetchOnWindowFocus` | boolean | false | Refetch on window focus |
| `refetchOnMount` | boolean | true | Refetch on component mount |
| `refetchOnReconnect` | boolean | true | Refetch on reconnect |
| `keepPreviousData` | boolean | false | Keep previous data during refetch |
| `relatedQueryKeys` | string\|array | undefined | Keys to invalidate after success |

#### Advanced Examples

```javascript
// Bulk requests - execute multiple API calls
const bulkUsers = ApiGetCall({
  url: "/api/ListUsers",
  queryKey: "bulkUsers",
  bulkRequest: true,
  data: [
    { tenantFilter: "contoso.com" },
    { tenantFilter: "fabrikam.com" },
    { tenantFilter: "tailspintoys.com" }
  ],
  onResult: (result) => {
    console.log("Received result:", result);
  }
});

// Conditional queries
const conditionalData = ApiGetCall({
  url: "/api/GetDetails",
  queryKey: ["details", userId],
  waiting: !!userId, // Only run if userId exists
  data: { id: userId }
});

// With error toasts and cache invalidation
const tenants = ApiGetCall({
  url: "/api/ListTenants",
  queryKey: "tenants",
  toast: true, // Show error toasts
  relatedQueryKeys: ["tenantsCount", "tenantStats"] // Invalidate related caches
});
```

### ApiPostCall - Data Mutations

The `ApiPostCall` hook is used for POST requests (create, update, delete operations) with mutation support.

```javascript
import { ApiPostCall } from "/src/api/ApiCall";

// Basic usage
const createUser = ApiPostCall({
  relatedQueryKeys: ["users", "userCount"], // Invalidate these queries on success
  onResult: (result) => {
    console.log("User created:", result);
  }
});

// Trigger the mutation
const handleCreateUser = () => {
  createUser.mutate({
    url: "/api/AddUser",
    data: {
      displayName: "John Doe",
      userPrincipalName: "john@contoso.com",
      tenantFilter: "contoso.com"
    }
  });
};

// Check mutation state
if (createUser.isPending) return <div>Creating user...</div>;
if (createUser.isError) return <div>Error: {createUser.error?.message}</div>;
if (createUser.isSuccess) return <div>User created successfully!</div>;
```

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `relatedQueryKeys` | string\|array | Query keys to invalidate on success ("*" for all) |
| `onResult` | function | Callback for each result |

#### Mutation Parameters (passed to mutate())

| Parameter | Type | Description |
|-----------|------|-------------|
| `url` | string | API endpoint to call |
| `data` | object\|array | POST data (array for bulk requests) |
| `bulkRequest` | boolean | Execute multiple requests with data array |

#### Advanced Examples

```javascript
// Bulk mutations
const bulkDelete = ApiPostCall({
  relatedQueryKeys: "*", // Invalidate all queries
});

const handleBulkDelete = () => {
  bulkDelete.mutate({
    url: "/api/RemoveUser",
    bulkRequest: true,
    data: [
      { userPrincipalName: "user1@contoso.com", tenantFilter: "contoso.com" },
      { userPrincipalName: "user2@contoso.com", tenantFilter: "contoso.com" }
    ]
  });
};

// With form integration
const updateUser = ApiPostCall({
  relatedQueryKeys: ["users", "userDetails"],
  onResult: (result) => {
    // Handle success (e.g., redirect or show success message)
    router.push("/users");
  }
});

const handleSubmit = (formData) => {
  updateUser.mutate({
    url: "/api/EditUser",
    data: {
      ...formData,
      tenantFilter: selectedTenant
    }
  });
};
```

### ApiGetCallWithPagination - Infinite Scroll

For endpoints that support pagination, use `ApiGetCallWithPagination` to implement infinite scrolling.

```javascript
import { ApiGetCallWithPagination } from "/src/api/ApiCall";

const auditLogs = ApiGetCallWithPagination({
  url: "/api/ListAuditLogs",
  queryKey: "auditLogs",
  data: { 
    tenantFilter: "contoso.com",
    manualPagination: true // Enable pagination
  },
  toast: true
});

// Access paginated data
const allLogs = auditLogs.data?.pages?.flatMap(page => page.data) || [];

// Load more data
const handleLoadMore = () => {
  if (auditLogs.hasNextPage && !auditLogs.isFetchingNextPage) {
    auditLogs.fetchNextPage();
  }
};

// Render with infinite scroll
return (
  <div>
    {allLogs.map(log => <LogItem key={log.id} log={log} />)}
    {auditLogs.hasNextPage && (
      <button 
        onClick={handleLoadMore}
        disabled={auditLogs.isFetchingNextPage}
      >
        {auditLogs.isFetchingNextPage ? "Loading..." : "Load More"}
      </button>
    )}
  </div>
);
```

## Error Handling Patterns

### Automatic Retry Logic

All API calls include intelligent retry logic:
- Maximum 3 retries by default (configurable)
- No retry on: 302, 401, 403, 404, 500 status codes
- Automatic auth redirect detection (302 to AAD login)
- Toast notifications for final failures (when `toast: true`)

### Custom Error Handling

```javascript
// Handle errors in component
const users = ApiGetCall({
  url: "/api/ListUsers",
  queryKey: "users",
  toast: false // Disable automatic toasts
});

if (users.isError) {
  const error = getCippError(users.error);
  return <Alert severity="error">{error}</Alert>;
}
```

### Error Processing Utility

The `getCippError` utility extracts meaningful error messages:

```javascript
import { getCippError } from "/src/utils/get-cipp-error";

// Returns the most appropriate error message from various response formats
const errorMessage = getCippError(error);
```

## Query Key Conventions

### Consistent Naming

Use descriptive, hierarchical query keys:

```javascript
// Good examples
ApiGetCall({ queryKey: "users", ... })                    // List all users
ApiGetCall({ queryKey: ["user", userId], ... })           // Specific user
ApiGetCall({ queryKey: ["users", tenantId], ... })        // Users by tenant
ApiGetCall({ queryKey: ["userDetails", userId], ... })    // User details
ApiGetCall({ queryKey: ["userDevices", userId], ... })    // User's devices

// Avoid generic keys
ApiGetCall({ queryKey: "data", ... })                     // Too generic
ApiGetCall({ queryKey: "apiCall", ... })                  // Not descriptive
```

### Cache Invalidation Strategy

Use `relatedQueryKeys` to maintain data consistency:

```javascript
// When creating a user, invalidate user lists
const createUser = ApiPostCall({
  relatedQueryKeys: ["users", "userCount", "tenantStats"]
});

// When updating a specific user, invalidate related data
const updateUser = ApiPostCall({
  relatedQueryKeys: [
    "users",                    // User list
    ["user", userId],           // Specific user
    ["userDetails", userId],    // User details
    ["userDevices", userId]     // User's devices
  ]
});

// Invalidate everything (use sparingly)
const massUpdate = ApiPostCall({
  relatedQueryKeys: "*"
});
```

## Integration with Components

### Table Integration

```javascript
// Standard table page pattern
const UsersPage = () => {
  const users = ApiGetCall({
    url: "/api/ListUsers",
    queryKey: "users",
    data: { tenantFilter: currentTenant }
  });

  return (
    <CippTablePage
      title="Users"
      apiUrl="/api/ListUsers"
      apiData={{ tenantFilter: currentTenant }}
      queryKey="users"
      columns={[
        { field: "displayName", headerName: "Display Name" },
        { field: "userPrincipalName", headerName: "UPN" }
      ]}
      // Table automatically handles loading, error states
    />
  );
};
```

### Form Integration

```javascript
// Form with API mutation
const UserForm = ({ userId }) => {
  const { data: user, isLoading } = ApiGetCall({
    url: "/api/GetUser",
    queryKey: ["user", userId],
    data: { userId },
    waiting: !!userId // Only fetch if userId exists
  });

  const updateUser = ApiPostCall({
    relatedQueryKeys: ["users", ["user", userId]],
    onResult: () => {
      // Handle success
      showToast({ message: "User updated successfully" });
    }
  });

  const handleSubmit = (formData) => {
    updateUser.mutate({
      url: "/api/EditUser",
      data: { ...formData, userId }
    });
  };

  if (isLoading) return <LoadingSkeleton />;

  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
      <button 
        type="submit" 
        disabled={updateUser.isPending}
      >
        {updateUser.isPending ? "Saving..." : "Save"}
      </button>
    </form>
  );
};
```

### Real-time Updates with onResult

```javascript
// Progress tracking for bulk operations
const [progress, setProgress] = useState(0);
const [results, setResults] = useState([]);

const bulkOperation = ApiGetCall({
  url: "/api/BulkOperation",
  queryKey: "bulkOperation",
  bulkRequest: true,
  data: itemsToProcess,
  onResult: (result) => {
    // Update progress and collect results
    setResults(prev => [...prev, result]);
    setProgress(prev => prev + 1);
  }
});

return (
  <div>
    <LinearProgress 
      variant="determinate" 
      value={(progress / itemsToProcess.length) * 100} 
    />
    <div>Processed: {progress} / {itemsToProcess.length}</div>
    {results.map((result, index) => (
      <ResultItem key={index} result={result} />
    ))}
  </div>
);
```

## Performance Optimization

### Caching Strategy

```javascript
// Long-lived data (rarely changes)
const tenants = ApiGetCall({
  url: "/api/ListTenants",
  queryKey: "tenants",
  staleTime: 1000 * 60 * 10, // 10 minutes
  refetchOnWindowFocus: false
});

// Frequently changing data
const notifications = ApiGetCall({
  url: "/api/GetNotifications",
  queryKey: "notifications",
  staleTime: 1000 * 30, // 30 seconds
  refetchOnWindowFocus: true
});

// Static reference data
const licenses = ApiGetCall({
  url: "/api/ListLicenses",
  queryKey: "licenses",
  staleTime: Infinity, // Cache forever
  refetchOnMount: false
});
```

### Conditional Queries

```javascript
// Only fetch when dependencies are available
const userDetails = ApiGetCall({
  url: "/api/GetUserDetails",
  queryKey: ["userDetails", userId],
  waiting: !!userId && !!selectedTenant,
  data: { userId, tenantFilter: selectedTenant }
});

// Fetch based on user permissions
const { hasAccess } = usePermissions();
const adminData = ApiGetCall({
  url: "/api/GetAdminData",
  queryKey: "adminData",
  waiting: hasAccess(['admin.*'])
});
```

## Best Practices

1. **Use descriptive query keys** that clearly identify the data
2. **Set appropriate stale times** based on data freshness requirements
3. **Invalidate related queries** after mutations to maintain consistency
4. **Handle loading and error states** in your components
5. **Use conditional queries** to avoid unnecessary requests
6. **Leverage onResult callbacks** for progress tracking and real-time updates
7. **Configure retry and toast settings** appropriately for your use case
8. **Use bulk requests sparingly** and only when the API supports them efficiently

## Common Patterns

### Master-Detail Pattern

```javascript
// Master list
const users = ApiGetCall({
  url: "/api/ListUsers",
  queryKey: "users"
});

// Detail view (conditional on selection)
const [selectedUserId, setSelectedUserId] = useState(null);
const userDetails = ApiGetCall({
  url: "/api/GetUserDetails",
  queryKey: ["userDetails", selectedUserId],
  waiting: !!selectedUserId,
  data: { userId: selectedUserId }
});
```

### Dependent Queries

```javascript
// First query
const tenant = ApiGetCall({
  url: "/api/GetTenant",
  queryKey: ["tenant", tenantId]
});

// Second query depends on first
const tenantUsers = ApiGetCall({
  url: "/api/ListUsers",
  queryKey: ["users", tenantId],
  waiting: tenant.isSuccess,
  data: { tenantFilter: tenant.data?.defaultDomainName }
});
```

### Search and Filter

```javascript
const [searchTerm, setSearchTerm] = useState("");
const [filters, setFilters] = useState({});

const searchResults = ApiGetCall({
  url: "/api/SearchUsers",
  queryKey: ["userSearch", searchTerm, filters],
  waiting: searchTerm.length >= 3, // Only search with 3+ characters
  data: { search: searchTerm, ...filters }
});
```

This API integration guide covers the core patterns used throughout CIPP. For component-specific examples, see the individual component documentation.