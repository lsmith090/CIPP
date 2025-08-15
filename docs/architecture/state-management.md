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

## React Query Implementation Patterns

CIPP uses TanStack Query v5 with custom wrapper hooks for all server state management. This section covers the implementation details and advanced patterns.

### Custom API Hooks Implementation

#### ApiGetCall Hook Structure
```javascript
// Actual implementation pattern from ApiCall.jsx
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

  const queryClient = useQueryClient();
  const dispatch = useDispatch();

  const retryFn = (failureCount, error) => {
    // Custom retry logic based on HTTP status codes
    let returnRetry = false;
    
    if (error.response?.status === 401) {
      // Handle authentication errors
      queryClient.invalidateQueries({ queryKey: ['authmeswa'] });
      returnRetry = false;
    } else if (error.response?.status >= 500) {
      // Retry server errors
      returnRetry = failureCount < retry;
    }
    
    if (!returnRetry && toast) {
      dispatch(showToast({
        message: getCippError(error),
        title: `${data?.tenantFilter || ''} Error`,
        toastError: error,
      }));
    }
    
    return returnRetry;
  };

  const queryInfo = useQuery({
    queryKey: [queryKey],
    queryFn: async ({ signal }) => {
      if (bulkRequest) {
        return handleBulkRequest(url, data);
      }
      return apiRequest(url, data, signal);
    },
    enabled: !waiting,
    retry: retryFn,
    staleTime,
    refetchOnWindowFocus,
    refetchOnMount,
    refetchOnReconnect,
    keepPreviousData,
  });

  useEffect(() => {
    if (queryInfo.isSuccess && onResult) {
      onResult(queryInfo.data);
    }
  }, [queryInfo.isSuccess, queryInfo.data, onResult]);

  return queryInfo;
}
```

#### ApiPostCall Hook for Mutations
```javascript
export function ApiPostCall({ relatedQueryKeys, onResult }) {
  const queryClient = useQueryClient();
  const dispatch = useDispatch();

  const mutation = useMutation({
    mutationFn: async (props) => {
      const { url, data, bulkRequest } = props;
      
      if (bulkRequest) {
        return handleBulkMutation(url, data);
      }
      
      return apiRequest(url, data, null, 'POST');
    },
    onSuccess: (data, variables, context) => {
      // Automatic cache invalidation
      if (relatedQueryKeys) {
        const clearKeys = Array.isArray(relatedQueryKeys) 
          ? relatedQueryKeys 
          : [relatedQueryKeys];
        
        setTimeout(() => {
          if (relatedQueryKeys === "*") {
            queryClient.invalidateQueries();
          } else {
            clearKeys.forEach((key) => {
              queryClient.invalidateQueries({ queryKey: [key] });
            });
          }
        }, 1000);
      }
      
      if (onResult) {
        onResult(data, variables, context);
      }
    },
    onError: (error, variables, context) => {
      dispatch(showToast({
        message: getCippError(error),
        title: 'Operation Failed',
        toastError: error,
      }));
    },
  });

  return mutation;
}
```

#### Pagination Hook Implementation
```javascript
export function ApiGetCallWithPagination({
  url, queryKey, retry = 3, data, toast = false, waiting = true
}) {
  const dispatch = useDispatch();
  
  const queryInfo = useInfiniteQuery({
    queryKey: [queryKey],
    queryFn: async ({ pageParam = null, signal }) => {
      const requestData = pageParam 
        ? { ...data, nextLink: pageParam }
        : data;
        
      const response = await apiRequest(url, requestData, signal);
      
      return {
        data: response.Results || response,
        nextCursor: response.Metadata?.nextLink || null,
      };
    },
    getNextPageParam: (lastPage) => {
      return lastPage.nextCursor;
    },
    enabled: !waiting,
    retry: (failureCount, error) => {
      if (error.response?.status === 401) {
        return false;
      }
      return failureCount < retry;
    },
    initialPageParam: null,
  });

  return queryInfo;
}
```

### Advanced Query Patterns

#### Dependent Queries
```javascript
const UserWithPermissions = ({ userId }) => {
  // First query: Get user basic info
  const userQuery = ApiGetCall({
    url: '/api/users',
    queryKey: ['user', userId],
    data: { userId },
    waiting: !userId,
  });

  // Second query: Get user permissions (depends on first query)
  const permissionsQuery = ApiGetCall({
    url: '/api/user-permissions',
    queryKey: ['userPermissions', userId],
    data: { userId },
    waiting: !userQuery.isSuccess || !userQuery.data?.id,
  });

  if (userQuery.isLoading) return <LoadingSkeleton />;
  if (userQuery.isError) return <ErrorComponent error={userQuery.error} />;

  return (
    <UserCard 
      user={userQuery.data}
      permissions={permissionsQuery.data}
      isLoadingPermissions={permissionsQuery.isLoading}
    />
  );
};
```

#### Conditional Queries with Tenant Context
```javascript
const TenantAwareDataTable = () => {
  const { currentTenant } = useSettings();
  
  const dataQuery = ApiGetCall({
    url: '/api/tenant-data',
    queryKey: ['tenantData', currentTenant],
    data: { tenantFilter: currentTenant },
    waiting: !currentTenant || currentTenant === 'AllTenants',
    staleTime: 2 * 60 * 1000, // 2 minutes for tenant data
  });

  // Clear query when tenant changes
  useEffect(() => {
    if (currentTenant) {
      queryClient.removeQueries({ queryKey: ['tenantData'] });
    }
  }, [currentTenant]);

  return (
    <CippDataTable 
      data={dataQuery.data}
      isLoading={dataQuery.isLoading}
      refreshFunction={dataQuery.refetch}
    />
  );
};
```

## Redux Store Implementation Details

While CIPP uses Redux minimally, understanding the current implementation and patterns for extension is important.

### Current Redux Implementation

#### Store Configuration
```javascript
// src/store/index.js
import { configureStore } from '@reduxjs/toolkit'
import { rootReducer } from './root-reducer'
import { FLUSH, PAUSE, PERSIST, PURGE, REGISTER, REHYDRATE } from 'redux-persist'

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

#### Toast Slice Implementation
```javascript
// src/store/toasts.js - The primary Redux slice in CIPP
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
  toasts: [],
  currentIndex: 0,
};

export const toastsSlice = createSlice({
  name: "toasts",
  initialState,
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

export const { showToast, closeToast, resetToast } = toastsSlice.actions;
```

#### Integration with API Layer
```javascript
// Automatic toast integration in API calls
const retryFn = (failureCount, error) => {
  // ... retry logic
  
  if (returnRetry === false && toast) {
    dispatch(showToast({
      message: getCippError(error),
      title: `${tenantFilter} Error`,
      toastError: error,
    }));
  }
  
  return returnRetry;
};
```

### Extending Redux Store

For future Redux extensions, here's the recommended pattern:

```javascript
// src/store/user-preferences.js - Example extension
import { createSlice } from '@reduxjs/toolkit'

const initialState = {
  tablePageSize: 25,
  defaultFilters: {},
  favoritePages: [],
  notifications: {
    email: true,
    push: false,
    desktop: true,
  },
}

export const userPreferencesSlice = createSlice({
  name: 'userPreferences',
  initialState,
  reducers: {
    setTablePageSize: (state, action) => {
      state.tablePageSize = action.payload
    },
    updateNotificationSettings: (state, action) => {
      state.notifications = { ...state.notifications, ...action.payload }
    },
    addFavoritePage: (state, action) => {
      if (!state.favoritePages.includes(action.payload)) {
        state.favoritePages.push(action.payload)
      }
    },
    removeFavoritePage: (state, action) => {
      state.favoritePages = state.favoritePages.filter(
        page => page !== action.payload
      )
    },
  },
})
```

## React Context Implementation Details

The Settings Context is the primary React Context in CIPP, managing user preferences and application configuration.

### Settings Context Structure

```javascript
// src/contexts/settings-context.js
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
  isInitialized: false,
  handleReset: () => {},
  handleUpdate: () => {},
  isCustom: false,
});
```

### Persistent Storage Implementation

```javascript
// Memory fallback for environments without localStorage
class MemoryStorage {
  constructor() {
    this.store = new Map();
  }

  getItem(key) {
    return this.store.get(key);
  }

  setItem(key, value) {
    this.store.set(key, value);
  }

  removeItem(key) {
    this.store.delete(key);
  }
}

// Storage abstraction with fallback
let storage;
try {
  storage = globalThis.localStorage;
} catch (err) {
  console.error("[Settings Context] Local storage is not available", err);
  storage = new MemoryStorage();
}

const STORAGE_KEY = "app.settings";

const restoreSettings = () => {
  try {
    const restored = storage.getItem(STORAGE_KEY);
    return restored ? JSON.parse(restored) : null;
  } catch (err) {
    console.error("Failed to restore settings:", err);
    return null;
  }
};

const storeSettings = (value) => {
  try {
    storage.setItem(STORAGE_KEY, JSON.stringify(value));
  } catch (err) {
    console.error("Failed to store settings:", err);
  }
};
```

### Settings Provider Implementation

```javascript
export const SettingsProvider = (props) => {
  const { children } = props;
  const [state, setState] = useState({
    ...initialSettings,
    isInitialized: false,
  });

  // Restore settings from localStorage on mount
  useEffect(() => {
    const restored = restoreSettings();
    
    if (restored) {
      // Handle migration of old settings format
      if (!restored.currentTheme && restored.paletteMode) {
        restored.currentTheme = { 
          value: restored.paletteMode, 
          label: restored.paletteMode 
        };
      }

      setState((prevState) => ({
        ...prevState,
        ...restored,
        isInitialized: true,
      }));
    }
  }, []);

  const handleUpdate = useCallback((settings) => {
    setState((prevState) => {
      const newState = {
        ...prevState,
        ...settings,
      };
      
      storeSettings(newState);
      return newState;
    });
  }, []);

  const handleReset = useCallback(() => {
    deleteSettings();
    setState((prevState) => ({
      ...prevState,
      ...initialSettings,
    }));
  }, []);

  // Determine if settings are customized
  const isCustom = useMemo(() => {
    return !isEqual(initialSettings, {
      direction: state.direction,
      paletteMode: state.paletteMode,
      currentTheme: state.currentTheme,
      pinNav: state.pinNav,
    });
  }, [state]);

  return (
    <SettingsContext.Provider
      value={{
        ...state,
        handleReset,
        handleUpdate,
        isCustom,
      }}
    >
      {children}
    </SettingsContext.Provider>
  );
};
```

### Theme Integration with Settings

```javascript
// Theme provider that responds to settings changes
const AppThemeProvider = ({ children }) => {
  const { currentTheme, customBranding } = useSettings();
  
  const theme = useMemo(() => {
    return createTheme({
      palette: {
        mode: currentTheme.value,
        primary: {
          main: customBranding.colour,
        },
      },
      // ... other theme configuration
    });
  }, [currentTheme, customBranding]);

  return (
    <ThemeProvider theme={theme}>
      {children}
    </ThemeProvider>
  );
};
```

## Performance Optimization Patterns

### Memoization Strategies

```javascript
// Memoize expensive computations in data processing
const processedData = useMemo(() => {
  if (!rawData) return [];
  
  return rawData
    .filter(item => applyFilters(item, filters))
    .sort((a, b) => applySorting(a, b, sortConfig))
    .map(item => transformItem(item));
}, [rawData, filters, sortConfig]);

// Memoize callback functions passed to child components
const handleUserAction = useCallback((action, userId) => {
  switch (action) {
    case 'edit':
      // Handle edit
      break;
    case 'delete':
      // Handle delete
      break;
    default:
      console.warn('Unknown action:', action);
  }
}, [/* dependencies */]);
```

### Query Optimization

```javascript
// Use select to optimize re-renders
const userName = ApiGetCall({
  url: '/api/user',
  queryKey: ['user', userId],
  data: { userId },
  select: (data) => data.name, // Only re-render when name changes
});

// Prefetch related data
const UserProfile = ({ userId }) => {
  const queryClient = useQueryClient();
  
  const userQuery = ApiGetCall({
    url: '/api/user',
    queryKey: ['user', userId],
    data: { userId },
  });

  // Prefetch user permissions when hovering
  const handleMouseEnter = () => {
    queryClient.prefetchQuery({
      queryKey: ['userPermissions', userId],
      queryFn: () => fetchUserPermissions(userId),
      staleTime: 5 * 60 * 1000,
    });
  };

  return (
    <Card onMouseEnter={handleMouseEnter}>
      <UserCard user={userQuery.data} />
    </Card>
  );
};
```

### Context Performance Optimization

```javascript
// Split context to prevent unnecessary re-renders
const SettingsStateContext = createContext();
const SettingsActionsContext = createContext();

export const SettingsProvider = ({ children }) => {
  const [state, setState] = useState(initialSettings);
  
  // Memoize actions to prevent re-creation
  const actions = useMemo(() => ({
    handleUpdate: (settings) => {
      setState(prev => ({ ...prev, ...settings }));
    },
    handleReset: () => {
      setState(initialSettings);
    },
  }), []);
  
  return (
    <SettingsStateContext.Provider value={state}>
      <SettingsActionsContext.Provider value={actions}>
        {children}
      </SettingsActionsContext.Provider>
    </SettingsStateContext.Provider>
  );
};
```