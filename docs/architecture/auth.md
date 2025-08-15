# Authentication & Authorization Architecture

## Overview

CIPP implements a comprehensive authentication and authorization system that combines Azure Static Web Apps authentication with a custom permission framework. The architecture provides secure access to Microsoft 365 partner management features while supporting complex multi-tenant scenarios and granular permission controls.

## Azure Static Web Apps Authentication

CIPP leverages Azure Static Web Apps' built-in authentication system for seamless Azure AD integration and session management.

### Authentication Flow

The authentication flow follows Azure SWA's OAuth 2.0 implementation:

1. **Unauthenticated Request** → User accesses CIPP
2. **Redirect to Azure AD** → Automatic redirect via SWA configuration
3. **Azure AD Authentication** → User signs in with organizational credentials
4. **Token Exchange** → SWA exchanges authorization code for tokens
5. **Session Creation** → SWA creates secure session cookie
6. **Application Access** → User gains access to CIPP features

### SWA Configuration

The authentication configuration is defined in `staticwebapp.config.json`:

```json
{
  "routes": [
    {
      "route": "/login",
      "rewrite": "/.auth/login/aad"
    },
    {
      "route": "/logout",
      "redirect": "/.auth/logout?post_logout_redirect_uri=/LogoutRedirect"
    },
    {
      "route": "/.auth/login/twitter",
      "statusCode": 404
    },
    {
      "route": "/.auth/login/github",
      "statusCode": 404
    },
    {
      "route": "/authredirect",
      "allowedRoles": ["admin", "editor", "readonly", "authenticated", "anonymous"]
    },
    {
      "route": "/api/Public*",
      "allowedRoles": ["admin", "editor", "readonly", "authenticated", "anonymous"]
    },
    {
      "route": "*",
      "allowedRoles": ["admin", "editor", "readonly", "authenticated"]
    }
  ],
  "responseOverrides": {
    "401": {
      "redirect": "/.auth/login/aad?post_login_redirect_uri=.referrer",
      "statusCode": 302,
      "exclude": ["/assets/illustrations/*.{png,jpg,gif}", "/css/*"]
    },
    "403": {
      "rewrite": "/403"
    },
    "404": {
      "rewrite": "/404"
    }
  }
}
```

**Key Configuration Features:**
- **Azure AD Only**: Disables other authentication providers (Twitter, GitHub)
- **Role-Based Routes**: Different routes require different role levels
- **Automatic Redirects**: Handles authentication redirects transparently
- **Public API Access**: Allows anonymous access to specific public endpoints

### Session Management

Azure SWA provides automatic session management:

```javascript
// Session data structure from /.auth/me endpoint
{
  "clientPrincipal": {
    "identityProvider": "aad",
    "userId": "user-guid",
    "userDetails": "user@domain.com",
    "userRoles": ["authenticated", "admin"]
  }
}
```

**Session Features:**
- **Automatic Token Refresh**: SWA handles token renewal transparently
- **Secure Cookie Storage**: Session stored in httpOnly, secure cookies
- **Cross-Domain Support**: Works across CIPP frontend and API domains
- **Logout Handling**: Clean session termination with redirect

## CIPP Permission System

Beyond Azure SWA's role-based authentication, CIPP implements a sophisticated permission system for granular access control.

### Permission Architecture

The permission system operates on two levels:

1. **Azure SWA Roles**: Basic role-based access (admin, editor, readonly, authenticated)
2. **CIPP Permissions**: Granular, feature-specific permissions with wildcard support

### Permission Structure

CIPP permissions follow a hierarchical naming convention:

```javascript
// Permission hierarchy examples
"CIPP.Core.*"                    // All CIPP core functions
"Identity.*"                     // All identity management
"Identity.User.*"                // All user operations
"Identity.User.Read"             // Specific user read permission
"Exchange.Mailbox.Edit"          // Specific mailbox edit permission
"Tenant.Administration.*"        // All tenant administration
```

### Permission Implementation

The permission system is implemented through utility functions in `/src/utils/permissions.js`:

#### Core Permission Checker

```javascript
export const hasPermission = (userPermissions, requiredPermissions) => {
  if (!userPermissions || !requiredPermissions) {
    return false;
  }

  if (!Array.isArray(userPermissions) || !Array.isArray(requiredPermissions)) {
    return false;
  }

  if (requiredPermissions.length === 0) {
    return true; // No permissions required
  }

  return userPermissions.some((userPerm) => {
    return requiredPermissions.some((requiredPerm) => {
      // Exact match
      if (userPerm === requiredPerm) {
        return true;
      }

      // Pattern matching - check if required permission contains wildcards
      if (requiredPerm.includes("*")) {
        // Convert wildcard pattern to regex
        const regexPattern = requiredPerm
          .replace(/\./g, "\\.") // Escape dots
          .replace(/\*/g, ".*"); // Convert * to .*
        const regex = new RegExp(`^${regexPattern}$`);
        return regex.test(userPerm);
      }

      return false;
    });
  });
};
```

#### Role Checker

```javascript
export const hasRole = (userRoles, requiredRoles) => {
  if (!userRoles || !requiredRoles) {
    return false;
  }

  if (!Array.isArray(userRoles) || !Array.isArray(requiredRoles)) {
    return false;
  }

  if (requiredRoles.length === 0) {
    return true; // No roles required
  }

  return requiredRoles.some((requiredRole) => userRoles.includes(requiredRole));
};
```

#### Combined Access Checker

```javascript
export const hasAccess = ({
  userPermissions,
  userRoles,
  requiredPermissions = [],
  requiredRoles = [],
}) => {
  // Check roles first (if any are required)
  if (requiredRoles.length > 0) {
    const hasRequiredRole = hasRole(userRoles, requiredRoles);
    if (!hasRequiredRole) {
      return false;
    }
  }

  // Check permissions (if any are required)
  if (requiredPermissions.length > 0) {
    const hasRequiredPermission = hasPermission(userPermissions, requiredPermissions);
    if (!hasRequiredPermission) {
      return false;
    }
  }

  return true;
};
```

### Wildcard Pattern Matching

CIPP's permission system supports sophisticated wildcard patterns:

**Pattern Examples:**
- `CIPP.*` - Matches any CIPP permission
- `Identity.User.*` - Matches any user-related permission
- `Exchange.*.Read` - Matches any read permission in Exchange
- `*.Administration.*` - Matches any administration permission

**Regex Conversion:**
```javascript
// Pattern: "Identity.User.*"
// Regex: "^Identity\.User\..*$"

// Pattern: "*.Admin.*"
// Regex: "^.*\.Admin\..*$"
```

## Authentication State Management

CIPP integrates authentication state with its hybrid state management system through React Query.

### Authentication Queries

#### Azure SWA Session Query

```javascript
const swaStatus = ApiGetCall({
  url: "/.auth/me",
  queryKey: "authmeswa",
  staleTime: 120000, // 2 minutes
  refetchOnWindowFocus: true,
});
```

**Features:**
- **Window Focus Refetch**: Validates session when user returns to tab
- **Short Stale Time**: Ensures fresh authentication status
- **Automatic Retry**: Handles temporary network issues

#### CIPP Permissions Query

```javascript
const currentRole = ApiGetCall({
  url: "/api/me",
  queryKey: "authmecipp",
  waiting: !swaStatus.isSuccess || swaStatus.data?.clientPrincipal === null,
});
```

**Features:**
- **Conditional Execution**: Only runs after SWA authentication succeeds
- **Permission Caching**: Caches user permissions for performance
- **Role Synchronization**: Ensures SWA and CIPP roles are synchronized

### Authentication State Flow

1. **Initial Load** → Check Azure SWA session status
2. **SWA Validation** → Verify active Azure AD session
3. **CIPP Permission Loading** → Fetch CIPP-specific permissions
4. **State Synchronization** → Merge SWA roles with CIPP permissions
5. **Navigation Filtering** → Apply permissions to navigation menu
6. **Route Protection** → Enforce access controls on routes

## Route Protection

CIPP implements multi-layered route protection through the PrivateRoute component and permission checks.

### PrivateRoute Component

The core route protection is implemented in `/src/components/PrivateRoute.js`:

```javascript
export const PrivateRoute = ({ children, routeType }) => {
  const session = ApiGetCall({
    url: "/.auth/me",
    queryKey: "authmeswa",
    refetchOnWindowFocus: true,
    staleTime: 120000, // 2 minutes
  });

  const apiRoles = ApiGetCall({
    url: "/api/me",
    queryKey: "authmecipp",
    retry: 2, // Reduced retry count to show offline message sooner
    waiting: !session.isSuccess || session.data?.clientPrincipal === null,
  });

  // Loading state
  if (
    session.isLoading ||
    apiRoles.isLoading ||
    (apiRoles.isFetching && (apiRoles.data === null || apiRoles.data === undefined))
  ) {
    return <LoadingPage />;
  }

  // API offline detection
  if (
    apiRoles?.error?.response?.status === 404 || // API endpoint not found
    apiRoles?.error?.response?.status === 502 || // Service unavailable
    (apiRoles?.isSuccess && !apiRoles?.data) // No client principal data
  ) {
    return <ApiOfflinePage />;
  }

  // Unauthenticated user
  if (null === session?.data?.clientPrincipal || session?.data === undefined) {
    return <UnauthenticatedPage />;
  }

  // Role validation
  let roles = null;
  
  if (
    session?.isSuccess &&
    apiRoles?.isSuccess &&
    undefined !== apiRoles?.data?.clientPrincipal &&
    session?.data?.clientPrincipal?.userDetails &&
    apiRoles?.data?.clientPrincipal?.userDetails &&
    session?.data?.clientPrincipal?.userDetails !== apiRoles?.data?.clientPrincipal?.userDetails
  ) {
    // Refetch if user details don't match
    apiRoles.refetch();
  }

  if (null !== apiRoles?.data?.clientPrincipal && undefined !== apiRoles?.data) {
    roles = apiRoles?.data?.clientPrincipal?.userRoles ?? [];
  } else if (null === apiRoles?.data?.clientPrincipal || undefined === apiRoles?.data) {
    return <UnauthenticatedPage />;
  }

  if (null === roles) {
    return <UnauthenticatedPage />;
  } else {
    const blockedRoles = ["anonymous", "authenticated"];
    const userRoles = roles?.filter((role) => !blockedRoles.includes(role)) ?? [];
    const isAuthenticated = userRoles.length > 0 && !apiRoles?.error;
    const isAdmin = roles?.includes("admin") || roles?.includes("superadmin");
    
    if (routeType === "admin") {
      return !isAdmin ? <UnauthenticatedPage /> : children;
    } else {
      return !isAuthenticated ? <UnauthenticatedPage /> : children;
    }
  }
};
```

**Protection Features:**
- **Multi-Stage Validation**: Validates both SWA and CIPP authentication
- **Error State Handling**: Different pages for different error types
- **Role-Based Access**: Supports admin-only routes
- **Session Synchronization**: Ensures SWA and CIPP sessions match
- **Graceful Degradation**: Shows appropriate messages for offline states

### Permission-Based UI Components

CIPP provides reusable components for permission-based UI rendering:

#### PermissionButton Component

```javascript
export const PermissionButton = ({
  requiredPermissions = [],
  requiredRoles = [],
  hideIfNoAccess = false,
  children,
  ...buttonProps
}) => {
  const { userPermissions, userRoles, isAuthenticated } = usePermissions();

  const hasRequiredAccess =
    isAuthenticated &&
    hasAccess({
      userPermissions,
      userRoles,
      requiredPermissions,
      requiredRoles,
    });

  if (!hasRequiredAccess && hideIfNoAccess) {
    return null;
  }

  return (
    <Button disabled={!hasRequiredAccess} {...buttonProps}>
      {children}
    </Button>
  );
};
```

#### PermissionCheck Component

```javascript
export const PermissionCheck = ({
  requiredPermissions = [],
  requiredRoles = [],
  children,
  fallback = null,
}) => {
  const { userPermissions, userRoles, isAuthenticated } = usePermissions();

  const hasRequiredAccess =
    isAuthenticated &&
    hasAccess({
      userPermissions,
      userRoles,
      requiredPermissions,
      requiredRoles,
    });

  if (hasRequiredAccess) {
    return children;
  }

  return fallback;
};
```

## Permission Hooks

CIPP provides React hooks for permission checking within components.

### usePermissions Hook

The primary permission hook (`/src/hooks/use-permissions.js`):

```javascript
export const usePermissions = () => {
  const currentRole = ApiGetCall({
    url: "/api/me",
    queryKey: "authmecipp",
  });

  const userRoles = currentRole.data?.clientPrincipal?.userRoles || [];
  const userPermissions = currentRole.data?.permissions || [];
  const isLoading = currentRole.isLoading;
  const isAuthenticated = currentRole.isSuccess && userRoles.length > 0;

  const checkPermissions = useCallback(
    (requiredPermissions) => {
      if (!isAuthenticated) return false;
      return hasPermission(userPermissions, requiredPermissions);
    },
    [userPermissions, isAuthenticated]
  );

  const checkRoles = useCallback(
    (requiredRoles) => {
      if (!isAuthenticated) return false;
      return hasRole(userRoles, requiredRoles);
    },
    [userRoles, isAuthenticated]
  );

  const checkAccess = useCallback(
    (config = {}) => {
      if (!isAuthenticated) return false;

      const { requiredPermissions = [], requiredRoles = [] } = config;

      return hasAccess({
        userPermissions,
        userRoles,
        requiredPermissions,
        requiredRoles,
      });
    },
    [userPermissions, userRoles, isAuthenticated]
  );

  return {
    userPermissions,
    userRoles,
    isLoading,
    isAuthenticated,
    checkPermissions,
    checkRoles,
    checkAccess,
  };
};
```

### useHasPermission Hook

Simplified permission checking hook:

```javascript
export const useHasPermission = (requiredPermissions = [], requiredRoles = []) => {
  const { checkAccess, isLoading, isAuthenticated } = usePermissions();

  const hasAccess = checkAccess({ requiredPermissions, requiredRoles });

  return {
    hasAccess,
    isLoading,
    isAuthenticated,
  };
};
```

### Usage Examples

```javascript
// In component code
const UserManagement = () => {
  const { checkPermissions, checkRoles, isLoading } = usePermissions();
  
  const canCreateUsers = checkPermissions(["Identity.User.Create"]);
  const canEditUsers = checkPermissions(["Identity.User.Edit"]);
  const isAdmin = checkRoles(["admin", "superadmin"]);
  
  if (isLoading) {
    return <LoadingSkeleton />;
  }
  
  return (
    <Box>
      {canCreateUsers && (
        <Button onClick={handleCreateUser}>Create User</Button>
      )}
      {canEditUsers && (
        <Button onClick={handleEditUser}>Edit User</Button>
      )}
      {isAdmin && (
        <AdminPanel />
      )}
    </Box>
  );
};
```

## Multi-Tenant Context

CIPP supports complex multi-tenant scenarios where authentication context varies by selected tenant.

### Tenant Selection Integration

The tenant context is managed through the Settings Context and integrated with authentication:

```javascript
// Tenant selection affects available permissions
const currentTenant = settings?.currentTenant;

// Some pages don't support "All Tenants" view
const allTenantsSupport = props.allTenantsSupport ?? true;

if ((currentTenant === "AllTenants" || !currentTenant) && !allTenantsSupport) {
  return <TenantNotSupportedMessage />;
}
```

### Tenant-Specific Permissions

Different tenants may have different permission sets:

```javascript
// API calls include tenant context
const usersAPI = ApiGetCall({
  url: "/api/ListUsers",
  queryKey: "usersList",
  data: { tenantFilter: currentTenant },
  waiting: !currentTenant || currentTenant === "AllTenants",
});
```

### Tenant Context Navigation

Navigation items can be filtered based on tenant support:

```javascript
// Navigation config with tenant support flags
{
  title: "User Management",
  path: "/identity/administration/users",
  permissions: ["Identity.User.*"],
  tenantSupport: "single", // "single", "all", or "both"
}
```

## Security Patterns

### Token Handling

CIPP follows secure token handling practices:

- **No Frontend Token Storage**: Tokens managed by Azure SWA
- **HttpOnly Cookies**: Session cookies not accessible via JavaScript
- **Automatic Token Refresh**: SWA handles token renewal transparently
- **Secure Transmission**: All authentication over HTTPS

### Cross-Site Request Forgery (CSRF) Protection

Azure SWA provides built-in CSRF protection:

- **SameSite Cookies**: Prevents cross-site request attacks
- **Origin Validation**: SWA validates request origins
- **Secure Headers**: Proper security headers set by SWA

### Permission Validation

All permission checks include validation:

```javascript
// Always validate inputs
if (!userPermissions || !requiredPermissions) {
  return false;
}

if (!Array.isArray(userPermissions) || !Array.isArray(requiredPermissions)) {
  return false;
}
```

### Error Handling

Authentication errors are handled gracefully:

```javascript
// Different error states
- UnauthenticatedPage: User needs to log in
- ApiOfflinePage: CIPP API is unavailable  
- Error500: General application errors
- LoadingPage: Authentication in progress
```

## Best Practices

### Permission Definition

```javascript
// Use hierarchical permission naming
"Feature.Entity.Action"       // Good: "Identity.User.Create"
"CreateUser"                  // Avoid: Flat naming

// Use wildcards for group permissions
"Identity.*"                  // All identity permissions
"*.Read"                      // All read permissions across features
```

### Component Protection

```javascript
// Prefer declarative permission components
<PermissionCheck requiredPermissions={["Identity.User.Edit"]}>
  <EditUserForm />
</PermissionCheck>

// Over imperative checks
const canEdit = checkPermissions(["Identity.User.Edit"]);
if (canEdit) {
  return <EditUserForm />;
}
```

### Hook Usage

```javascript
// Use specific permission hooks for better performance
const { hasAccess } = useHasPermission(["Identity.User.Create"]);

// Instead of general permission hook when you only need one check
const { checkPermissions } = usePermissions();
const hasAccess = checkPermissions(["Identity.User.Create"]);
```

### Error Boundaries

```javascript
// Wrap authentication-sensitive areas
<ErrorBoundary FallbackComponent={AuthError}>
  <PermissionBasedComponent />
</ErrorBoundary>
```

### Testing Patterns

```javascript
// Mock authentication state for testing
const mockAuthState = {
  userRoles: ["admin"],
  userPermissions: ["CIPP.Core.*", "Identity.*"],
  isAuthenticated: true,
};

// Test with different permission sets
describe("UserManagement with admin permissions", () => {
  // Test admin functionality
});

describe("UserManagement with read-only permissions", () => {
  // Test limited functionality
});
```

This authentication and authorization architecture provides CIPP with enterprise-grade security while maintaining excellent user experience and supporting the complex requirements of Microsoft 365 partner management scenarios.

## Permission Components Implementation

CIPP provides a comprehensive set of UI components that adapt based on user permissions, enabling seamless role-based user interface behavior.

### Core Permission Components

#### PermissionCheck Component

The `PermissionCheck` component provides conditional rendering based on user permissions:

```jsx
/**
 * Permission-aware conditional rendering component
 * @param {Object} props - Component props
 * @param {string[]} props.requiredPermissions - Array of required permissions
 * @param {string[]} props.requiredRoles - Array of required roles
 * @param {React.ReactNode} props.children - Content to render if user has access
 * @param {React.ReactNode} props.fallback - Content to render if user doesn't have access
 */
export const PermissionCheck = ({
  requiredPermissions = [],
  requiredRoles = [],
  children,
  fallback = null,
}) => {
  const { userPermissions, userRoles, isAuthenticated } = usePermissions();

  const hasRequiredAccess =
    isAuthenticated &&
    hasAccess({
      userPermissions,
      userRoles,
      requiredPermissions,
      requiredRoles,
    });

  if (hasRequiredAccess) {
    return children;
  }

  return fallback;
};
```

**Usage Examples:**
```jsx
// Basic permission check
<PermissionCheck requiredPermissions={['Identity.User.ReadWrite']}>
  <Button>Edit User</Button>
</PermissionCheck>

// With fallback content
<PermissionCheck 
  requiredPermissions={['CIPP.AppSettings.ReadWrite']}
  fallback={<Typography color="text.secondary">Access denied</Typography>}
>
  <SettingsPanel />
</PermissionCheck>

// Multiple permissions (OR logic)
<PermissionCheck requiredPermissions={['CIPP.Admin.*', 'Identity.User.ReadWrite']}>
  <UserEditForm />
</PermissionCheck>

// Role-based access
<PermissionCheck requiredRoles={['admin', 'superadmin']}>
  <AdminDashboard />
</PermissionCheck>
```

#### PermissionButton Component

The `PermissionButton` component automatically disables or hides buttons based on permissions:

```jsx
/**
 * Permission-aware Button component
 * @param {Object} props - Button props
 * @param {string[]} props.requiredPermissions - Array of required permissions
 * @param {string[]} props.requiredRoles - Array of required roles
 * @param {boolean} props.hideIfNoAccess - Hide button if user doesn't have access
 * @param {React.ReactNode} props.children - Button content
 */
export const PermissionButton = ({
  requiredPermissions = [],
  requiredRoles = [],
  hideIfNoAccess = false,
  children,
  ...buttonProps
}) => {
  const { userPermissions, userRoles, isAuthenticated } = usePermissions();

  const hasRequiredAccess =
    isAuthenticated &&
    hasAccess({
      userPermissions,
      userRoles,
      requiredPermissions,
      requiredRoles,
    });

  if (!hasRequiredAccess && hideIfNoAccess) {
    return null;
  }

  return (
    <Button 
      disabled={!hasRequiredAccess} 
      {...buttonProps}
      title={!hasRequiredAccess ? 'Insufficient permissions' : buttonProps.title}
    >
      {children}
    </Button>
  );
};
```

**Usage Examples:**
```jsx
// Basic permission button
<PermissionButton 
  requiredPermissions={['Identity.User.ReadWrite']}
  variant="outlined"
  color="error"
>
  Delete User
</PermissionButton>

// Hidden if no access
<PermissionButton
  requiredPermissions={['CIPP.AppSettings.ReadWrite']}
  hideIfNoAccess
  variant="contained"
>
  System Settings
</PermissionButton>
```

### Actual CIPP Permission Components

#### CippPermissionCheck Component

CIPP includes a specialized system permission checker for validating CIPP application permissions:

**Location**: `/src/components/CippSettings/CippPermissionCheck.jsx`

This component validates system-level permissions like API access, GDAP configuration, and tenant access.

#### Navigation Permission Filtering

Navigation items are filtered based on user permissions in the layout components:

**Location**: `/src/layouts/index.js`

```jsx
// Navigation items are filtered based on user permissions
const filteredNavItems = navItems.filter(item => {
  if (!item.requiredPermissions) return true;
  return hasPermission(userPermissions, item.requiredPermissions);
});
```

#### CippRoles Management Components

CIPP includes dedicated components for role management:

**Locations**:
- `/src/components/CippSettings/CippRoles.jsx` - Role management interface
- `/src/components/CippSettings/CippRoleAddEdit.jsx` - Role editing
- `/src/components/CippSettings/CippCustomRoles.jsx` - Custom role management

## Comprehensive RBAC System Implementation

CIPP implements a sophisticated Role-Based Access Control (RBAC) system that provides fine-grained permission management for Microsoft 365 partner portal operations.

### Permission System Architecture

The permission system consists of several key components:

1. **Hierarchical Permissions** - Structured permission inheritance
2. **Wildcard Patterns** - Flexible permission matching
3. **Role-Based Assignment** - Role-to-permission mapping
4. **Permission Validation** - Client and server-side checks
5. **Dynamic Permission Checking** - Real-time access validation

### Permission Hierarchy

```
CIPP.*                          # Core CIPP system permissions
├── CIPP.SuperAdmin.*          # Super administrator functions
├── CIPP.Admin.*               # Administrative operations  
├── CIPP.Core.*                # Core system functions
├── CIPP.AppSettings.*         # Application settings management
└── CIPP.Scheduler.*           # Task scheduling operations

Identity.*                      # Identity management permissions
├── Identity.User.*            # User account management
├── Identity.Group.*           # Security group management
├── Identity.Device.*          # Device registration and management
└── Identity.Application.*     # Application registrations

Tenant.*                       # Tenant-level permissions
├── Tenant.Administration.*    # Tenant configuration and settings
├── Tenant.Application.*       # Application management within tenants
└── Tenant.Standards.ReadWrite # Tenant compliance and standards

Exchange.*                     # Exchange Online permissions
├── Exchange.Mailbox.*         # Mailbox creation and management
├── Exchange.SpamFilter.*      # Anti-spam and mail filtering
├── Exchange.TransportRule.*   # Mail flow and transport rules
└── Exchange.Connector.*       # Mail connector management

Endpoint.*                     # Microsoft Endpoint Manager permissions
├── Endpoint.MEM.*             # Intune device management
├── Endpoint.Application.*     # Application deployment and management
├── Endpoint.Policy.*          # Device compliance and configuration policies
└── Endpoint.Autopilot.*       # Windows Autopilot management

Security.*                     # Security and compliance permissions
├── Security.Incident.*        # Security incident management
├── Security.Alert.*           # Security alert management
├── Security.Policy.*          # Security policy management
└── Security.Audit.*           # Audit log access and management
```

### Permission Implementation Details

#### Core Permission Validation (`/src/utils/permissions.js`)

```jsx
/**
 * Check if user has permission using pattern matching
 * @param {string[]} userPermissions - Array of user permissions
 * @param {string[]} requiredPermissions - Array of required permissions (can include wildcards)
 * @returns {boolean} - True if user has at least one of the required permissions
 */
export const hasPermission = (userPermissions, requiredPermissions) => {
  if (!userPermissions || !requiredPermissions) {
    return false;
  }

  if (!Array.isArray(userPermissions) || !Array.isArray(requiredPermissions)) {
    return false;
  }

  if (requiredPermissions.length === 0) {
    return true; // No permissions required
  }

  return userPermissions.some((userPerm) => {
    return requiredPermissions.some((requiredPerm) => {
      // Exact match
      if (userPerm === requiredPerm) {
        return true;
      }

      // Pattern matching - check if required permission contains wildcards
      if (requiredPerm.includes("*")) {
        // Convert wildcard pattern to regex
        const regexPattern = requiredPerm
          .replace(/\./g, "\\.") // Escape dots
          .replace(/\*/g, ".*"); // Convert * to .*
        const regex = new RegExp(`^${regexPattern}$`);
        return regex.test(userPerm);
      }

      return false;
    });
  });
};

/**
 * Check if user has access based on both permissions and roles
 */
export const hasAccess = ({
  userPermissions,
  userRoles,
  requiredPermissions = [],
  requiredRoles = [],
}) => {
  // Check roles first (if any are required)
  if (requiredRoles.length > 0) {
    const hasRequiredRole = hasRole(userRoles, requiredRoles);
    if (!hasRequiredRole) {
      return false;
    }
  }

  // Check permissions (if any are required)
  if (requiredPermissions.length > 0) {
    const hasRequiredPermission = hasPermission(userPermissions, requiredPermissions);
    if (!hasRequiredPermission) {
      return false;
    }
  }

  return true;
};
```

### Role Configuration System

#### Role Templates (`/src/data/cipp-roles.json`)

CIPP uses a template-based role system with include/exclude patterns:

```json
{
  "readonly": {
    "include": ["*.Read"],
    "exclude": ["CIPP.SuperAdmin.*"]
  },
  "editor": {
    "include": ["*.Read", "*.ReadWrite"],
    "exclude": [
      "CIPP.SuperAdmin.*",
      "CIPP.Admin.*",
      "CIPP.AppSettings.*",
      "Tenant.Standards.ReadWrite"
    ]
  },
  "admin": {
    "include": ["*"],
    "exclude": ["CIPP.SuperAdmin.*"]
  },
  "superadmin": {
    "include": ["*"],
    "exclude": []
  }
}
```

**Role Descriptions:**
- **superadmin**: Full access to all CIPP functions including super admin operations
- **admin**: Administrative access to all functions except super admin operations  
- **editor**: Read and write access with restrictions on admin functions and app settings
- **readonly**: View-only access to most data, excluding super admin functions

### Wildcard Permission Matching

CIPP supports flexible wildcard patterns for permissions:

```javascript
// Wildcard examples and matches using CIPP permission patterns
const wildcardExamples = [
  {
    userPermission: 'Identity.User.Read',
    requiredPermission: 'Identity.*',
    matches: true,  // Identity.* covers Identity.User.Read
  },
  {
    userPermission: 'Exchange.Mailbox.Read',
    requiredPermission: 'Exchange.*',
    matches: true,  // Exchange.* covers all Exchange permissions
  },
  {
    userPermission: 'Identity.User.Read',
    requiredPermission: '*.Read',
    matches: true,  // *.Read covers all Read permissions
  },
];
```

### Practical Permission Implementation Patterns

#### Utility-First Permission Checking

CIPP primarily uses utility functions for permission checking:

```jsx
import { usePermissions } from '/src/hooks/use-permissions';

const UserManagementPage = () => {
  const { checkPermissions } = usePermissions();
  
  const canEditUsers = checkPermissions(['Identity.User.ReadWrite']);
  const canViewUsers = checkPermissions(['Identity.User.Read']);
  
  return (
    <div>
      {canViewUsers && <UserList />}
      {canEditUsers && <Button>Add User</Button>}
    </div>
  );
};
```

#### Direct Permission Checks in Components

Most CIPP pages implement permissions through direct checks:

```jsx
// Real example pattern from CIPP codebase
const MyPage = () => {
  const { data: permissions } = usePermissions();
  
  const pagePermissions = ['Identity.User.Read'];
  const buttonPermissions = ['Identity.User.ReadWrite'];
  
  return (
    <CippContentCard title="User Management">
      <PermissionButton 
        requiredPermissions={buttonPermissions}
        component={Link}
        href="/users/add"
      >
        Add User
      </PermissionButton>
    </CippContentCard>
  );
};
```

## Permission Testing and Debugging

### Permission Testing Patterns

```jsx
describe('Permission System', () => {
  test('exact permission match', () => {
    const userPermissions = ['Identity.User.Read', 'Identity.User.ReadWrite'];
    const requiredPermissions = ['Identity.User.Read'];
    
    expect(hasPermission(userPermissions, requiredPermissions)).toBe(true);
  });
  
  test('wildcard permission match', () => {
    const userPermissions = ['CIPP.Admin.*'];
    const requiredPermissions = ['CIPP.Admin.Settings'];
    
    expect(hasPermission(userPermissions, requiredPermissions)).toBe(true);
  });
});
```

### Component Permission Testing

```jsx
describe('Permission Components', () => {
  test('renders content with sufficient permissions', () => {
    const mockPermissions = {
      checkPermissions: jest.fn(() => true),
      userRoles: ['admin'],
      isAuthenticated: true,
    };
    
    render(
      <PermissionProvider value={mockPermissions}>
        <PermissionCheck requiredPermissions={['CIPP.Admin.*']}>
          <div>Admin Content</div>
        </PermissionCheck>
      </PermissionProvider>
    );
    
    expect(screen.getByText('Admin Content')).toBeInTheDocument();
  });
});
```

This comprehensive authentication and authorization system provides CIPP with enterprise-grade security while maintaining excellent user experience and supporting the complex requirements of Microsoft 365 partner management scenarios.