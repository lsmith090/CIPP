# Performance Optimization Patterns

This document outlines the performance optimization techniques and patterns actually used in the CIPP codebase to ensure fast, responsive user experiences.

## React Performance Patterns

### 1. Component Memoization

#### React.memo for Component Optimization
CIPP uses `React.memo` to prevent unnecessary re-renders of expensive components:

```jsx
// Real implementation from CippFormComponent.jsx
const MemoizedCippAutoComplete = React.memo((props) => {
  return <CippAutoComplete {...props} />;
});

// Real implementation from CippAutocomplete.jsx  
const MemoTextField = React.memo(function MemoTextField({
  params,
  label,
  placeholder,
  ...otherProps
}) {
  const { InputProps, ...otherParams } = params;
  
  return (
    <TextField
      {...otherParams}
      label={label}
      placeholder={placeholder}
      {...otherProps}
      slotProps={{
        inputLabel: {
          shrink: true,
          sx: { transition: "none" },
          required: otherProps.required,
        },
        input: {
          ...InputProps,
          notched: true,
          sx: {
            transition: "none",
            "& .MuiOutlinedInput-notchedOutline": {
              transition: "none",
            },
          },
        },
      }}
    />
  );
});

// Real implementation from CippStandardAccordion.jsx
const CippAddedComponent = React.memo(({ standardName, component, formControl }) => {
  const updatedComponent = { ...component };
  
  if (component.type === "AdminRolesMultiSelect") {
    updatedComponent.type = "autoComplete";
    updatedComponent.options = GDAPRoles.map((role) => ({
      label: role.Name,
      value: role.ObjectId,
    }));
  }
  
  return (
    <Grid size={12}>
      <CippFormComponent
        type={updatedComponent.type}
        label={updatedComponent.label}
        formControl={formControl}
        {...updatedComponent}
        name={`${standardName}.${updatedComponent.name}`}
      />
    </Grid>
  );
});
```

### 2. Hook Optimization

#### useMemo for Expensive Computations
```jsx
// Real implementation from CippDataTable.js
const memoizedColumns = useMemo(() => usedColumns, [usedColumns]);
const memoizedData = useMemo(() => usedData, [usedData]);

// Real implementation from email contacts
const actions = useMemo(() => [
  {
    label: "Edit Contact",
    link: "/email/administration/contacts/edit?contactId=[id]",
    multiPost: false,
  },
  {
    label: "Delete Contact",
    link: "/api/removeContact",
    multiPost: true,
    confirm: true,
    color: "danger",
  },
], []);

const simpleColumns = useMemo(() => [
  "displayName",
  "mail", 
  "companyName",
  "onPremisesSync",
], []);
```

#### useCallback for Stable Function References
```jsx
// Real implementation from contact forms
const resetForm = useCallback(() => {
  formControl.reset();
  setFormData(defaultFormValues);
}, [formControl, defaultFormValues]);

const customDataFormatter = useCallback((values) => {
  const formattedValues = { ...values };
  // Custom formatting logic
  return formattedValues;
}, []);
```

## API Performance Patterns

### 1. Query Optimization

#### React Query Configuration
```jsx
// Real implementation from ApiCall.jsx
export function ApiGetCall(props) {
  const {
    url,
    queryKey,
    waiting = true,
    retry = 3,
    data,
    staleTime = 300000, // 5 minutes - actual default
    refetchOnWindowFocus = false,
    refetchOnMount = true,
    refetchOnReconnect = true,
    keepPreviousData = false,
  } = props;

  const queryInfo = useQuery({
    enabled: waiting,
    queryKey: [queryKey],
    queryFn: async ({ signal }) => {
      const response = await axios.get(url, {
        signal: url === "/api/tenantFilter" ? null : signal,
        params: data,
        headers: {
          "Content-Type": "application/json",
        },
      });
      return response.data;
    },
    staleTime: staleTime,
    refetchOnWindowFocus: refetchOnWindowFocus,
    refetchOnMount: refetchOnMount,
    refetchOnReconnect: refetchOnReconnect,
    keepPreviousData: keepPreviousData,
    retry: retryFn,
  });
  return queryInfo;
}
```

#### Pagination Implementation
```jsx
// Real implementation from ApiCall.jsx
export function ApiGetCallWithPagination(props) {
  const queryInfo = useInfiniteQuery({
    enabled: waiting,
    queryKey: [queryKey],
    queryFn: async ({ pageParam = null, signal }) => {
      const response = await axios.get(url, {
        signal: signal,
        params: { ...data, ...pageParam },
        headers: {
          "Content-Type": "application/json",
        },
      });
      return response.data;
    },
    getNextPageParam: (lastPage) => {
      if (
        data?.noPagination ||
        data?.manualPagination === false ||
        data?.tenantFilter === "AllTenants"
      ) {
        return undefined;
      }
      return lastPage?.Metadata?.nextLink ? { nextLink: lastPage.Metadata.nextLink } : undefined;
    },
    staleTime: 300000,
    refetchOnWindowFocus: false,
    retry: retryFn,
  });

  return queryInfo;
}
```

## UI Performance Patterns

### 1. Material-UI Grid v6 Optimization

#### Correct Grid Import and Usage
```jsx
// ✅ CORRECT - Import from @mui/system as per CLAUDE.md
import { Grid } from '@mui/system';

// ✅ CORRECT - MUI v6 Grid v2 syntax used throughout codebase
<Grid container spacing={3}>
  <Grid size={{ xs: 12, md: 6 }}>
    <CippFormComponent
      type="textField"
      label="Display Name"
      name="displayName"
      formControl={formControl}
    />
  </Grid>
  <Grid size={{ xs: 12, md: 6 }}>
    <CippFormComponent
      type="textField"
      label="Email Address"
      name="mail"
      formControl={formControl}
    />
  </Grid>
</Grid>

// ❌ AVOID - Old MUI syntax not used in CIPP
<Grid item xs={12} md={6}>
```

### 2. Data Table Performance

#### Material React Table Implementation
```jsx
// Real implementation from CippDataTable.js
import { MaterialReactTable, useMaterialReactTable } from "material-react-table";

export const CippDataTable = (props) => {
  const {
    data = [],
    columns = [],
    isFetching = false,
    exportEnabled = true,
    title = "Report",
  } = props;

  // Memoize processed data and columns
  const memoizedColumns = useMemo(() => usedColumns, [usedColumns]);
  const memoizedData = useMemo(() => usedData, [usedData]);

  const table = useMaterialReactTable({
    columns: memoizedColumns,
    data: memoizedData,
    state: {
      isLoading: isFetching,
      showProgressBars: isFetching,
    },
    enableExport: exportEnabled,
    // Additional MRT configuration
  });

  return <MaterialReactTable table={table} />;
};
```

## Bundle Optimization

### 1. Actual Dependencies Used

The CIPP codebase includes these performance-related dependencies:

```json
// From package.json
{
  "react": "19.0.0",
  "react-window": "^1.8.10",      // For virtual scrolling when needed
  "react-virtuoso": "^4.12.8",    // Alternative virtualization
  "material-react-table": "^3.0.1", // Optimized data tables
  "@tanstack/react-query": "^5.51.11", // Data fetching optimization
  "lodash.isequal": "4.5.0"        // Efficient deep comparison
}
```

### 2. Tree Shaking Optimization
```jsx
// ✅ Good: Import only what you need (used throughout codebase)
import { Box, Card, Typography } from '@mui/material';
import { format } from 'date-fns';
import { get } from 'lodash';

// ❌ Avoid: Importing entire libraries
import * as MUI from '@mui/material';
import * as DateFns from 'date-fns';
import _ from 'lodash';
```

## Form Performance

### 1. Form Field Optimization

```jsx
// Real pattern from CippFormComponent.jsx
export const CippFormComponent = (props) => {
  const { formControl, name, validators } = props;
  const { errors } = useFormState({ control: formControl.control });
  
  // Convert bracket notation to dot notation for form field names
  const convertedName = convertBracketsToDots(name);

  switch (type) {
    case "autoComplete":
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
  }
};
```

## Best Practices Summary

### ✅ Implemented Patterns in CIPP

1. **Use React.memo** for expensive form components and memoized text fields
2. **Memoize table data and columns** with useMemo in CippDataTable
3. **Use useCallback** for form reset and data formatting functions
4. **Configure React Query** with 5-minute staleTime for API calls
5. **Import selectively** from MUI and other libraries for tree shaking
6. **Use MUI v6 Grid syntax** with size prop throughout the application
7. **Leverage Material React Table** for optimized data display

### ❌ Avoid These Anti-patterns

1. **Don't create inline objects/functions** in JSX when passing to memoized components
2. **Don't use old MUI Grid syntax** (item, xs props) - use size prop instead
3. **Don't import entire libraries** - use specific imports for tree shaking
4. **Don't ignore React Query defaults** - leverage built-in caching

### Performance Checklist

- [ ] Form components use React.memo where beneficial (`CippFormComponent.jsx:40`)
- [ ] Table data is memoized in CippDataTable (`CippDataTable.js:192-193`)
- [ ] API calls use proper staleTime configuration (`ApiCall.jsx:220`)
- [ ] Grid components use MUI v6 syntax with size prop
- [ ] Imports are selective for better tree shaking
- [ ] Material React Table is used for data display
- [ ] Form validation uses react-hook-form patterns

## Component File References

- **CippFormComponent**: `/src/components/CippComponents/CippFormComponent.jsx`
- **CippAutocomplete**: `/src/components/CippComponents/CippAutocomplete.jsx`
- **CippDataTable**: `/src/components/CippTable/CippDataTable.js`
- **ApiCall utilities**: `/src/api/ApiCall.jsx`
- **CippStandardAccordion**: `/src/components/CippStandards/CippStandardAccordion.jsx`

Following these verified patterns ensures CIPP maintains fast, responsive user experiences based on the actual implementation.