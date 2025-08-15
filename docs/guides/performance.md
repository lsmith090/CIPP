# Performance Optimization Patterns

This document outlines the performance optimization techniques and patterns used in CIPP to ensure fast, responsive user experiences.

## React Performance Patterns

### 1. Component Memoization

#### React.memo for Component Optimization
CIPP uses `React.memo` to prevent unnecessary re-renders of expensive components:

```jsx
// Memoized component from CippFormComponent.jsx
const MemoizedCippAutoComplete = React.memo((props) => {
  return <CippAutoComplete {...props} />;
});

// Usage in forms
export const CippFormComponent = (props) => {
  // ... component logic
  
  return (
    <Controller
      name={convertedName}
      control={formControl.control}
      render={({ field }) => (
        <MemoizedCippAutoComplete
          {...field}
          label={label}
          error={!!get(errors, convertedName)}
          helperText={get(errors, convertedName)?.message || helperText}
          {...other}
        />
      )}
    />
  );
};
```

#### Custom Comparison Functions
For components with complex props, use custom comparison functions:

```jsx
const CippDataTableRow = React.memo(({ row, columns, actions, onAction }) => {
  return (
    <TableRow>
      {columns.map(column => (
        <TableCell key={column.accessorKey}>
          {column.cell ? column.cell({ row }) : row[column.accessorKey]}
        </TableCell>
      ))}
      <TableCell>
        <CippRowActions actions={actions} onAction={onAction} />
      </TableCell>
    </TableRow>
  );
}, (prevProps, nextProps) => {
  // Custom comparison logic
  return (
    prevProps.row.id === nextProps.row.id &&
    prevProps.actions.length === nextProps.actions.length &&
    isEqual(prevProps.row, nextProps.row)
  );
});
```

### 2. Hook Optimization

#### useMemo for Expensive Computations
```jsx
import { useMemo } from 'react';

const CippDataTable = ({ data, columns, filters }) => {
  // Memoize filtered and sorted data
  const processedData = useMemo(() => {
    let result = data || [];
    
    // Apply filters
    if (filters && Object.keys(filters).length > 0) {
      result = applyFilters(result, filters);
    }
    
    // Apply sorting
    result = applySort(result, sorting);
    
    return result;
  }, [data, filters, sorting]);

  // Memoize column configuration
  const enhancedColumns = useMemo(() => {
    return columns.map(column => ({
      ...column,
      cell: column.cell || (({ row }) => row[column.accessorKey]),
    }));
  }, [columns]);

  return (
    <MaterialReactTable
      data={processedData}
      columns={enhancedColumns}
    />
  );
};
```

#### useCallback for Stable Function References
```jsx
import { useCallback, useState } from 'react';

const CippUserManagement = ({ users, onUserUpdate }) => {
  const [selectedUsers, setSelectedUsers] = useState([]);

  // Memoize event handlers to prevent child re-renders
  const handleUserAction = useCallback((action, userId) => {
    switch (action) {
      case 'edit':
        // Handle edit action
        break;
      case 'disable':
        // Handle disable action
        break;
      default:
        console.warn('Unknown action:', action);
    }
  }, [onUserUpdate]);

  const handleUserSelect = useCallback((selectedUsers) => {
    setSelectedUsers(selectedUsers);
  }, []);

  return (
    <CippDataTable
      data={users}
      onAction={handleUserAction}
      onSelectionChange={handleUserSelect}
    />
  );
};
```

### 3. State Optimization

#### Minimize State Updates
```jsx
// ✅ Good: Batch related state updates
const [formState, setFormState] = useState({
  isSubmitting: false,
  error: null,
  data: {},
});

const handleSubmit = async (data) => {
  setFormState(prev => ({ ...prev, isSubmitting: true, error: null }));
  
  try {
    await submitData(data);
    setFormState(prev => ({ ...prev, isSubmitting: false }));
  } catch (error) {
    setFormState(prev => ({ 
      ...prev, 
      isSubmitting: false, 
      error: error.message 
    }));
  }
};

// ❌ Avoid: Multiple separate state updates
const [isSubmitting, setIsSubmitting] = useState(false);
const [error, setError] = useState(null);
const [data, setData] = useState({});

const handleSubmit = async (data) => {
  setIsSubmitting(true);  // Re-render 1
  setError(null);         // Re-render 2
  // ... rest of logic
};
```

#### Lazy Initial State
```jsx
// ✅ Good: Lazy initialization for expensive initial state
const [expensiveData, setExpensiveData] = useState(() => {
  return computeExpensiveInitialValue();
});

// ❌ Avoid: Computing on every render
const [expensiveData, setExpensiveData] = useState(computeExpensiveInitialValue());
```

## API Performance Patterns

### 1. Query Optimization

#### React Query Configuration
```jsx
import { ApiGetCall } from '/src/api/ApiCall';

const CippUserList = ({ tenantFilter }) => {
  const {
    data,
    isLoading,
    error,
    refetch
  } = ApiGetCall({
    url: '/api/listUsers',
    data: { TenantFilter: tenantFilter },
    queryKey: ['users', tenantFilter],
    staleTime: 5 * 60 * 1000, // 5 minutes
    gcTime: 10 * 60 * 1000, // 10 minutes (renamed in v5)
    refetchOnWindowFocus: false,
    retry: 3,
  });

  return (
    <CippDataTable
      data={data}
      isLoading={isLoading}
      error={error}
      refreshFunction={refetch}
    />
  );
};
```

#### Pagination for Large Datasets
```jsx
const CippPaginatedTable = ({ endpoint, pageSize = 25 }) => {
  const [page, setPage] = useState(0);
  
  const {
    data,
    isLoading,
    isFetching,
    hasNextPage,
    fetchNextPage
  } = ApiGetCallWithPagination({
    url: endpoint,
    data: { pageSize, page },
    queryKey: [endpoint, pageSize],
  });

  // Flatten paginated data
  const flatData = useMemo(() => {
    return data?.pages?.flatMap(page => page.Results) || [];
  }, [data]);

  return (
    <div>
      <CippDataTable
        data={flatData}
        isLoading={isLoading}
      />
      {hasNextPage && (
        <Button 
          onClick={fetchNextPage}
          disabled={isFetching}
        >
          Load More
        </Button>
      )}
    </div>
  );
};
```

### 2. Data Fetching Optimization

#### Debounced Search
```jsx
import { useMemo, useState, useEffect } from 'react';
import { debounce } from 'lodash';

const CippSearchableTable = ({ endpoint }) => {
  const [searchTerm, setSearchTerm] = useState('');
  const [debouncedSearchTerm, setDebouncedSearchTerm] = useState('');

  // Debounce search term updates
  const debouncedSetSearch = useMemo(
    () => debounce((term) => setDebouncedSearchTerm(term), 300),
    []
  );

  useEffect(() => {
    debouncedSetSearch(searchTerm);
    return () => debouncedSetSearch.cancel();
  }, [searchTerm, debouncedSetSearch]);

  const { data, isLoading } = ApiGetCall({
    url: endpoint,
    data: { search: debouncedSearchTerm },
    queryKey: [endpoint, debouncedSearchTerm],
    enabled: debouncedSearchTerm.length > 2, // Only search after 3 chars
  });

  return (
    <div>
      <TextField
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search..."
      />
      <CippDataTable data={data} isLoading={isLoading} />
    </div>
  );
};
```

#### Parallel Data Fetching
```jsx
const CippDashboard = () => {
  // Fetch multiple endpoints in parallel
  const usersQuery = ApiGetCall({
    url: '/api/users/count',
    queryKey: ['userCount'],
  });

  const tenantsQuery = ApiGetCall({
    url: '/api/tenants/count',
    queryKey: ['tenantCount'],
  });

  const alertsQuery = ApiGetCall({
    url: '/api/alerts/recent',
    queryKey: ['recentAlerts'],
  });

  const isLoading = usersQuery.isLoading || tenantsQuery.isLoading || alertsQuery.isLoading;

  return (
    <Grid container spacing={3}>
      <Grid item xs={12} md={4}>
        <CippInfoCard
          label="Users"
          value={usersQuery.data?.count}
          isLoading={usersQuery.isLoading}
        />
      </Grid>
      <Grid item xs={12} md={4}>
        <CippInfoCard
          label="Tenants"
          value={tenantsQuery.data?.count}
          isLoading={tenantsQuery.isLoading}
        />
      </Grid>
      <Grid item xs={12} md={8}>
        <CippAlertsList
          alerts={alertsQuery.data}
          isLoading={alertsQuery.isLoading}
        />
      </Grid>
    </Grid>
  );
};
```

## Rendering Performance

### 1. Virtual Scrolling for Large Lists
```jsx
import { FixedSizeList as List } from 'react-window';

const CippVirtualizedTable = ({ data, itemHeight = 50 }) => {
  const Row = ({ index, style }) => (
    <div style={style}>
      <CippTableRow data={data[index]} />
    </div>
  );

  return (
    <List
      height={600}
      itemCount={data.length}
      itemSize={itemHeight}
      width="100%"
    >
      {Row}
    </List>
  );
};

// Usage for large datasets
const CippLargeDataTable = ({ data }) => {
  if (data.length > 1000) {
    return <CippVirtualizedTable data={data} />;
  }
  
  return <CippDataTable data={data} />;
};
```

### 2. Lazy Loading Components
```jsx
import { lazy, Suspense } from 'react';

// Lazy load heavy components
const CippHeavyChart = lazy(() => import('./CippHeavyChart'));
const CippComplexReport = lazy(() => import('./CippComplexReport'));

const CippDashboard = () => {
  const [activeTab, setActiveTab] = useState(0);

  return (
    <div>
      <Tabs value={activeTab} onChange={(e, newValue) => setActiveTab(newValue)}>
        <Tab label="Overview" />
        <Tab label="Charts" />
        <Tab label="Reports" />
      </Tabs>

      {activeTab === 0 && <CippOverview />}
      
      {activeTab === 1 && (
        <Suspense fallback={<CippLoadingSkeleton />}>
          <CippHeavyChart />
        </Suspense>
      )}
      
      {activeTab === 2 && (
        <Suspense fallback={<CippLoadingSkeleton />}>
          <CippComplexReport />
        </Suspense>
      )}
    </div>
  );
};
```

### 3. Conditional Rendering Optimization
```jsx
const CippConditionalContent = ({ user, permissions, isLoading }) => {
  // Optimize conditional rendering with early returns
  if (isLoading) {
    return <CippLoadingSkeleton />;
  }

  if (!user) {
    return <CippEmptyState message="No user selected" />;
  }

  // Use logical AND for simple conditionals
  return (
    <div>
      <CippUserProfile user={user} />
      
      {permissions.includes('edit') && (
        <CippUserEditForm user={user} />
      )}
      
      {permissions.includes('admin') && (
        <CippAdminActions user={user} />
      )}
    </div>
  );
};
```

## Bundle Optimization

### 1. Tree Shaking Optimization
```jsx
// ✅ Good: Import only what you need
import { Box, Card, Typography } from '@mui/material';
import { format } from 'date-fns';
import { get } from 'lodash';

// ❌ Avoid: Importing entire libraries
import * as MUI from '@mui/material';
import * as DateFns from 'date-fns';
import _ from 'lodash';
```

### 2. Dynamic Imports for Code Splitting
```jsx
// Route-level code splitting
const CippUserManagement = lazy(() => 
  import('./pages/identity/administration/users')
);

const CippTenantManagement = lazy(() => 
  import('./pages/tenant/administration/tenants')
);

// Feature-level code splitting
const loadAdvancedFeatures = async () => {
  const module = await import('./advanced-features');
  return module.AdvancedFeatures;
};

const CippAdvancedSection = () => {
  const [AdvancedComponent, setAdvancedComponent] = useState(null);

  const handleLoadAdvanced = async () => {
    const Component = await loadAdvancedFeatures();
    setAdvancedComponent(() => Component);
  };

  return (
    <div>
      {!AdvancedComponent ? (
        <Button onClick={handleLoadAdvanced}>
          Load Advanced Features
        </Button>
      ) : (
        <AdvancedComponent />
      )}
    </div>
  );
};
```

## Form Performance

### 1. Optimized Form Validation
```jsx
import { useForm } from 'react-hook-form';

const CippOptimizedForm = ({ defaultValues, onSubmit }) => {
  const formControl = useForm({
    defaultValues,
    mode: 'onBlur', // Validate on blur instead of onChange
    reValidateMode: 'onBlur',
    shouldFocusError: true,
    shouldUnregister: false, // Keep values when unmounting
  });

  // Memoize validation rules
  const validationRules = useMemo(() => ({
    email: {
      required: 'Email is required',
      pattern: {
        value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
        message: 'Invalid email address'
      }
    },
    password: {
      required: 'Password is required',
      minLength: {
        value: 8,
        message: 'Password must be at least 8 characters'
      }
    },
  }), []);

  return (
    <form onSubmit={formControl.handleSubmit(onSubmit)}>
      <CippFormComponent
        formControl={formControl}
        name="email"
        label="Email"
        validators={validationRules.email}
      />
      <CippFormComponent
        formControl={formControl}
        name="password"
        label="Password"
        type="password"
        validators={validationRules.password}
      />
    </form>
  );
};
```

### 2. Form Field Optimization
```jsx
// Optimized form component with proper memoization
const CippOptimizedFormField = React.memo(({
  name,
  label,
  formControl,
  validators,
  ...props
}) => {
  const { errors } = useFormState({ 
    control: formControl.control,
    name, // Only subscribe to specific field changes
  });

  const fieldError = get(errors, name);

  return (
    <TextField
      {...formControl.register(name, validators)}
      label={label}
      error={!!fieldError}
      helperText={fieldError?.message}
      {...props}
    />
  );
}, (prevProps, nextProps) => {
  // Custom comparison to prevent unnecessary re-renders
  return (
    prevProps.name === nextProps.name &&
    prevProps.label === nextProps.label &&
    isEqual(prevProps.validators, nextProps.validators)
  );
});
```

## Performance Monitoring

### 1. React DevTools Profiler Integration
```jsx
// Development-only performance profiler
const CippPerformanceProfiler = ({ id, children }) => {
  if (process.env.NODE_ENV !== 'development') {
    return children;
  }

  return (
    <Profiler
      id={id}
      onRender={(id, phase, actualDuration, baseDuration, startTime, commitTime) => {
        console.log('Performance:', {
          id,
          phase,
          actualDuration,
          baseDuration,
          startTime,
          commitTime,
        });
      }}
    >
      {children}
    </Profiler>
  );
};

// Usage
const CippDashboard = () => (
  <CippPerformanceProfiler id="Dashboard">
    <DashboardContent />
  </CippPerformanceProfiler>
);
```

### 2. Custom Performance Hooks
```jsx
import { useEffect, useRef } from 'react';

const usePerformanceMetrics = (componentName) => {
  const startTime = useRef(performance.now());
  const renderCount = useRef(0);

  useEffect(() => {
    renderCount.current += 1;
  });

  useEffect(() => {
    const endTime = performance.now();
    const renderTime = endTime - startTime.current;
    
    if (process.env.NODE_ENV === 'development') {
      console.log(`${componentName} render metrics:`, {
        renderTime: `${renderTime.toFixed(2)}ms`,
        renderCount: renderCount.current,
      });
    }
  });
};

// Usage
const CippExpensiveComponent = () => {
  usePerformanceMetrics('CippExpensiveComponent');
  
  // Component implementation
};
```

## Best Practices Summary

### ✅ Do's
1. **Use React.memo** for components that render frequently with the same props
2. **Memoize expensive computations** with useMemo
3. **Memoize callback functions** with useCallback when passing to child components
4. **Implement virtual scrolling** for large datasets
5. **Use lazy loading** for heavy components and routes
6. **Debounce user inputs** for search and filtering
7. **Batch state updates** when updating multiple related values
8. **Use proper query configurations** with staleTime and gcTime
9. **Import only what you need** for better tree shaking

### ❌ Don'ts
1. **Don't memoize everything** - only memoize when there's a measurable benefit
2. **Don't use inline objects/functions** in JSX when passing to memoized components
3. **Don't fetch data in render** - use proper data fetching hooks
4. **Don't create new objects/arrays** in render without memoization
5. **Don't ignore React DevTools warnings** about performance
6. **Don't load all data at once** for large datasets
7. **Don't use complex logic** in render functions

### Performance Checklist
- [ ] Components are properly memoized where beneficial
- [ ] Expensive computations use useMemo
- [ ] Event handlers use useCallback
- [ ] Large lists use virtual scrolling
- [ ] Heavy components are lazy loaded
- [ ] API calls are optimized with proper caching
- [ ] Bundle size is monitored and optimized
- [ ] Images and assets are lazy loaded
- [ ] Form validation is optimized
- [ ] Performance metrics are monitored in development

Following these performance patterns ensures CIPP maintains fast, responsive user experiences across all devices and network conditions.