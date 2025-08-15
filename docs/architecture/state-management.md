# State Management Architecture

## Overview

CIPP employs a sophisticated hybrid state management approach optimized for Microsoft 365 partner management applications. The architecture combines React Query (TanStack Query v5) for server state management, Redux Toolkit for application-wide state, and React Context for UI settings - creating a performant and maintainable state management solution.

## Primary State Management: React Query (TanStack Query)

React Query serves as the primary state management system for all server-side data operations in CIPP, handling API calls, caching, synchronization, and background updates.

### Core Implementation

The API layer is implemented in `/src/api/ApiCall.jsx` with three main functions:

#### ApiGetCall - Standard GET Requests
```javascript
export function ApiGetCall(props) {
  const {
    url,
    queryKey,
    relatedQueryKeys,
    waiting = true,
    retry = 3,
    data,
    bulkRequest = false,
    toast = false,
    onResult,
    staleTime = 300000,
    refetchOnWindowFocus = false,
    refetchOnMount = true,
    refetchOnReconnect = true,
    keepPreviousData = false,
  } = props;
  
  // Implementation uses useQuery with custom retry logic
  // Handles bulk requests and query invalidation
}
```

#### ApiPostCall - POST/PUT Operations
```javascript
export function ApiPostCall({ relatedQueryKeys, onResult }) {
  const mutation = useMutation({
    mutationFn: async (props) => {
      const { url, data, bulkRequest } = props;
      // Handles both single and bulk POST operations
    },
    onSuccess: () => {
      // Automatic query cache invalidation
    },
  });
  return mutation;
}
```

#### ApiGetCallWithPagination - Infinite Scroll Support
```javascript
export function ApiGetCallWithPagination({
  url, queryKey, retry = 3, data, toast = false, waiting = true
}) {
  const queryInfo = useInfiniteQuery({
    queryKey: [queryKey],
    queryFn: async ({ pageParam = null, signal }) => {
      // Handles Microsoft Graph API pagination
    },
    getNextPageParam: (lastPage) => {
      // Extracts nextLink from Microsoft Graph responses
    },
  });
  return queryInfo;
}
```

### Query Configuration Patterns

**Standard Query Configuration:**
- **Stale Time**: 300000ms (5 minutes) - Data remains fresh for 5 minutes
- **Cache Time**: Default 5 minutes inactive cache retention
- **Retry Logic**: Custom retry function with HTTP status code filtering
- **Background Refetch**: Disabled by default for performance

**Authentication-Specific Queries:**
```javascript
// Azure SWA Authentication
const swaStatus = ApiGetCall({
  url: "/.auth/me",
  queryKey: "authmeswa",
  staleTime: 120000,
  refetchOnWindowFocus: true,
});

// CIPP Permission System
const currentRole = ApiGetCall({
  url: "/api/me",
  queryKey: "authmecipp",
  waiting: !swaStatus.isSuccess || swaStatus.data?.clientPrincipal === null,
});
```

### Cache Invalidation Strategy

CIPP implements intelligent cache invalidation through the `relatedQueryKeys` parameter:

```javascript
// Automatic invalidation after mutations
if (relatedQueryKeys) {
  const clearKeys = Array.isArray(relatedQueryKeys) ? relatedQueryKeys : [relatedQueryKeys];
  setTimeout(() => {
    if (relatedQueryKeys === "*") {
      queryClient.invalidateQueries(); // Invalidate all queries
    } else {
      clearKeys.forEach((key) => {
        queryClient.invalidateQueries({ queryKey: [key] });
      });
    }
  }, 1000);
}
```

## Application State: Redux Toolkit

Redux Toolkit is used minimally in CIPP, primarily for application-wide notifications and toasts.

### Store Configuration

```javascript
// /src/store/index.js
export const store = configureStore({
  reducer: rootReducer,
  devTools: process.env.REACT_APP_ENABLE_REDUX_DEV_TOOLS === 'true',
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: [FLUSH, REHYDRATE, PAUSE, PERSIST, PURGE, REGISTER],
      },
    }).concat([]),
})
```

### Root Reducer Structure

```javascript
// /src/store/root-reducer.js
export const rootReducer = combineReducers({
  [toastsSlice.name]: toastsSlice.reducer,
})
```

### Toast Management

The only Redux slice currently in use manages application-wide toast notifications:

```javascript
// /src/store/toasts.js
export const toastsSlice = createSlice({
  name: "toasts",
  initialState: {
    toasts: [],
    currentIndex: 0,
  },
  reducers: {
    showToast: (state, { payload: { message, title, toastError } }) => {
      state.currentIndex++;
      state.toasts.push({
        message,
        title,
        date: Date.now(),
        toastError,
        index: state.currentIndex,
      });
    },
    closeToast: (state, { payload: { index } }) => {
      state.toasts = state.toasts.filter((el) => el.index !== index);
    },
    resetToast: () => ({ ...initialState }),
  },
});
```

### Usage Pattern

Toast notifications are triggered from API calls and components:

```javascript
import { useDispatch } from "react-redux";
import { showToast } from "../store/toasts";

const dispatch = useDispatch();

// Error handling in API calls
dispatch(showToast({
  message: `${getCippError(error)}`,
  title: `${error.config?.params?.tenantFilter ? error.config?.params?.tenantFilter : ""} Error`,
}));
```

## UI State Management: Settings Context

React Context manages UI-specific state including theme, navigation preferences, and current tenant context.

### Settings Context Implementation

```javascript
// /src/contexts/settings-context.js
const initialSettings = {
  direction: "ltr",
  paletteMode: "light",
  currentTheme: { value: "light", label: "light" },
  pinNav: true,
  currentTenant: null,
  showDevtools: false,
  customBranding: {
    colour: "#F77F00",
    logo: null,
  },
};

export const SettingsContext = createContext({
  ...initialSettings,
  handleReset: () => {},
  handleUpdate: () => {},
  isCustom: false,
});
```

### Context Provider Features

**Persistent Storage Integration:**
- Automatic localStorage synchronization
- Memory fallback for environments without localStorage
- Settings restoration on application initialization

**Update Mechanism:**
```javascript
const handleUpdate = useCallback((settings) => {
  setState((prevState) => {
    storeSettings({
      ...prevState,
      ...settings,
    });
    
    return {
      ...prevState,
      ...settings,
    };
  });
}, []);
```

**Customization Detection:**
```javascript
const isCustom = useMemo(() => {
  return !isEqual(initialSettings, {
    direction: state.direction,
    paletteMode: state.paletteMode,
    currentTheme: state.currentTheme,
    pinNav: state.pinNav,
  });
}, [state]);
```

## Data Persistence Strategy

CIPP implements comprehensive data persistence across all state management layers.

### React Query Persistence

Implemented in `/src/pages/_app.js` using TanStack Query persistence:

```javascript
useEffect(() => {
  if (typeof window !== "undefined") {
    const localStoragePersister = createSyncStoragePersister({
      storage: window.localStorage,
    });

    persistQueryClient({
      queryClient,
      persister: localStoragePersister,
      maxAge: 1000 * 60 * 60 * 24, // 24 hours
      staleTime: 1000 * 60 * 5, // 5 minutes
      buster: "v1",
      dehydrateOptions: {
        shouldDehydrateQuery: (query) => {
          const queryIsReadyForPersistence = query.state.status === "success";
          if (queryIsReadyForPersistence) {
            const { queryKey } = query;
            if (!queryKey || !queryKey.length) return false;
            
            const queryKeyString = String(queryKey[0] || "");
            const excludeFromPersisting = excludeQueryKeys.some((key) =>
              queryKeyString.includes(key)
            );
            return !excludeFromPersisting;
          }
          return queryIsReadyForPersistence;
        },
      },
    });
  }
}, []);
```

**Exclusion Strategy:**
```javascript
const excludeQueryKeys = ["authmeswa", "alertsDashboard"];
```

### Settings Persistence

The Settings Context automatically persists all UI preferences:
- Theme settings and custom branding
- Navigation preferences
- Current tenant context
- Developer tools visibility

## Integration Patterns

### Authentication Integration

The hybrid state management system seamlessly integrates with CIPP's authentication flow:

1. **Azure SWA Session** → Cached via React Query (`authmeswa`)
2. **CIPP Permissions** → Cached via React Query (`authmecipp`) 
3. **User Settings** → Merged with Settings Context
4. **Navigation State** → Filtered based on permissions

### Error Handling Integration

Errors flow through the state management layers:

1. **API Errors** → Caught by React Query retry logic
2. **Authentication Errors** → Handled by PrivateRoute component
3. **User Notifications** → Dispatched to Redux toast slice
4. **Settings Errors** → Graceful fallback to memory storage

### Performance Optimization

The hybrid approach optimizes for:

- **Server State**: React Query handles caching, background sync, and optimistic updates
- **Application State**: Redux Toolkit provides predictable updates for global notifications
- **UI State**: Context provides efficient local state with persistence
- **Persistence**: Selective caching with exclusion rules prevents storage bloat

## Best Practices

### Query Key Conventions

```javascript
// Authentication queries
"authmeswa" // Azure SWA session
"authmecipp" // CIPP user permissions

// Entity-specific queries
"usersList" // Users listing
"tenantInfo" // Tenant information
"[feature]Settings" // Feature-specific settings
```

### Cache Management

```javascript
// Invalidate related queries after mutations
const mutation = ApiPostCall({
  relatedQueryKeys: ["usersList", "tenantUsers"],
  onResult: (data) => {
    // Handle success
  }
});

// Bulk invalidation for major changes
const mutation = ApiPostCall({
  relatedQueryKeys: "*", // Invalidates all queries
});
```

### Error Boundaries

All state management errors are caught by the application-level ErrorBoundary:

```javascript
<ErrorBoundary FallbackComponent={Error500}>
  <PrivateRoute>{getLayout(<Component {...pageProps} />)}</PrivateRoute>
</ErrorBoundary>
```

### Context Usage

```javascript
// Settings Context hook
const settings = useSettings();

// Update settings
settings.handleUpdate({
  currentTenant: newTenant,
  customBranding: { colour: "#FF0000" }
});

// Reset to defaults
settings.handleReset();
```

This hybrid state management architecture provides CIPP with the flexibility to handle complex Microsoft 365 partner management scenarios while maintaining excellent performance and user experience.