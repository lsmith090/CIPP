# CIPP Custom Hooks Documentation

This document provides comprehensive documentation for all custom React hooks used in the CIPP application. These hooks encapsulate common functionality and provide reusable logic for components.

## Table of Contents

1. [Authentication & User Management](#authentication--user-management)
2. [Settings & Configuration](#settings--configuration)
3. [UI State Management](#ui-state-management)
4. [Data Processing](#data-processing)
5. [Utilities](#utilities)
6. [Best Practices](#best-practices)

## Authentication & User Management

### useAuth

Simple wrapper hook that provides access to the authentication context.

```javascript
import { useAuth } from '/src/hooks/use-auth';

const MyComponent = () => {
  const auth = useAuth();
  
  // This hook is a simple context wrapper
  return <div>Auth context: {JSON.stringify(auth)}</div>;
};
```

**Implementation:**
```javascript
export const useAuth = () => useContext(AuthContext);
```

**Return Value:**
Returns whatever the AuthContext provides (context-dependent).

**Usage:**
- Access authentication context state
- Simple wrapper for AuthContext consumption
- Integrates with CIPP's Azure AD authentication system

### usePermissions

Comprehensive hook for checking user permissions and roles with pattern matching support.

```javascript
import { usePermissions, useHasPermission } from '/src/hooks/use-permissions';

const MyComponent = () => {
  const { 
    userPermissions, 
    userRoles, 
    isLoading, 
    isAuthenticated,
    checkPermissions,
    checkRoles,
    checkAccess 
  } = usePermissions();

  // Check specific permissions
  const canEditUsers = checkPermissions(['Identity.User.ReadWrite']);
  
  // Check roles
  const isAdmin = checkRoles(['admin', 'superadmin']);
  
  // Combined access check
  const hasAccess = checkAccess({
    requiredPermissions: ['Identity.*.Read'],
    requiredRoles: ['admin']
  });

  return (
    <div>
      {canEditUsers && <EditButton />}
      {isAdmin && <AdminPanel />}
    </div>
  );
};

// Simplified permission check hook
const ComponentWithSimpleCheck = () => {
  const { hasAccess, isLoading } = useHasPermission(
    ['Identity.User.ReadWrite'],  // Required permissions
    ['admin']                     // Required roles
  );

  if (isLoading) return <Spinner />;
  if (!hasAccess) return <AccessDenied />;
  
  return <ProtectedContent />;
};
```

**Methods:**

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `checkPermissions` | `string[]` | `boolean` | Check if user has any of the specified permissions (supports wildcards) |
| `checkRoles` | `string[]` | `boolean` | Check if user has any of the specified roles |
| `checkAccess` | `{ requiredPermissions, requiredRoles }` | `boolean` | Combined permission and role check |

**Permission Patterns:**
- Exact match: `"Identity.User.Read"`
- Wildcard: `"Identity.*"` (matches all Identity permissions)
- Multiple: `["Identity.User.Read", "Identity.User.ReadWrite"]`

## Settings & Configuration

### useSettings

Provides access to application settings and configuration.

```javascript
import { useSettings } from '/src/hooks/use-settings';

const MyComponent = () => {
  const settings = useSettings();
  
  const {
    currentTenant,        // Currently selected tenant
    paletteMode,          // 'light' or 'dark'
    pinNav,               // Navigation pinned state
    customBranding,       // Custom branding settings
    // ... other settings
  } = settings;

  return (
    <div>
      <h1>Current Tenant: {currentTenant}</h1>
      <button onClick={() => settings.handleUpdate({ paletteMode: 'dark' })}>
        Switch to Dark Mode
      </button>
    </div>
  );
};
```

**Key Properties:**
- `currentTenant` - Selected tenant filter
- `paletteMode` - Theme mode ('light'/'dark')
- `pinNav` - Navigation drawer state
- `customBranding` - Branding configuration
- `showDevtools` - Development tools visibility

**Methods:**
- `handleUpdate(newSettings)` - Update settings
- `handleDrawerToggle()` - Toggle navigation drawer
- `handleReset()` - Reset to default settings

## UI State Management

### useDialog

Manages dialog/modal open/close state and associated data.

```javascript
import { useDialog } from '/src/hooks/use-dialog';

const MyComponent = () => {
  const dialog = useDialog();

  const handleOpenDialog = () => {
    dialog.handleOpen({ 
      userId: '123',
      action: 'edit'
    });
  };

  return (
    <>
      <button onClick={handleOpenDialog}>
        Edit User
      </button>
      
      <Dialog open={dialog.open} onClose={dialog.handleClose}>
        <DialogContent>
          {dialog.data && (
            <UserForm 
              userId={dialog.data.userId} 
              action={dialog.data.action}
            />
          )}
        </DialogContent>
      </Dialog>
    </>
  );
};
```

**Return Value:**
```javascript
{
  open: boolean,           // Dialog open state
  data: any,              // Data passed to handleOpen (persists after close)
  handleOpen: (data?) => void,   // Open dialog with optional data
  handleClose: () => void        // Close dialog (data persists until next handleOpen)
}
```

### usePopover

Manages popover/menu positioning and state.

```javascript
import { usePopover } from '/src/hooks/use-popover';

const MyComponent = () => {
  const popover = usePopover();

  return (
    <>
      <IconButton 
        ref={popover.anchorRef}
        onClick={popover.handleToggle}
      >
        <MoreVertIcon />
      </IconButton>
      
      <Popover
        open={popover.open}
        anchorEl={popover.anchorRef.current}
        onClose={popover.handleClose}
        anchorOrigin={{
          vertical: 'bottom',
          horizontal: 'right',
        }}
      >
        <MenuList>
          <MenuItem onClick={popover.handleClose}>Action 1</MenuItem>
          <MenuItem onClick={popover.handleClose}>Action 2</MenuItem>
        </MenuList>
      </Popover>
    </>
  );
};
```

**Return Value:**
```javascript
{
  anchorRef: RefObject,         // Reference for popover anchor
  open: boolean,               // Popover open state
  handleOpen: () => void,      // Open popover
  handleClose: () => void,     // Close popover
  handleToggle: () => void     // Toggle popover state
}
```

### useSelection

Manages multi-item selection state with helper methods.

```javascript
import { useSelection } from '/src/hooks/use-selection';

const UserListComponent = () => {
  const [users, setUsers] = useState([]);
  const selection = useSelection(users);

  return (
    <div>
      <div>
        <Checkbox
          checked={selection.selected.length === users.length}
          indeterminate={selection.selected.length > 0 && selection.selected.length < users.length}
          onChange={(event) => {
            if (event.target.checked) {
              selection.handleSelectAll();
            } else {
              selection.handleDeselectAll();
            }
          }}
        />
        Select All ({selection.selected.length} selected)
      </div>
      
      {users.map((user) => (
        <div key={user.id}>
          <Checkbox
            checked={selection.selected.includes(user)}
            onChange={(event) => {
              if (event.target.checked) {
                selection.handleSelectOne(user);
              } else {
                selection.handleDeselectOne(user);
              }
            }}
          />
          {user.name}
        </div>
      ))}
      
      {selection.selected.length > 0 && (
        <Button onClick={() => handleBulkAction(selection.selected)}>
          Process Selected ({selection.selected.length})
        </Button>
      )}
    </div>
  );
};
```

**Parameters:**
- `items` - Array of items to manage selection for

**Return Value:**
```javascript
{
  selected: any[],                    // Currently selected items
  handleSelectAll: () => void,        // Select all items
  handleDeselectAll: () => void,      // Deselect all items  
  handleSelectOne: (item) => void,    // Select single item
  handleDeselectOne: (item) => void   // Deselect single item
}
```

### useFilters

Advanced filtering system with validation and state management.

```javascript
import { useFilters } from '/src/hooks/use-filters';

const FilterableTable = () => {
  const operators = [
    { name: 'contains', label: 'Contains' },
    { name: 'equals', label: 'Equals' },
    { name: 'isBlank', label: 'Is Blank' },
    { name: 'isPresent', label: 'Is Present' }
  ];

  const properties = [
    { name: 'displayName', label: 'Display Name' },
    { name: 'department', label: 'Department' },
    { name: 'jobTitle', label: 'Job Title' }
  ];

  const filters = useFilters(operators, properties);

  return (
    <div>
      <FilterBuilder
        filters={filters.filters}
        operators={operators}
        properties={properties}
        onAddFilter={filters.handleFilterAdd}
        onRemoveFilter={filters.handleFilterRemove}
        onOperatorChange={filters.handleOperatorChange}
        onPropertyChange={filters.handlePropertyChange}
        onValueChange={filters.handleValueChange}
        onClear={filters.handleFiltersClear}
      />
      
      <div>
        Valid filters: {filters.valid ? 'Yes' : 'No'}
        <button 
          disabled={!filters.valid}
          onClick={() => applyFilters(filters.filters)}
        >
          Apply Filters
        </button>
      </div>
    </div>
  );
};
```

**Parameters:**
- `operators` - Array of available filter operators
- `properties` - Array of filterable properties
- `initialFilters` - Initial filter configuration

**Return Value:**
```javascript
{
  filters: Array,                                    // Current filter configuration
  valid: boolean,                                   // Whether all filters are valid
  handleFilterAdd: (index) => void,                 // Add filter at index
  handleFilterRemove: (index) => void,              // Remove filter at index
  handleFiltersClear: () => void,                   // Clear all filters
  handleOperatorChange: (index, operator) => void,  // Change filter operator
  handlePropertyChange: (index, property) => void,  // Change filter property
  handleValueChange: (index, value) => void         // Change filter value
}
```

**Filter Structure:**
```javascript
{
  operator: 'contains',    // Filter operator
  property: 'displayName', // Property to filter on
  value: 'John'           // Filter value (undefined for isBlank/isPresent)
}
```

## Data Processing

### useGuidResolver

Complex hook for resolving GUIDs to human-readable names with support for partner tenants and batch processing.

```javascript
import { useGuidResolver } from '/src/hooks/use-guid-resolver';

const AuditLogComponent = () => {
  const guidResolver = useGuidResolver();

  const auditLog = {
    id: "550e8400-e29b-41d4-a716-446655440000",
    activity: "User modified",
    target: "user_a1b2c3d4e5f6@partner.onmicrosoft.com",
    initiatedBy: "admin@tenant.onmicrosoft.com"
  };

  // Scan object for GUIDs and trigger resolution
  useEffect(() => {
    guidResolver.resolveGuids(auditLog);
  }, [auditLog, guidResolver.resolveGuids]);

  // Replace GUIDs in strings with resolved names
  const processedActivity = guidResolver.replaceGuidsAndUpnsInString(auditLog.activity);
  const processedTarget = guidResolver.replaceGuidsAndUpnsInString(auditLog.target);

  return (
    <div>
      {guidResolver.isLoadingGuids && <Spinner />}
      <div>ID: {guidResolver.guidMapping[auditLog.id] || auditLog.id}</div>
      <div>Activity: {processedActivity.result}</div>
      <div>Target: {processedTarget.result}</div>
    </div>
  );
};

// Using with custom tenant
const CrossTenantComponent = () => {
  const resolver = useGuidResolver("custom-tenant-id");
  // Hook will use specified tenant instead of current tenant
};
```

**Parameters:**
- `manualTenant` (optional) - Specific tenant to use instead of current tenant

**Return Value:**
```javascript
{
  guidMapping: Object,                           // GUID -> display name mapping
  upnMapping: Object,                           // GUID -> UPN mapping  
  isLoadingGuids: boolean,                      // Loading state
  resolveGuids: (object) => void,               // Scan object and resolve GUIDs
  isGuid: (string) => boolean,                  // Check if string is GUID
  extractObjectIdFromPartnerUPN: (string) => Array, // Extract GUIDs from partner UPNs
  replaceGuidsAndUpnsInString: (string) => {    // Replace GUIDs with names
    result: string,
    hasResolvedNames: boolean
  }
}
```

**GUID Resolution Features:**
- Automatic GUID detection in objects and strings
- Partner tenant GUID resolution
- Batch processing with rate limiting
- Intelligent retry logic for API rate limits
- Support for embedded GUIDs in text
- Special handling for partner UPN formats

### useSecureScore

Fetches and processes Microsoft Secure Score data with CIPP standards integration.

```javascript
import { useSecureScore } from '/src/hooks/use-securescore';

const SecureScoreComponent = () => {
  const {
    controlScore,      // Control score API data
    secureScore,       // Secure score API data  
    translatedData,    // Processed data with translations
    isFetching,        // Loading state
    isSuccess         // Success state
  } = useSecureScore();

  if (isFetching) return <Spinner />;
  if (!isSuccess) return <ErrorMessage />;

  return (
    <div>
      <h2>Secure Score Overview</h2>
      <div>
        Current Score: {translatedData.currentScore} / {translatedData.maxScore}
        ({translatedData.percentageCurrent}%)
      </div>
      
      <div>
        vs All Tenants: {translatedData.percentageVsAllTenants}%
        vs Similar: {translatedData.percentageVsSimilar}%
      </div>

      <h3>Control Scores</h3>
      {translatedData.controlScores?.map((control) => (
        <div key={control.controlName}>
          <h4>{control.title || control.controlName}</h4>
          <div>Score: {control.scoreInPercentage}%</div>
          <div>Threats: {control.threats?.join(', ')}</div>
          {control.remediation && (
            <div>Remediation: {control.remediation}</div>
          )}
          {control.actionUrl && (
            <a href={control.actionUrl}>Take Action</a>
          )}
        </div>
      ))}
    </div>
  );
};
```

**Dependencies:**
- Requires current tenant to be selected (returns empty state for "AllTenants")
- Integrates with CIPP standards data for remediation suggestions
- Automatically sorts controls by score percentage

**Data Processing:**
- Merges secure score data with control profiles
- Adds CIPP standard remediations when available
- Calculates percentage scores and comparisons
- Filters out default control state updates

## Utilities

### useMounted

Tracks component mount status to prevent state updates on unmounted components.

```javascript
import { useMounted } from '/src/hooks/use-mounted';

const MyComponent = () => {
  const isMounted = useMounted();
  const [data, setData] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      const result = await api.getData();
      
      // Only update state if component is still mounted
      if (isMounted()) {
        setData(result);
      }
    };

    fetchData();
  }, [isMounted]);

  return <div>{data ? data.content : 'Loading...'}</div>;
};
```

**Return Value:**
Function that returns `boolean` indicating if component is mounted.

**Use Cases:**
- Preventing memory leaks from async operations
- Avoiding state updates after component unmount
- Conditional API calls based on mount status

### useWindowScroll

Throttled window scroll event handler.

```javascript
import { useWindowScroll } from '/src/hooks/use-window-scroll';

const ScrollAwareComponent = () => {
  const [scrollPosition, setScrollPosition] = useState(0);

  useWindowScroll({
    handler: () => {
      setScrollPosition(window.scrollY);
    },
    delay: 100 // Throttle to every 100ms
  });

  return (
    <div>
      <div style={{ 
        position: 'fixed', 
        top: 0,
        opacity: scrollPosition > 100 ? 1 : 0 
      }}>
        Scroll Top Button
      </div>
      {/* Page content */}
    </div>
  );
};
```

**Parameters:**
```javascript
{
  handler: () => void,  // Scroll event handler
  delay: number        // Throttle delay in milliseconds
}
```

### usePageView

Placeholder hook for page view tracking.

```javascript
import { usePageView } from '/src/hooks/use-page-view';

const MyPage = () => {
  usePageView(); // No-op implementation, intended for analytics
  
  return <div>Page content</div>;
};
```

**Usage:**
- Placeholder for future analytics integration
- Can be extended to track page views
- No-op implementation ready for enhancement

### useHasAccess

Utility function for permission checking (located in `/src/utils/permissions.js`). Note: This is not a typical React hook but a utility function.

```javascript
import { useHasAccess } from '/src/utils/permissions';

const MyComponent = () => {
  const hasAccess = useHasAccess();
  
  // Returns a function that checks access based on user data
  const userPermissions = ['Identity.User.Read'];
  const userRoles = ['admin'];
  
  const canAccess = hasAccess(userPermissions, userRoles);
  
  return (
    <div>
      {canAccess && <AdminContent />}
    </div>
  );
};
```

**Parameters:**
- `requiredPermissions` - Array of required permissions
- `requiredRoles` - Array of required roles

**Return Value:**
Returns a function `(userPermissions, userRoles) => boolean` that checks if user has access.

**Usage:**
- Utility for checking user access without React hooks
- Can be used in non-React contexts
- Integrates with the main permission checking system

### useMockedUser

Development utility that returns mock user data.

```javascript
import { useMockedUser } from '/src/hooks/use-mocked-user';

const DevelopmentComponent = () => {
  const user = useMockedUser();
  
  return (
    <div>
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
};
```

**Return Value:**
```javascript
{
  id: string,
  avatar: string,
  name: string,
  email: string
}
```

### Redux Store Hooks

Redux store integration hooks exported from `/src/store/index.js`.

#### useSelector

Wrapper for Redux Toolkit's useSelector hook.

```javascript
import { useSelector } from '/src/store';

const MyComponent = () => {
  const settings = useSelector((state) => state.settings);
  const toasts = useSelector((state) => state.toasts);
  
  return (
    <div>
      Current theme: {settings.currentTheme?.label}
    </div>
  );
};
```

#### useDispatch

Wrapper for Redux Toolkit's useDispatch hook.

```javascript
import { useDispatch } from '/src/store';
import { showToast } from '/src/store/toasts';

const MyComponent = () => {
  const dispatch = useDispatch();
  
  const handleAction = () => {
    dispatch(showToast({
      message: 'Action completed',
      severity: 'success'
    }));
  };
  
  return (
    <button onClick={handleAction}>
      Perform Action
    </button>
  );
};
```

**Implementation:**
```javascript
export const useSelector = useReduxSelector;
export const useDispatch = () => useReduxDispatch();
```

**Usage:**
- Access Redux store state
- Dispatch actions to update store
- Integrates with CIPP's centralized state management

## Best Practices

### 1. Hook Composition
Combine multiple hooks for complex functionality:

```javascript
const useUserManagement = () => {
  const { currentTenant } = useSettings();
  const { checkPermissions } = usePermissions();
  const selection = useSelection([]);
  const dialog = useDialog();

  const canEditUsers = checkPermissions(['Identity.User.ReadWrite']);
  
  return {
    currentTenant,
    canEditUsers,
    selection,
    dialog
  };
};
```

### 2. Conditional Hook Usage
Some hooks depend on application state:

```javascript
const ConditionalComponent = () => {
  const { currentTenant } = useSettings();
  
  // Only use secure score hook when tenant is selected
  const secureScore = currentTenant !== "AllTenants" 
    ? useSecureScore()
    : { isFetching: false, isSuccess: false, translatedData: [] };
    
  return <div>...</div>;
};
```

### 3. Performance Optimization
Use memoization with complex hooks:

```javascript
const OptimizedComponent = ({ data }) => {
  const guidResolver = useGuidResolver();
  
  // Memoize expensive operations
  const processedData = useMemo(() => {
    return data.map(item => ({
      ...item,
      resolvedName: guidResolver.guidMapping[item.id] || item.id
    }));
  }, [data, guidResolver.guidMapping]);

  return <div>{/* Render processed data */}</div>;
};
```

### 4. Error Handling
Always handle loading and error states:

```javascript
const RobustComponent = () => {
  const { isLoading, isAuthenticated } = usePermissions();
  const secureScore = useSecureScore();

  if (isLoading) return <LoadingSpinner />;
  if (!isAuthenticated) return <LoginPrompt />;
  if (secureScore.isError) return <ErrorMessage />;

  return <ComponentContent />;
};
```

This comprehensive documentation covers all custom hooks in CIPP. Each hook is designed to encapsulate specific functionality and can be composed together to build complex application features while maintaining clean, reusable code.