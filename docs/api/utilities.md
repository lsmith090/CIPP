# API Call Utilities

This document provides comprehensive documentation for CIPP's API utility functions, which provide a standardized way to interact with the backend API using TanStack Query (React Query) with built-in error handling, caching, and retry logic.

## Overview

CIPP's API utilities are built on top of TanStack Query and provide three main functions:
- `ApiGetCall` - For GET requests with caching and retry logic
- `ApiPostCall` - For POST requests with mutation support  
- `ApiGetCallWithPagination` - For paginated GET requests with infinite scroll support

All utilities include automatic error handling, retry logic, and integration with CIPP's authentication system.

## ApiGetCall

The primary function for fetching data from the CIPP API with automatic caching, error handling, and retry logic.

### Signature

```javascript
ApiGetCall({
  url,                    // string - API endpoint
  queryKey,              // string - Unique cache key
  relatedQueryKeys,      // string|array - Keys to invalidate
  waiting,               // boolean - Enable/disable query
  retry,                 // number - Max retry attempts
  data,                  // object|array - Query parameters
  bulkRequest,           // boolean - Execute bulk requests
  toast,                 // boolean - Show error toasts
  onResult,              // function - Result callback
  staleTime,             // number - Cache staleness time
  refetchOnWindowFocus,  // boolean - Refetch on focus
  refetchOnMount,        // boolean - Refetch on mount
  refetchOnReconnect,    // boolean - Refetch on reconnect
  keepPreviousData       // boolean - Keep previous data
})
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `url` | string | **required** | API endpoint URL (e.g., "/api/ListUsers") |
| `queryKey` | string | **required** | Unique identifier for caching (e.g., "users") |
| `data` | object\|array | `undefined` | Query parameters or bulk request data |
| `waiting` | boolean | `true` | Whether to execute the query (false disables it) |
| `retry` | number | `3` | Maximum number of retry attempts |
| `bulkRequest` | boolean | `false` | Execute multiple requests with data array |
| `toast` | boolean | `false` | Show error toast notifications on failure |
| `onResult` | function | `undefined` | Callback executed for each result |
| `staleTime` | number | `300000` | Cache staleness time in milliseconds (5 minutes) |
| `refetchOnWindowFocus` | boolean | `false` | Refetch when window regains focus |
| `refetchOnMount` | boolean | `true` | Refetch when component mounts |
| `refetchOnReconnect` | boolean | `true` | Refetch when network reconnects |
| `keepPreviousData` | boolean | `false` | Keep previous data during refetch |
| `relatedQueryKeys` | string\|array | `undefined` | Query keys to invalidate after success |

### Return Value

Returns a React Query object with the following properties:

```javascript
{
  data,           // API response data
  isLoading,      // true if first load is in progress
  isFetching,     // true if any fetch is in progress
  isError,        // true if query has error
  isSuccess,      // true if query succeeded
  error,          // error object if query failed
  refetch,        // function to manually refetch
  // ...other React Query properties
}
```

### Basic Usage

```javascript
import { ApiGetCall } from "/src/api/ApiCall";

const MyComponent = () => {
  const users = ApiGetCall({
    url: "/api/ListUsers",
    queryKey: "users",
    data: { tenantFilter: "contoso.com" }
  });

  if (users.isLoading) return <div>Loading...</div>;
  if (users.isError) return <div>Error: {users.error?.message}</div>;
  
  return (
    <div>
      {users.data?.map(user => (
        <div key={user.id}>{user.displayName}</div>
      ))}
    </div>
  );
};
```

### Advanced Examples

#### Conditional Queries

```javascript
const userDetails = ApiGetCall({
  url: "/api/GetUserDetails",
  queryKey: ["userDetails", userId],
  data: { userId },
  waiting: !!userId, // Only fetch when userId exists
  toast: true        // Show error toasts
});
```

#### Bulk Requests

```javascript
const bulkUserData = ApiGetCall({
  url: "/api/ListUsers",
  queryKey: "bulkUsers",
  bulkRequest: true,
  data: [
    { tenantFilter: "contoso.com" },
    { tenantFilter: "fabrikam.com" },
    { tenantFilter: "tailspintoys.com" }
  ],
  onResult: (result) => {
    console.log("Received tenant data:", result);
  }
});
```

#### Custom Cache Configuration

```javascript
const staticData = ApiGetCall({
  url: "/api/GetStaticConfiguration", 
  queryKey: "staticConfig",
  staleTime: Infinity,           // Cache forever
  refetchOnMount: false,         // Don't refetch on mount
  refetchOnWindowFocus: false    // Don't refetch on focus
});
```

### Error Handling

ApiGetCall includes intelligent retry logic:

- **Maximum retries**: 3 by default (configurable)
- **No retry on**: 302, 401, 403, 404, 500 status codes
- **Authentication handling**: Automatic redirect detection for 302 responses to AAD login
- **Error toasts**: Optional toast notifications on final failure

```javascript
// HTTP status codes that prevent retries
const HTTP_STATUS_TO_NOT_RETRY = [302, 401, 403, 404, 500];

// Authentication redirect detection
if (error.response?.status === 302 && 
    error.response?.headers.get("location").includes("/.auth/login/aad")) {
  queryClient.invalidateQueries({ queryKey: ["authmecipp"] });
}
```

## ApiPostCall

Used for POST requests (create, update, delete operations) with mutation support and automatic cache invalidation.

### Signature

```javascript
ApiPostCall({
  relatedQueryKeys,  // string|array - Keys to invalidate on success
  onResult          // function - Result callback
})
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `relatedQueryKeys` | string\|array | Query keys to invalidate on success. Use "*" to invalidate all queries |
| `onResult` | function | Callback function executed for each result |

### Mutation Parameters

When calling `mutation.mutate()`, pass these parameters:

| Parameter | Type | Description |
|-----------|------|-------------|
| `url` | string | API endpoint URL |
| `data` | object\|array | POST data. Use array for bulk requests |
| `bulkRequest` | boolean | Execute multiple requests with data array |

### Return Value

Returns a React Query mutation object:

```javascript
{
  mutate,         // function to trigger the mutation
  isPending,      // true if mutation is in progress
  isError,        // true if mutation failed
  isSuccess,      // true if mutation succeeded
  error,          // error object if mutation failed
  data,           // mutation result data
  // ...other React Query mutation properties
}
```

### Basic Usage

```javascript
import { ApiPostCall } from "/src/api/ApiCall";

const MyComponent = () => {
  const createUser = ApiPostCall({
    relatedQueryKeys: ["users"], // Invalidate users cache on success
    onResult: (result) => {
      console.log("User created:", result);
    }
  });

  const handleCreateUser = (userData) => {
    createUser.mutate({
      url: "/api/AddUser",
      data: {
        displayName: userData.name,
        userPrincipalName: userData.email,
        tenantFilter: "contoso.com"
      }
    });
  };

  return (
    <button 
      onClick={() => handleCreateUser({ name: "John Doe", email: "john@contoso.com" })}
      disabled={createUser.isPending}
    >
      {createUser.isPending ? "Creating..." : "Create User"}
    </button>
  );
};
```

### Advanced Examples

#### Bulk Mutations

```javascript
const bulkDelete = ApiPostCall({
  relatedQueryKeys: "*", // Invalidate all queries
  onResult: (result) => {
    console.log("Bulk operation result:", result);
  }
});

const handleBulkDelete = (userIds) => {
  bulkDelete.mutate({
    url: "/api/RemoveUser",
    bulkRequest: true,
    data: userIds.map(id => ({
      userPrincipalName: id,
      tenantFilter: "contoso.com"
    }))
  });
};
```

#### Multiple Cache Invalidation

```javascript
const updateUser = ApiPostCall({
  relatedQueryKeys: [
    "users",                    // Users list
    ["user", userId],           // Specific user
    ["userDetails", userId],    // User details
    "userCount"                 // User statistics
  ]
});
```

### Cache Invalidation Strategy

The `relatedQueryKeys` parameter controls which cached data is refreshed after successful mutations:

```javascript
// Invalidate specific queries
relatedQueryKeys: ["users", "userCount"]

// Invalidate query with parameters
relatedQueryKeys: [["user", userId], ["userDetails", userId]]

// Invalidate all queries (use sparingly)
relatedQueryKeys: "*"

// Mixed invalidation
relatedQueryKeys: ["users", ["user", userId], "statistics"]
```

## ApiGetCallWithPagination

For endpoints that support pagination, provides infinite scroll functionality.

### Signature

```javascript
ApiGetCallWithPagination({
  url,        // string - API endpoint
  queryKey,   // string - Cache key
  retry,      // number - Max retries
  data,       // object - Query parameters
  toast,      // boolean - Show error toasts
  waiting     // boolean - Enable/disable query
})
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `url` | string | **required** | API endpoint URL |
| `queryKey` | string | **required** | Unique cache key |
| `data` | object | `undefined` | Query parameters (must include pagination config) |
| `retry` | number | `3` | Maximum retry attempts |
| `toast` | boolean | `false` | Show error toast notifications |
| `waiting` | boolean | `true` | Whether to execute the query |

### Pagination Configuration

The `data` object should include pagination configuration:

```javascript
data: {
  tenantFilter: "contoso.com",
  manualPagination: true,  // Enable pagination
  noPagination: false      // Disable pagination (overrides manualPagination)
}
```

### Return Value

Returns a React Query infinite query object:

```javascript
{
  data: {
    pages: [...],      // Array of page results
    pageParams: [...]  // Array of page parameters
  },
  isLoading,           // true if first load is in progress
  isFetching,          // true if any fetch is in progress
  isError,             // true if query has error
  error,               // error object if query failed
  hasNextPage,         // true if more pages available
  isFetchingNextPage,  // true if loading next page
  fetchNextPage,       // function to load next page
  // ...other React Query infinite properties
}
```

### Usage Example

```javascript
import { ApiGetCallWithPagination } from "/src/api/ApiCall";

const InfiniteListComponent = () => {
  const auditLogs = ApiGetCallWithPagination({
    url: "/api/ListAuditLogs",
    queryKey: "auditLogs",
    data: {
      tenantFilter: "contoso.com",
      manualPagination: true
    },
    toast: true
  });

  // Flatten all pages into single array
  const allLogs = auditLogs.data?.pages?.flatMap(page => page.data) || [];

  const handleLoadMore = () => {
    if (auditLogs.hasNextPage && !auditLogs.isFetchingNextPage) {
      auditLogs.fetchNextPage();
    }
  };

  if (auditLogs.isLoading) return <div>Loading...</div>;

  return (
    <div>
      {allLogs.map((log, index) => (
        <div key={`${log.id}-${index}`}>
          {log.activity} - {log.timestamp}
        </div>
      ))}
      
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
};
```

### Infinite Scroll Integration

```javascript
import { useInfiniteScroll } from "react-infinite-scroll-hook";

const InfiniteScrollList = () => {
  const query = ApiGetCallWithPagination({
    url: "/api/ListItems",
    queryKey: "infiniteItems",
    data: { manualPagination: true }
  });

  const [sentryRef] = useInfiniteScroll({
    loading: query.isFetchingNextPage,
    hasNextPage: query.hasNextPage ?? false,
    onLoadMore: query.fetchNextPage,
    disabled: query.isError,
    rootMargin: "0px 0px 400px 0px", // Start loading 400px before end
  });

  const items = query.data?.pages?.flatMap(page => page.data) || [];

  return (
    <div>
      {items.map((item, index) => (
        <ItemComponent key={`${item.id}-${index}`} item={item} />
      ))}
      
      {(query.isFetchingNextPage || query.hasNextPage) && (
        <div ref={sentryRef}>
          <LoadingSpinner />
        </div>
      )}
    </div>
  );
};
```

## Authentication Integration

All API utilities automatically integrate with CIPP's authentication system:

### Automatic Token Handling

- Tokens are automatically included in requests
- No manual token management required
- Uses Azure Static Web Apps authentication

### Authentication Error Handling

```javascript
// Automatic auth redirect detection
if (error.response?.status === 302 && 
    error.response?.headers.get("location").includes("/.auth/login/aad")) {
  // Automatically invalidate auth cache to trigger re-authentication
  queryClient.invalidateQueries({ queryKey: ["authmecipp"] });
}
```

### Tenant Context

Most API calls should include tenant context:

```javascript
import { useSettings } from "/src/hooks/use-settings";

const TenantAwareComponent = () => {
  const { currentTenant } = useSettings();

  const data = ApiGetCall({
    url: "/api/ListUsers",
    queryKey: ["users", currentTenant],
    data: { tenantFilter: currentTenant },
    waiting: !!currentTenant // Only fetch when tenant is selected
  });

  // Component logic...
};
```

## Error Processing

All API utilities use the `getCippError` utility for consistent error message extraction:

```javascript
import { getCippError } from "/src/utils/get-cipp-error";

// Error processing hierarchy
const extractError = (error) => {
  // 1. Try response.data.result
  if (error.response?.data?.result) return error.response.data.result;
  
  // 2. Try response.data.error  
  if (error.response?.data?.error) return error.response.data.error;
  
  // 3. Try response.data.message
  if (error.response?.data?.message) return error.response.data.message;
  
  // 4. Handle HTML responses (error pages)
  if (typeof error.response?.data === "string" && 
      error.response.data.includes("<!DOCTYPE html>")) {
    return error.message;
  }
  
  // 5. Try response.data.Results
  if (error.response?.data?.Results) return error.response.data.Results;
  
  // 6. Fallback to response.data or message
  return error.response?.data || error.message;
};
```

## Performance Considerations

### Caching Strategy

```javascript
// Short-lived data (frequent updates)
const notifications = ApiGetCall({
  url: "/api/GetNotifications",
  queryKey: "notifications",
  staleTime: 30000, // 30 seconds
  refetchOnWindowFocus: true
});

// Medium-lived data (occasional updates)
const users = ApiGetCall({
  url: "/api/ListUsers", 
  queryKey: "users",
  staleTime: 300000, // 5 minutes (default)
  refetchOnWindowFocus: false
});

// Long-lived data (rarely changes)
const configuration = ApiGetCall({
  url: "/api/GetConfiguration",
  queryKey: "config",
  staleTime: Infinity, // Cache forever
  refetchOnMount: false
});
```

### Query Key Best Practices

```javascript
// Good: Descriptive, hierarchical keys
ApiGetCall({ queryKey: "users" })                    // All users
ApiGetCall({ queryKey: ["user", userId] })           // Specific user
ApiGetCall({ queryKey: ["users", tenantId] })        // Users by tenant
ApiGetCall({ queryKey: ["userDevices", userId] })    // User's devices

// Bad: Generic or unclear keys
ApiGetCall({ queryKey: "data" })                     // Too generic
ApiGetCall({ queryKey: "apiCall" })                  // Not descriptive
```

### Bulk Request Optimization

```javascript
// Use bulk requests for multiple related API calls
const bulkData = ApiGetCall({
  url: "/api/GetUserData",
  queryKey: "bulkUserData", 
  bulkRequest: true,
  data: userIds.map(id => ({ userId: id })),
  onResult: (result) => {
    // Process each result as it arrives
    updateLocalState(result);
  }
});
```

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

### Real-time Progress Tracking

```javascript
const [progress, setProgress] = useState(0);
const [results, setResults] = useState([]);

const bulkOperation = ApiGetCall({
  url: "/api/BulkOperation",
  queryKey: "bulkOp",
  bulkRequest: true,
  data: itemsToProcess,
  onResult: (result) => {
    setResults(prev => [...prev, result]);
    setProgress(prev => prev + 1);
  }
});

// Progress: {progress} / {itemsToProcess.length}
```

This comprehensive documentation covers all aspects of CIPP's API utilities. These functions provide a robust, consistent way to interact with the backend API while handling authentication, caching, error handling, and performance optimization automatically.