# Naming Conventions

This document outlines the naming conventions used throughout the CIPP frontend application to ensure consistency and maintainability.

## Component Naming

### Component File Names
CIPP follows PascalCase for component file names with descriptive prefixes:

```
✅ Good Examples:
CippInfoCard.jsx
CippDataTable.js
CippFormComponent.jsx
CippUserActions.jsx
CippPermissionButton.jsx

❌ Avoid:
infoCard.jsx
dataTable.js
formComponent.jsx
userActions.jsx
```

### Component Name Patterns

#### 1. **Cipp Prefix**
Most components use the "Cipp" prefix to identify them as CIPP-specific components:
```jsx
export const CippInfoCard = (props) => { /* ... */ };
export const CippDataTable = (props) => { /* ... */ };
export const CippFormComponent = (props) => { /* ... */ };
```

#### 2. **Descriptive Component Names**
Component names should clearly describe their purpose:
```jsx
// ✅ Clear purpose
CippUserInfoCard
CippPropertyListCard
CippExchangeInfoCard
CippScheduledTaskActions
CippMailboxPermissionsDialog

// ❌ Vague purpose
CippCard
CippDialog
CippComponent
```

#### 3. **Component Type Suffixes**
Common suffixes indicate component type:
```jsx
// Cards
CippInfoCard
CippButtonCard
CippChartCard

// Dialogs
CippApiDialog
CippComponentDialog
CippLocationDialog

// Forms
CippFormComponent
CippFormSection
CippFormPage

// Actions
CippUserActions
CippExchangeActions
CippGdapActions
```

## File Naming Conventions

### Directory Structure Naming
```
src/
├── components/
│   ├── CippCards/              # PascalCase for component groups
│   ├── CippComponents/         # Core utilities
│   ├── CippFormPages/          # Form-specific components
│   ├── CippTable/              # Table components
│   └── CippWizard/             # Multi-step components
├── hooks/                      # Lowercase with hyphens
│   ├── use-auth.js
│   ├── use-permissions.js
│   └── use-settings.js
├── utils/                      # Lowercase with hyphens
│   ├── get-cipp-error.js
│   ├── apply-filters.js
│   └── permissions.js
└── theme/                      # Lowercase
    ├── colors.js
    ├── index.js
    └── utils.js
```

### File Extension Patterns
```
.jsx    # React components with JSX (optional - React supports JSX in .js files)
.js     # React components, utility functions, hooks, configs
.json   # Static data and configuration
.md     # Documentation files
```

**Note**: React supports JSX syntax in `.js` files without requiring `.jsx` extension. CIPP uses both patterns depending on the component.

## Variable and Function Naming

### React Hooks
Custom hooks follow the `use` prefix pattern:
```jsx
// ✅ Good hook names
const usePermissions = () => { /* ... */ };
const useHasPermission = (permissions) => { /* ... */ };
const useBreakpoints = () => { /* ... */ };
const useSettings = () => { /* ... */ };

// Hook destructuring
const { userPermissions, checkPermissions, isLoading } = usePermissions();
const { hasAccess, isAuthenticated } = useHasPermission(['read.users']);
```

### State Variables
State variables use descriptive camelCase:
```jsx
// ✅ Descriptive state names
const [isLoading, setIsLoading] = useState(false);
const [userRoles, setUserRoles] = useState([]);
const [offcanvasVisible, setOffcanvasVisible] = useState(false);
const [columnVisibility, setColumnVisibility] = useState({});
const [actionData, setActionData] = useState({ data: {}, action: {}, ready: false });

// ✅ Boolean states with is/has/can prefixes
const [isAuthenticated, setIsAuthenticated] = useState(false);
const [hasPermission, setHasPermission] = useState(false);
const [canEdit, setCanEdit] = useState(false);
```

### Function Naming
Functions use camelCase with descriptive verbs:
```jsx
// ✅ Action-oriented function names
const handleSubmit = () => { /* ... */ };
const handleUserAction = (action) => { /* ... */ };
const checkPermissions = (permissions) => { /* ... */ };
const convertBracketsToDots = (name) => { /* ... */ };
const getCippError = (error) => { /* ... */ };

// ✅ Event handler naming
const handleClick = () => { /* ... */ };
const handleChange = (value) => { /* ... */ };
const handleClose = () => { /* ... */ };
const handleSubmit = (data) => { /* ... */ };

// ✅ Utility function naming
const applyFilters = (data, filters) => { /* ... */ };
const formatLicenseTranslation = (license) => { /* ... */ };
const getInitials = (name) => { /* ... */ };
```

## Prop Naming Conventions

### Component Props
Props follow camelCase and are descriptive:
```jsx
const CippInfoCard = ({
  // Data props
  value,
  label,
  icon,
  
  // Behavior props
  isFetching = false,
  actionLink,
  actionText,
  cardSize = 'auto',
  
  // Event handlers
  onClick,
  onAction,
  onError,
  
  // Style props
  className,
  sx,
  
  // Rest props
  ...other
}) => {
  // Component implementation
};
```

### Boolean Props
Boolean props often use `is`, `has`, `can`, or `should` prefixes:
```jsx
const Component = ({
  isLoading = false,
  isAuthenticated = false,
  hasPermission = false,
  canEdit = false,
  shouldShow = true,
  hideTitle = false,
  exportEnabled = true,
  refreshFunction,
}) => { /* ... */ };
```

### Event Handler Props
Event handlers follow the `on` prefix pattern:
```jsx
const Component = ({
  onClick,
  onSubmit,
  onChange,
  onError,
  onSuccess,
  onClose,
  onOpen,
  onRefresh,
}) => { /* ... */ };
```

## API and Data Naming

### API Endpoint Naming
API calls use descriptive naming:
```jsx
// ✅ Clear API endpoint identification
const userList = ApiGetCall({
  url: '/api/listUsers',
  queryKey: 'users',
});

const tenantData = ApiGetCall({
  url: '/api/tenants',
  data: { TenantFilter: tenantId },
  queryKey: ['tenant', tenantId],
});

const permissionCheck = ApiGetCall({
  url: '/api/me',
  queryKey: 'authmecipp',
});
```

### Query Key Naming
Query keys use descriptive strings or arrays:
```jsx
// ✅ Descriptive query keys
queryKey: 'users'
queryKey: 'authmecipp'
queryKey: ['tenant', tenantId]
queryKey: ['permissions', userId]
queryKey: ['data', tenantFilter, endpoint]
```

### Data Property Naming
Data properties follow API conventions but are transformed when needed:
```jsx
// API response transformation
const transformedData = useMemo(() => {
  return apiData?.map(item => ({
    id: item.ID,
    displayName: item.DisplayName,
    userPrincipalName: item.UserPrincipalName,
    licenses: item.AssignedLicenses || [],
    lastSignIn: item.LastSignInDateTime,
  }));
}, [apiData]);
```

## CSS and Styling Naming

### CSS Class Naming
When custom CSS classes are needed, use kebab-case:
```css
/* ✅ Good CSS class names */
.cipp-card-container { }
.cipp-table-header { }
.cipp-form-section { }
.cipp-loading-state { }

/* ❌ Avoid camelCase in CSS */
.cippCardContainer { }
.cippTableHeader { }
```

### Material-UI sx Props
Material-UI styling uses camelCase properties:
```jsx
// ✅ Material-UI sx prop naming
<Box sx={{
  backgroundColor: 'primary.main',
  borderRadius: 1,
  padding: 2,
  marginBottom: 3,
  '&:hover': {
    backgroundColor: 'primary.dark',
  },
}} />
```

## Constant Naming

### Configuration Constants
Constants use SCREAMING_SNAKE_CASE:
```jsx
// ✅ Configuration constants
const SIDE_NAV_WIDTH = 270;
const SIDE_NAV_COLLAPSED_WIDTH = 73;
const TOP_NAV_HEIGHT = 64;
const DEFAULT_PAGE_SIZE = 25;
const MAX_RETRIES = 3;

// ✅ API endpoints
const API_ENDPOINTS = {
  USERS: '/api/listUsers',
  TENANTS: '/api/tenants',
  PERMISSIONS: '/api/me',
};
```

### Enum-like Constants
Use PascalCase for enum-like objects:
```jsx
// ✅ Enum-like constants
const ComponentTypes = {
  TextField: 'textField',
  AutoComplete: 'autoComplete',
  Switch: 'switch',
  Radio: 'radio',
  Checkbox: 'checkbox',
};

const TableModes = {
  Simple: 'simple',
  Advanced: 'advanced',
  Custom: 'custom',
};
```

## Form Field Naming

### Form Field Names
Form fields use camelCase matching backend expectations:
```jsx
// ✅ Form field naming
<CippFormComponent
  name="displayName"
  label="Display Name"
  formControl={formControl}
/>

<CippFormComponent
  name="userPrincipalName"
  label="User Principal Name"
  formControl={formControl}
/>

// ✅ Nested form fields with bracket notation
<CippFormComponent
  name="licenses[0].skuId"
  label="License SKU"
  formControl={formControl}
/>
```

### Form Validation Names
Validation rules use descriptive names:
```jsx
const validators = {
  required: 'This field is required',
  email: 'Please enter a valid email address',
  minLength: { value: 3, message: 'Minimum 3 characters required' },
  pattern: {
    value: /^[A-Za-z0-9]+$/,
    message: 'Only alphanumeric characters allowed'
  },
};
```

## Error and Status Naming

### Error Handling
Error-related variables and functions use clear naming:
```jsx
// ✅ Error handling naming
const [error, setError] = useState(null);
const [hasError, setHasError] = useState(false);
const handleError = (error) => { /* ... */ };
const getCippError = (error) => { /* ... */ };
const clearError = () => setError(null);

// ✅ Status naming
const [status, setStatus] = useState('idle'); // 'idle', 'loading', 'success', 'error'
const [operationStatus, setOperationStatus] = useState('pending');
```

## Permission Naming

### Permission Strings
Permission strings follow a hierarchical pattern:
```jsx
// ✅ Permission naming patterns
const requiredPermissions = [
  'Identity.User.Read',
  'Identity.User.ReadWrite',
  'Exchange.Mailbox.Read',
  'CIPP.Admin.Settings',
  'Tenant.Administration.ReadWrite',
];

// ✅ Wildcard permissions
const adminPermissions = [
  'CIPP.Admin.*',
  'Identity.*',
  'Exchange.*',
];
```

### Role Names
Role names use lowercase with descriptive terms:
```jsx
// ✅ Role naming
const userRoles = [
  'admin',
  'superadmin',
  'editor',
  'readonly',
];
```

## Import and Export Naming

### Import Statements
Import names should be clear and consistent:
```jsx
// ✅ Clear import naming
import { CippInfoCard } from '../CippCards/CippInfoCard';
import { ApiGetCall } from '/src/api/ApiCall';
import { usePermissions } from '/src/hooks/use-permissions';
import { getCippError } from '/src/utils/get-cipp-error';

// ✅ Grouped imports
import React, { useState, useMemo, useCallback } from 'react';
import { Box, Card, Typography, Button } from '@mui/material';
import { useTheme } from '@mui/material/styles';
```

### Export Patterns
Exports use consistent patterns:
```jsx
// ✅ Named exports for components
export const CippInfoCard = (props) => { /* ... */ };
export const CippDataTable = (props) => { /* ... */ };

// ✅ Default exports for single-purpose files
export default function UtilityFunction() { /* ... */ }

// ✅ Mixed exports
export const helper1 = () => { /* ... */ };
export const helper2 = () => { /* ... */ };
export default MainComponent;
```

## Best Practices Summary

### ✅ Do's
1. **Use descriptive names** that clearly indicate purpose
2. **Follow established patterns** (Cipp prefix, camelCase, etc.)
3. **Be consistent** across similar components and functions
4. **Use boolean prefixes** (is, has, can, should) for boolean values
5. **Group related items** with consistent naming patterns
6. **Use verb-noun patterns** for functions (handleClick, getUser)

### ❌ Don'ts
1. **Avoid abbreviations** unless they're widely understood
2. **Don't use generic names** (Component, Handler, etc.)
3. **Avoid inconsistent casing** within the same context
4. **Don't use misleading names** that don't match functionality
5. **Avoid overly long names** that hurt readability

### Examples of Good vs. Bad Naming

```jsx
// ✅ Good naming
const CippUserPermissionDialog = ({ user, permissions, onUpdate }) => {
  const [isSubmitting, setIsSubmitting] = useState(false);
  const handlePermissionUpdate = useCallback((newPermissions) => {
    setIsSubmitting(true);
    onUpdate(user.id, newPermissions);
  }, [user.id, onUpdate]);
};

// ❌ Poor naming
const Dialog = ({ u, p, onUpd }) => {
  const [submit, setSubmit] = useState(false);
  const handle = useCallback((newP) => {
    setSubmit(true);
    onUpd(u.id, newP);
  }, [u.id, onUpd]);
};
```

Following these naming conventions ensures code readability, maintainability, and consistency across the CIPP application.