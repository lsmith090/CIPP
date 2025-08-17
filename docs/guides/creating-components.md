# Component Development Guidelines

This guide covers how to create, organize, and maintain components in the CIPP frontend application, following established patterns and best practices.

## Component Organization

CIPP components are organized into logical categories under `/src/components/`:

```
/src/components/
├── CippCards/           # Card-based display components
├── CippComponents/      # Core functional components  
├── CippFormPages/       # Full-page form components
├── CippIntegrations/    # Integration-specific components
├── CippSettings/        # Settings and configuration components
├── CippStandards/       # Standards and compliance components
├── CippTable/           # Table and data grid components
├── CippWizard/          # Multi-step wizard components
└── [individual files]   # Standalone utility components
```

### Category Guidelines

**CippCards/**: Display components that present data in card format
- Info cards, banner cards, property list cards
- Focus on data presentation, minimal interaction
- Usually consume API data and display it formatted

**CippComponents/**: Core functional components used across the application
- Form components, dialogs, selectors, actions
- Reusable business logic components
- Integration with CIPP's API and state management

**CippTable/**: Data table and grid components
- Table functionality, filtering, sorting, pagination
- Data export, bulk actions, column management

**CippWizard/**: Multi-step processes and workflows
- Step-based interfaces for complex operations
- Wizard navigation and state management

## Development Standards

### Component Structure

Follow this standard component structure:

```javascript
import React, { useState, useCallback, useMemo } from 'react';
import PropTypes from 'prop-types';
import {
  Card,
  CardContent,
  Typography,
  Button,
  CircularProgress
} from '@mui/material';
import { ApiGetCall } from '/src/api/ApiCall';
import { useSettings } from '/src/hooks/use-settings';

/**
 * Component description - what it does and when to use it
 * 
 * @param {Object} props - Component props
 * @param {string} props.title - Card title
 * @param {boolean} props.loading - Loading state override
 * @param {Function} props.onAction - Action button callback
 */
export const CippExampleCard = (props) => {
  const {
    title = "Default Title",
    loading = false,
    onAction,
    children,
    ...other
  } = props;

  // Hooks
  const settings = useSettings();
  const [localState, setLocalState] = useState(null);

  // API calls
  const { data, isLoading, isError } = ApiGetCall({
    url: "/api/GetData",
    queryKey: "exampleData",
    waiting: !loading // Don't fetch if externally loading
  });

  // Computed values
  const isLoadingState = useMemo(() => {
    return loading || isLoading;
  }, [loading, isLoading]);

  // Event handlers
  const handleAction = useCallback(() => {
    if (onAction) {
      onAction(data);
    }
  }, [onAction, data]);

  // Render
  if (isError) {
    return (
      <Card {...other}>
        <CardContent>
          <Typography color="error">Error loading data</Typography>
        </CardContent>
      </Card>
    );
  }

  return (
    <Card {...other}>
      <CardContent>
        <Typography variant="h6" gutterBottom>
          {title}
        </Typography>
        
        {isLoadingState ? (
          <CircularProgress size={24} />
        ) : (
          <>
            <Typography variant="body2" color="text.secondary">
              {data?.description || "No description available"}
            </Typography>
            {children}
          </>
        )}
        
        {onAction && (
          <Button
            variant="contained"
            onClick={handleAction}
            disabled={isLoadingState}
            sx={{ mt: 2 }}
          >
            Take Action
          </Button>
        )}
      </CardContent>
    </Card>
  );
};

// PropTypes for development
CippExampleCard.propTypes = {
  title: PropTypes.string,
  loading: PropTypes.bool,
  onAction: PropTypes.func,
  children: PropTypes.node,
};
```

### Material-UI Integration

CIPP uses Material-UI with custom theming. Follow these patterns:

#### Component Styling

```javascript
import { styled } from '@mui/material/styles';
import { Card, alpha } from '@mui/material';

// Styled components for complex styling
const StyledCard = styled(Card)(({ theme }) => ({
  position: 'relative',
  backgroundColor: alpha(theme.palette.primary.main, 0.05),
  '&:hover': {
    backgroundColor: alpha(theme.palette.primary.main, 0.1),
  },
  '& .MuiCardContent-root': {
    padding: theme.spacing(3),
  }
}));

// Inline sx prop for simple styling
const SimpleCard = (props) => (
  <Card
    sx={{
      p: 2,
      borderRadius: 2,
      '&:hover': {
        boxShadow: (theme) => theme.shadows[4]
      }
    }}
    {...props}
  />
);
```

#### Theme Integration

```javascript
import { useTheme } from '@mui/material/styles';
import { useMediaQuery } from '@mui/material';

const ResponsiveComponent = () => {
  const theme = useTheme();
  const isMobile = useMediaQuery(theme.breakpoints.down('md'));
  
  return (
    <Card
      sx={{
        flexDirection: isMobile ? 'column' : 'row',
        backgroundColor: theme.palette.mode === 'dark' 
          ? 'grey.900' 
          : 'background.paper'
      }}
    >
      {/* Component content */}
    </Card>
  );
};
```

### Props Patterns

#### Standard Props Structure

```javascript
export const CippComponent = (props) => {
  const {
    // Required props first
    data,
    onAction,
    
    // Optional props with defaults
    title = "Default Title",
    variant = "outlined",
    size = "medium",
    disabled = false,
    loading = false,
    
    // Event handlers
    onClick,
    onClose,
    onSubmit,
    
    // Style and layout props
    sx,
    className,
    
    // Children and content
    children,
    
    // Rest props for passthrough
    ...other
  } = props;

  // Component logic...
};
```

#### Advanced Props Patterns

```javascript
// Union types for controlled variants
const CippStatusCard = ({ status, ...props }) => {
  const variants = {
    success: { color: 'success', icon: CheckIcon },
    warning: { color: 'warning', icon: WarningIcon },
    error: { color: 'error', icon: ErrorIcon },
    info: { color: 'info', icon: InfoIcon }
  };
  
  const config = variants[status] || variants.info;
  // Use config...
};

// Flexible data structure props
const CippPropertyList = ({ items, ...props }) => {
  // Handle both array and object formats
  const processedItems = useMemo(() => {
    if (Array.isArray(items)) {
      return items;
    }
    if (typeof items === 'object') {
      return Object.entries(items).map(([key, value]) => ({ key, value }));
    }
    return [];
  }, [items]);
  
  // Render processed items...
};
```

### Performance Optimization

#### Memoization Patterns

```javascript
import React, { memo, useMemo, useCallback } from 'react';

// Memoize expensive calculations
const CippDataCard = memo(({ data, onFilter }) => {
  const processedData = useMemo(() => {
    return data?.map(item => ({
      ...item,
      computed: expensiveCalculation(item)
    })) || [];
  }, [data]);

  const handleFilter = useCallback((filterValue) => {
    onFilter?.(processedData.filter(item => 
      item.name.toLowerCase().includes(filterValue.toLowerCase())
    ));
  }, [processedData, onFilter]);

  return (
    <Card>
      {/* Render processed data */}
    </Card>
  );
});

// Memo comparison for complex props
const CippComplexCard = memo(({ config, data, ...props }) => {
  // Component logic...
}, (prevProps, nextProps) => {
  // Custom comparison logic
  return (
    prevProps.data === nextProps.data &&
    JSON.stringify(prevProps.config) === JSON.stringify(nextProps.config)
  );
});
```

#### Lazy Loading

```javascript
import { lazy, Suspense } from 'react';
import { CircularProgress } from '@mui/material';

// Lazy load heavy components
const CippAdvancedChart = lazy(() => import('./CippAdvancedChart'));

const CippDashboard = () => (
  <div>
    <Suspense fallback={<CircularProgress />}>
      <CippAdvancedChart />
    </Suspense>
  </div>
);
```

## Component Templates

### Basic Info Card Template

```javascript
import { Card, CardContent, Typography, Avatar, Stack, SvgIcon } from '@mui/material';
import { CubeIcon } from '@heroicons/react/24/outline';

export const CippInfoCard = ({ 
  label, 
  value, 
  icon, 
  loading = false,
  ...other 
}) => (
  <Card {...other}>
    <Stack alignItems="center" direction="row" spacing={2} sx={{ p: 2 }}>
      <Avatar sx={{ backgroundColor: 'primary.alpha12', color: 'primary.main' }}>
        <SvgIcon fontSize="small">
          {icon || <CubeIcon />}
        </SvgIcon>
      </Avatar>
      <div>
        <Typography color="text.secondary" variant="overline">
          {loading ? <Skeleton width={150} /> : label}
        </Typography>
        <Typography variant="h6">
          {loading ? <Skeleton width={200} /> : value}
        </Typography>
      </div>
    </Stack>
  </Card>
);
```

### Form Component Template

```javascript
import { Controller, useFormState } from 'react-hook-form';
import { TextField, FormHelperText } from '@mui/material';

export const CippFormField = ({ 
  name, 
  label, 
  control, 
  validators, 
  type = "text",
  ...other 
}) => {
  const { errors } = useFormState({ control });
  const error = errors[name];

  return (
    <Controller
      name={name}
      control={control}
      rules={validators}
      render={({ field }) => (
        <>
          <TextField
            {...field}
            label={label}
            type={type}
            error={!!error}
            fullWidth
            variant="outlined"
            {...other}
          />
          {error && (
            <FormHelperText error>
              {error.message}
            </FormHelperText>
          )}
        </>
      )}
    />
  );
};
```

### Action Button Template

```javascript
import { Button, CircularProgress } from '@mui/material';
import { ApiPostCall } from '/src/api/ApiCall';

export const CippActionButton = ({ 
  action, 
  data, 
  onSuccess, 
  children,
  ...other 
}) => {
  const mutation = ApiPostCall({
    relatedQueryKeys: ['refreshData'],
    onResult: onSuccess
  });

  const handleClick = () => {
    mutation.mutate({
      url: action,
      data: data
    });
  };

  return (
    <Button
      onClick={handleClick}
      disabled={mutation.isPending}
      startIcon={mutation.isPending ? <CircularProgress size={16} /> : null}
      {...other}
    >
      {children}
    </Button>
  );
};
```

### Dialog Component Template

```javascript
import { 
  Dialog, 
  DialogTitle, 
  DialogContent, 
  DialogActions, 
  Button, 
  IconButton 
} from '@mui/material';
import { XMarkIcon } from '@heroicons/react/24/outline';

export const CippDialog = ({ 
  open, 
  onClose, 
  title, 
  children, 
  actions,
  maxWidth = "sm",
  ...other 
}) => (
  <Dialog 
    open={open} 
    onClose={onClose} 
    maxWidth={maxWidth} 
    fullWidth 
    {...other}
  >
    <DialogTitle sx={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
      {title}
      <IconButton onClick={onClose} size="small">
        <XMarkIcon />
      </IconButton>
    </DialogTitle>
    
    <DialogContent>
      {children}
    </DialogContent>
    
    {actions && (
      <DialogActions sx={{ px: 3, pb: 3 }}>
        {actions}
      </DialogActions>
    )}
  </Dialog>
);
```

## Integration Patterns

### API Integration

```javascript
import { ApiGetCall, ApiPostCall } from '/src/api/ApiCall';
import { useSettings } from '/src/hooks/use-settings';

export const CippTenantComponent = () => {
  const { currentTenant } = useSettings();
  
  // GET data
  const { data, isLoading, isError } = ApiGetCall({
    url: "/api/ListData",
    queryKey: ["tenantData", currentTenant],
    data: { tenantFilter: currentTenant },
    waiting: !!currentTenant // Only fetch when tenant is selected
  });

  // POST mutations
  const updateMutation = ApiPostCall({
    relatedQueryKeys: ["tenantData"],
    onResult: (result) => {
      // Handle success
    }
  });

  const handleUpdate = (newData) => {
    updateMutation.mutate({
      url: "/api/UpdateData",
      data: { ...newData, tenantFilter: currentTenant }
    });
  };

  // Component render logic...
};
```

### Permission Integration

```javascript
import { usePermissions } from '/src/hooks/use-permissions';
import { PermissionCheck } from '/src/utils/permissions';

export const CippAdminComponent = () => {
  const { checkPermissions } = usePermissions();
  const canEdit = checkPermissions(['admin.write', 'tenant.*.write']);

  return (
    <Card>
      <CardContent>
        {/* Always visible content */}
        <Typography>Public information</Typography>
        
        {/* Permission-gated content */}
        <PermissionCheck requiredPermissions={['admin.read']}>
          <Typography>Admin-only information</Typography>
        </PermissionCheck>
        
        {/* Conditional rendering */}
        {canEdit && (
          <Button variant="contained">
            Edit Data
          </Button>
        )}
      </CardContent>
    </Card>
  );
};
```

### Settings Integration

```javascript
import { useSettings } from '/src/hooks/use-settings';

export const CippPreferencesComponent = () => {
  const settings = useSettings();
  
  return (
    <Card>
      <CardContent>
        {/* Use current tenant */}
        <Typography>Current Tenant: {settings.currentTenant}</Typography>
        
        {/* Theme-aware styling */}
        <Box
          sx={{
            backgroundColor: settings.paletteMode === 'dark' 
              ? 'grey.800' 
              : 'grey.100'
          }}
        >
          Theme-aware content
        </Box>
        
        {/* Update settings */}
        <Button 
          onClick={() => settings.handleDrawerToggle()}
        >
          Toggle Sidebar
        </Button>
      </CardContent>
    </Card>
  );
};
```

## Best Practices

### 1. Component Naming
- Use PascalCase with "Cipp" prefix: `CippUserCard`, `CippDataTable`
- Be descriptive: `CippUserPermissionsDialog` vs `CippDialog`
- Group related components: `CippFormField`, `CippFormSection`, `CippFormPage`

### 2. Props Design
- Use destructuring with defaults
- Group props logically (required, optional, events, styling)
- Provide PropTypes for development
- Support passthrough props with `...other`

### 3. State Management
- Use local state for component-specific UI state
- Use API hooks for server state
- Use settings context for global preferences
- Lift state up when needed by multiple components

### 4. Error Handling
- Handle API errors gracefully
- Provide fallback UI for error states
- Use error boundaries for unexpected errors
- Show appropriate loading states

### 5. Performance
- Memoize expensive calculations
- Use React.memo for pure components
- Lazy load heavy components
- Avoid unnecessary re-renders

### 6. Accessibility
- Use semantic HTML elements
- Provide proper ARIA labels
- Support keyboard navigation
- Ensure proper color contrast

### 7. Testing Considerations
- Write components that are easy to test
- Separate business logic from presentation
- Use data attributes for test selectors
- Mock external dependencies

## Common Anti-Patterns to Avoid

1. **Prop drilling**: Use context or state management instead
2. **Massive components**: Break down into smaller, focused components
3. **Direct DOM manipulation**: Use React patterns and refs appropriately
4. **Inline object creation**: Memoize objects and functions
5. **Missing error boundaries**: Always handle potential failures
6. **Inconsistent styling**: Follow the established theme and patterns
7. **Hard-coded values**: Use theme tokens and configuration

This guide provides the foundation for creating consistent, maintainable components in CIPP. For specific component types, refer to the individual component documentation and examples in the codebase.