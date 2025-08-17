# CIPP Routing & Navigation Guide

This comprehensive guide covers CIPP's routing architecture, implementation patterns, and navigation system. CIPP uses Next.js file-based routing with custom layouts, permission-based access control, and Azure Static Web Apps integration.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [File-Based Routing](#file-based-routing)
3. [Layout System](#layout-system)
4. [Route Types & Patterns](#route-types--patterns)
5. [Navigation Implementation](#navigation-implementation)
6. [Permission-Based Access](#permission-based-access)
7. [Query Parameters & State](#query-parameters--state)
8. [Performance & Optimization](#performance--optimization)
9. [Best Practices](#best-practices)
10. [Troubleshooting](#troubleshooting)

---

## Architecture Overview

CIPP implements a routing system built on Next.js with the following key components:

### Core Technologies
- **Framework**: Next.js with React
- **Routing**: File-based routing with custom layout composition
- **Authentication**: Azure Static Web Apps with Azure AD
- **State Management**: Redux Toolkit with Redux Persist
- **Data Fetching**: TanStack Query (React Query) v5

### System Flow
```
User Request → Azure SWA Auth → Next.js Router → Permission Check → Layout Composition → Page Component
```

### Key Features
- Hierarchical permission-based navigation filtering
- Multi-tenant route context management  
- Responsive layout adaptation
- Route-level code splitting
- Dynamic navigation menu generation

---

## File-Based Routing

### Directory Structure

CIPP follows Next.js conventions in `/src/pages/` with a hierarchical organization:

```
/src/pages/
├── _app.js                          # Global app wrapper
├── _document.js                     # HTML document structure
├── index.js                         # Dashboard homepage
├── domains.js                       # Domains management
├── license.js                       # License information
├── identity/
│   ├── administration/
│   │   ├── index.js                 # Identity admin landing
│   │   ├── users/
│   │   │   ├── index.js             # Users list
│   │   │   ├── add.jsx              # Add user form
│   │   │   ├── bulk-add.js          # Bulk user creation
│   │   │   ├── invite.jsx           # User invitation
│   │   │   ├── patch-wizard.jsx     # User patching wizard
│   │   │   └── user/
│   │   │       ├── index.jsx        # User details
│   │   │       ├── edit.jsx         # Edit user
│   │   │       ├── exchange.jsx     # Exchange settings
│   │   │       ├── bec.jsx          # Business email compromise
│   │   │       ├── conditional-access.jsx # Conditional access
│   │   │       ├── devices.jsx      # User devices
│   │   │       └── tabOptions.json  # Tab configuration
│   │   ├── groups/
│   │   ├── roles/
│   │   └── deleted-items/
│   └── reports/
│       └── mfa-report/
├── tenant/
│   ├── administration/
│   ├── conditional/
│   └── gdap-management/
├── email/
├── security/
├── endpoint/
├── teams-share/
└── cipp/
```

### File Naming Conventions

| Pattern | Example | Route Generated |
|---------|---------|----------------|
| `index.js` | `/users/index.js` | `/users` |
| `[param].js` | `/users/[id].js` | `/users/123` |
| `[...slug].js` | `/docs/[...slug].js` | `/docs/a/b/c` |
| **Nested directories** | `/users/edit/index.js` | `/users/edit` |

**File Extensions**: CIPP uses both `.js` and `.jsx` extensions. Use `.jsx` for components with significant JSX content.

---

## Layout System

CIPP uses a flexible layout composition system allowing pages to define their own layout requirements through the `getLayout` pattern.

### App Wrapper (_app.js)

The global app wrapper orchestrates the entire application:

```javascript
const App = (props) => {
  const { Component, emotionCache = clientSideEmotionCache, pageProps } = props;
  const getLayout = Component.getLayout ?? ((page) => page);
  
  return (
    <CacheProvider value={emotionCache}>
      <ReduxProvider store={store}>
        <QueryClientProvider client={queryClient}>
          <SettingsProvider>
            <LocalizationProvider dateAdapter={AdapterDateFns}>
              <SettingsConsumer>
                {(settings) => (
                  <ThemeProvider theme={createTheme(settings)}>
                    <RTL direction={settings.direction}>
                      <CssBaseline />
                      <ErrorBoundary FallbackComponent={Error500}>
                        <PrivateRoute>
                          {getLayout(<Component {...pageProps} />)}
                        </PrivateRoute>
                      </ErrorBoundary>
                    </RTL>
                  </ThemeProvider>
                )}
              </SettingsConsumer>
            </LocalizationProvider>
          </SettingsProvider>
        </QueryClientProvider>
      </ReduxProvider>
    </CacheProvider>
  );
};
```

### Layout Types

#### 1. DashboardLayout (Default)
Primary layout with sidebar navigation and tenant context:

```javascript
// /src/layouts/index.js
export const Layout = (props) => {
  const { children, allTenantsSupport = true } = props;
  const mdDown = useMediaQuery((theme) => theme.breakpoints.down("md"));
  
  return (
    <>
      <TopNav onNavOpen={mobileNav.handleOpen} />
      {mdDown ? (
        <MobileNav items={menuItems} />
      ) : (
        <SideNav items={menuItems} />
      )}
      <LayoutRoot>
        <LayoutContainer>
          {!allTenantsSupport && currentTenant === "AllTenants" ? (
            <TenantNotSupportedMessage />
          ) : (
            children
          )}
        </LayoutContainer>
      </LayoutRoot>
    </>
  );
};
```

#### 2. TabbedLayout
For pages requiring tab navigation:

```javascript
// /src/layouts/TabbedLayout.jsx
export const TabbedLayout = (props) => {
  const { tabOptions, children } = props;
  const router = useRouter();
  
  const handleTabsChange = (event, value) => {
    router.push(value);
  };
  
  return (
    <Box>
      <Tabs onChange={handleTabsChange} value={currentTab?.path}>
        {tabOptions.map((option) => (
          <Tab key={option.path} label={option.label} value={option.path} />
        ))}
      </Tabs>
      {children}
    </Box>
  );
};
```

#### 3. HeaderedTabbedLayout
Combines header content with tabbed navigation:

```javascript
// /src/layouts/HeaderedTabbedLayout.jsx
export const HeaderedTabbedLayout = (props) => {
  const { title, tabOptions, children, isFetching } = props;
  
  return (
    <Box>
      <Header title={title} loading={isFetching} />
      <TabbedLayout tabOptions={tabOptions}>
        {children}
      </TabbedLayout>
    </Box>
  );
};
```

### Layout Composition Pattern

Pages define their layout through the `getLayout` static method:

```javascript
const UsersPage = () => {
  return (
    <CippTablePage
      title="Users"
      apiUrl="/api/ListGraphRequest"
      // ... configuration
    />
  );
};

// Layout assignment
UsersPage.getLayout = (page) => (
  <Layout allTenantsSupport={false}>
    <TabbedLayout tabOptions={userTabOptions}>
      {page}
    </TabbedLayout>
  </Layout>
);

export default UsersPage;
```

---

## Route Types & Patterns

### 1. List/Index Routes
Display collections of data using `CippTablePage`:

```javascript
// /src/pages/identity/administration/users/index.js
import { CippTablePage } from "../../components/CippComponents/CippTablePage.jsx";
import { Layout as DashboardLayout } from "/src/layouts/index.js";

const UsersPage = () => {
  return (
    <CippTablePage
      title="Users"
      apiUrl="/api/ListGraphRequest"
      apiData={{ 
        endpoint: "users",
        $select: "id,displayName,userPrincipalName,accountEnabled"
      }}
      columns={[
        { field: "displayName", headerName: "Display Name" },
        { field: "userPrincipalName", headerName: "UPN" },
        { field: "accountEnabled", headerName: "Enabled" }
      ]}
    />
  );
};

UsersPage.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default UsersPage;
```

### 2. Add/Create Routes
Form pages for creating new entities using `CippFormPage`:

```javascript
// /src/pages/identity/administration/users/add.jsx
import CippFormPage from "../../../../components/CippFormPages/CippFormPage";
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { useForm } from "react-hook-form";

const AddUserPage = () => {
  const formControl = useForm({
    mode: "onBlur",
    defaultValues: {
      tenantFilter: userSettingsDefaults.currentTenant,
    },
  });

  return (
    <CippFormPage
      title="Add User"
      formControl={formControl}
      postUrl="/api/AddUser"
      fields={[
        { name: "displayName", label: "Display Name", required: true },
        { name: "userPrincipalName", label: "User Principal Name", required: true },
        { name: "password", label: "Password", type: "password" }
      ]}
    />
  );
};

AddUserPage.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default AddUserPage;
```

### 3. Detail/View Routes
Display detailed information about specific entities:

```javascript
// /src/pages/identity/administration/users/user/index.jsx
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { HeaderedTabbedLayout } from "../../../../../layouts/HeaderedTabbedLayout";
import { ApiGetCall } from "/src/api/ApiCall";
import tabOptions from "./tabOptions.json";

const UserDetailPage = () => {
  const { userId, tenantFilter } = useRouter().query;
  
  const userRequest = ApiGetCall({
    url: `/api/ListUsers?UserId=${userId}&tenantFilter=${tenantFilter}`,
    queryKey: `ListUsers-${userId}`,
    waiting: !!userId,
  });

  return (
    <HeaderedTabbedLayout
      title={userRequest.data?.[0]?.displayName || "Loading..."}
      tabOptions={tabOptions}
      isFetching={userRequest.isLoading}
    >
      {/* User details content */}
    </HeaderedTabbedLayout>
  );
};

UserDetailPage.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default UserDetailPage;
```

### 4. Tab Configuration
For pages using `HeaderedTabbedLayout`, define tabs in `tabOptions.json`:

```json
[
  {
    "label": "View User",
    "path": "/identity/administration/users/user"
  },
  {
    "label": "Edit User", 
    "path": "/identity/administration/users/user/edit"
  },
  {
    "label": "Exchange Settings",
    "path": "/identity/administration/users/user/exchange"
  }
]
```

---

## Navigation Implementation

### Programmatic Navigation

#### Using Next.js Router
```javascript
import { useRouter } from "next/router";

const NavigationExample = () => {
  const router = useRouter();

  // Simple navigation
  const goToUsers = () => {
    router.push("/identity/administration/users");
  };

  // Navigation with query parameters
  const viewUser = (userId, tenantFilter) => {
    router.push({
      pathname: "/identity/administration/users/user",
      query: { userId, tenantFilter }
    });
  };

  // Replace current route (no back button entry)
  const redirect = () => {
    router.replace("/dashboard");
  };

  return (
    <Box>
      <Button onClick={goToUsers}>View Users</Button>
      <Button onClick={() => viewUser("123", "domain.com")}>View User</Button>
    </Box>
  );
};
```

### Link-Based Navigation

#### Next.js Link Component
```javascript
import Link from "next/link";

const NavigationLinks = () => {
  return (
    <Box>
      {/* Simple link */}
      <Link href="/identity/administration/users">
        <Button>Users</Button>
      </Link>

      {/* Link with query parameters */}
      <Link href={{
        pathname: "/identity/administration/users/user",
        query: { userId: "123", tenantFilter: "domain.com" }
      }}>
        <Button>View User</Button>
      </Link>
    </Box>
  );
};
```

#### Permission-Protected Links
```javascript
import { PermissionButton } from "/src/utils/permissions";

const ProtectedLinks = () => {
  return (
    <Box>
      <PermissionButton
        requiredPermissions={["Identity.User.*"]}
        component={Link}
        href="/identity/administration/users/add"
        startIcon={<PersonAdd />}
      >
        Add User
      </PermissionButton>
    </Box>
  );
};
```

---

## Permission-Based Access

### Navigation Menu Configuration

Navigation structure is defined in `/src/layouts/config.js` with permission requirements:

```javascript
export const nativeMenuItems = [
  {
    title: "Identity Management",
    type: "header",
    icon: <SvgIcon><UsersIcon /></SvgIcon>,
    permissions: ["Identity.*"],
    items: [
      {
        title: "Administration",
        path: "/identity/administration",
        permissions: ["Identity.User.*"],
        items: [
          {
            title: "Users",
            path: "/identity/administration/users",
            permissions: ["Identity.User.*"]
          },
          {
            title: "Groups", 
            path: "/identity/administration/groups",
            permissions: ["Identity.Group.*"]
          }
        ]
      }
    ]
  }
];
```

### Permission Filtering Logic

Navigation items are filtered based on user permissions:

```javascript
const filterItemsByRole = (items, userPermissions) => {
  return items
    .map((item) => {
      // Check permissions with pattern matching
      if (item.permissions && item.permissions.length > 0) {
        const hasPermission = userPermissions?.some((userPerm) => {
          return item.permissions.some((requiredPerm) => {
            // Exact match
            if (userPerm === requiredPerm) return true;

            // Wildcard pattern matching
            if (requiredPerm.includes("*")) {
              const regexPattern = requiredPerm
                .replace(/\./g, "\\.")
                .replace(/\*/g, ".*");
              const regex = new RegExp(`^${regexPattern}$`);
              return regex.test(userPerm);
            }

            return false;
          });
        });
        if (!hasPermission) return null;
      }
      
      // Recursively filter sub-items
      if (item.items && item.items.length > 0) {
        const filteredSubItems = filterItemsByRole(item.items, userPermissions);
        return { ...item, items: filteredSubItems };
      }

      return item;
    })
    .filter(Boolean);
};
```

### Permission Pattern Matching

CIPP supports wildcard permissions:
- `CIPP.Core.*` - Matches any CIPP core permission
- `Identity.*` - Matches any identity management permission  
- `Exchange.Mailbox.*` - Matches any mailbox-related permission

### Component-Level Protection

```javascript
import { usePermissions } from "/src/hooks/use-permissions.js";

const ProtectedComponent = () => {
  const { checkPermissions } = usePermissions();
  
  const canEditUsers = checkPermissions(["Identity.User.Edit"]);
  const canDeleteUsers = checkPermissions(["Identity.User.Delete"]);
  
  return (
    <Stack direction="row" spacing={1}>
      {canEditUsers && (
        <Button onClick={handleEdit}>Edit</Button>
      )}
      {canDeleteUsers && (
        <Button onClick={handleDelete}>Delete</Button>
      )}
    </Stack>
  );
};
```

---

## Query Parameters & State

### URL Query Parameter Patterns

CIPP extensively uses query parameters for state management:

```javascript
// Common patterns:
// Entity identification: ?userId=123&tenantFilter=domain.com
// View state: ?page=1&pageSize=25&sortBy=displayName&filter=john
// Tab state: ?tab=exchange&section=mailbox
// Form pre-population: ?displayName=John&department=IT
```

### Query Parameter Handling

```javascript
const QueryParameterExample = () => {
  const router = useRouter();
  const { userId, tenantFilter, tab = "overview" } = router.query;

  // Access query parameters
  useEffect(() => {
    if (userId && tenantFilter) {
      loadUserData(userId, tenantFilter);
    }
  }, [userId, tenantFilter]);

  // Update query parameters (shallow routing)
  const handleTabChange = (newTab) => {
    router.push({
      pathname: router.pathname,
      query: { ...router.query, tab: newTab }
    }, undefined, { shallow: true });
  };

  return (
    <UserInterface 
      userId={userId}
      tenantFilter={tenantFilter}
      activeTab={tab}
      onTabChange={handleTabChange}
    />
  );
};
```

### Tenant Context Management

```javascript
const TenantAwarePage = () => {
  const settings = useSettings();
  const router = useRouter();
  
  // Get tenant from query or settings
  const tenantFilter = router.query.tenantFilter || settings.currentTenant;

  // Redirect if no tenant selected
  useEffect(() => {
    if (!tenantFilter) {
      router.push("/tenant/administration/tenants");
    }
  }, [tenantFilter]);

  return <PageComponent tenantFilter={tenantFilter} />;
};
```

---

## Performance & Optimization

### Code Splitting

Next.js provides automatic code splitting by route:

```javascript
// Automatic splitting per page
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(
  () => import('../components/HeavyComponent'),
  { 
    ssr: false,
    loading: () => <LoadingSkeleton />
  }
);
```

### Navigation Caching

Navigation state and permissions are cached via React Query:
- Navigation menu items cached after permission calculation
- User permissions cached with 2-minute stale time
- Route transitions don't refetch unless data is stale

### Shallow Routing

Use shallow routing for query parameter updates that don't require data refetching:

```javascript
// Prevents full page reload for UI state changes
const handleFilterChange = (newFilter) => {
  router.push({
    pathname: router.pathname,
    query: { ...router.query, filter: newFilter }
  }, undefined, { shallow: true });
};
```

---

## Best Practices

### 1. Layout Assignment
Always use consistent layout assignment patterns:

```javascript
PageComponent.getLayout = (page) => (
  <Layout allTenantsSupport={false}>
    <TabbedLayout tabOptions={tabOptions}>
      {page}
    </TabbedLayout>
  </Layout>
);
```

### 2. Permission Declaration
Clearly declare permission requirements:

```javascript
{
  title: "User Management",
  path: "/identity/administration/users",
  permissions: ["Identity.User.Read", "Identity.User.List"],
}
```

### 3. Route Naming Conventions
Follow hierarchical naming patterns:

```javascript
// ✅ Good
/identity/administration/users
/tenant/conditional/policies

// ❌ Avoid  
/users-admin-identity
/policies-conditional-tenant
```

### 4. Error Handling
Implement comprehensive error boundaries:

```javascript
<ErrorBoundary FallbackComponent={Error500}>
  <PrivateRoute>
    {getLayout(<Component {...pageProps} />)}
  </PrivateRoute>
</ErrorBoundary>
```

### 5. Query Parameter Management
Use descriptive, consistent parameter names:

```javascript
// ✅ Good
?userId=123&tenantFilter=domain.com&activeTab=exchange

// ❌ Avoid
?id=123&t=domain.com&tab=1
```

---

## Troubleshooting

### Common Issues

#### Route Not Found
- **Check**: File naming and directory structure
- **Verify**: Page component is exported as default
- **Ensure**: `getLayout` is properly configured

#### Query Parameters Not Working
- **Check**: `router.query` usage is correct
- **Verify**: Parameter names match expectations
- **Ensure**: Shallow routing is used for client-side updates

#### Permission Issues
- **Check**: User permissions in browser dev tools
- **Verify**: Permission strings match configuration exactly
- **Test**: With different user roles

#### Navigation Not Working
- **Check**: JavaScript errors in console
- **Verify**: Router hooks are used correctly
- **Test**: Fallback navigation methods

### Debugging Tools

#### Router Information
```javascript
// Debug current route state
const router = useRouter();
console.log({
  pathname: router.pathname,
  query: router.query,
  asPath: router.asPath,
  isReady: router.isReady
});
```

#### Permission Debugging
```javascript
// Debug user permissions
const { userPermissions } = usePermissions();
console.log("User permissions:", userPermissions);
```

---

## Related Documentation

- **[Component Library](./components/README.md)** - CIPP component usage patterns
- **[Architecture Overview](./architecture/)** - System design and patterns
- **[Authentication](./authentication.md)** - User authentication flows
- **[API Integration](./api/)** - Backend API patterns

---

*This guide covers CIPP's complete routing system. For specific implementation questions or to report issues, please refer to the project's GitHub repository.*