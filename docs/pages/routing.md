# Routing Patterns and Navigation

CIPP uses Next.js file-based routing with custom navigation patterns, permission-based access control, and dynamic route handling. This document covers how routing works throughout the application and how to implement navigation correctly.

## Next.js File-Based Routing

### Directory Structure

CIPP follows a hierarchical routing structure based on the file system:

```
/src/pages/
├── index.js                          # Route: /
├── domains.js                        # Route: /domains
├── license.js                        # Route: /license
├── identity/
│   ├── administration/
│   │   ├── users/
│   │   │   ├── index.js             # Route: /identity/administration/users
│   │   │   ├── add.jsx              # Route: /identity/administration/users/add
│   │   │   ├── bulk-add.js          # Route: /identity/administration/users/bulk-add
│   │   │   ├── invite.jsx           # Route: /identity/administration/users/invite
│   │   │   ├── patch-wizard.jsx     # Route: /identity/administration/users/patch-wizard
│   │   │   └── user/
│   │   │       ├── index.jsx        # Route: /identity/administration/users/user
│   │   │       ├── edit.jsx         # Route: /identity/administration/users/user/edit
│   │   │       ├── exchange.jsx     # Route: /identity/administration/users/user/exchange
│   │   │       ├── bec.jsx          # Route: /identity/administration/users/user/bec
│   │   │       ├── conditional-access.jsx # Route: /identity/administration/users/user/conditional-access
│   │   │       ├── devices.jsx      # Route: /identity/administration/users/user/devices
│   │   │       └── tabOptions.json  # Tab configuration for user detail page
│   │   └── groups/
│   │       ├── index.js             # Route: /identity/administration/groups
│   │       └── add.jsx              # Route: /identity/administration/groups/add
│   └── reports/
│       └── mfa-report/
│           └── index.js             # Route: /identity/reports/mfa-report
└── tenant/
    ├── administration/
    │   └── tenants/
    │       ├── index.js             # Route: /tenant/administration/tenants
    │       └── add.js               # Route: /tenant/administration/tenants/add
    └── reports/
        └── list-licenses/
            └── index.js             # Route: /tenant/reports/list-licenses
```

### File Naming Conventions

- **`index.js`**: Default route for a directory (e.g., `/users/index.js` → `/users`)
- **`[param].js`**: Dynamic route parameter (e.g., `/users/[id].js` → `/users/123`)
- **`[...slug].js`**: Catch-all dynamic routes (e.g., `/docs/[...slug].js` → `/docs/a/b/c`)
- **Directory names**: Create nested routes (e.g., `/users/edit/` → `/users/edit`)

### Route Types in CIPP

#### 1. List/Index Routes
Display collections of data (typically table pages):

```javascript
// /src/pages/identity/administration/users/index.js
// Route: /identity/administration/users
import { CippTablePage } from "/src/components/CippComponents/CippTablePage.jsx";
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { PermissionButton } from "../../../../utils/permissions";

const UsersPage = () => {
  return (
    <CippTablePage
      title="Users"
      apiUrl="/api/ListGraphRequest"
      // ... table configuration
    />
  );
};

UsersPage.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default UsersPage;
```

#### 2. Add/Create Routes
Form pages for creating new entities:

```javascript
// /src/pages/identity/administration/users/add.jsx
// Route: /identity/administration/users/add
import CippFormPage from "../../../../components/CippFormPages/CippFormPage";
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { useForm } from "react-hook-form";
import { useSettings } from "../../../../hooks/use-settings";

const AddUserPage = () => {
  const userSettingsDefaults = useSettings();
  const formControl = useForm({
    mode: "onBlur",
    defaultValues: {
      tenantFilter: userSettingsDefaults.currentTenant,
    },
  });

  return (
    <CippFormPage
      title="User"
      formControl={formControl}
      postUrl="/api/AddUser"
      // ... form configuration
    />
  );
};

AddUserPage.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default AddUserPage;
```

#### 3. Detail/View Routes
Display detailed information about specific entities:

```javascript
// /src/pages/identity/administration/users/user/index.jsx
// Route: /identity/administration/users/user?userId=123
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { useSettings } from "/src/hooks/use-settings";
import { useRouter } from "next/router";
import { ApiGetCall } from "/src/api/ApiCall";
import { HeaderedTabbedLayout } from "../../../../../layouts/HeaderedTabbedLayout";
import tabOptions from "./tabOptions";

const UserDetailPage = () => {
  const userSettingsDefaults = useSettings();
  const router = useRouter();
  const { userId } = router.query;
  
  const userRequest = ApiGetCall({
    url: `/api/ListUsers?UserId=${userId}&tenantFilter=${router.query.tenantFilter ?? userSettingsDefaults.currentTenant}`,
    queryKey: `ListUsers-${userId}`,
    waiting: !!userId,
  });

  const title = userRequest.isSuccess ? userRequest.data?.[0]?.displayName : "Loading...";
  
  return (
    <HeaderedTabbedLayout
      tabOptions={tabOptions}
      title={title}
      isFetching={userRequest.isLoading}
    >
      {/* User details content */}
    </HeaderedTabbedLayout>
  );
};

UserDetailPage.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default UserDetailPage;
```

#### Tab Configuration (tabOptions.json)

For pages using `HeaderedTabbedLayout`, tab configuration is defined in `tabOptions.json`:

```json
// /src/pages/identity/administration/users/user/tabOptions.json
[
  {
    "label": "Overview",
    "value": "overview"
  },
  {
    "label": "Edit User",
    "value": "edit"
  },
  {
    "label": "Exchange",
    "value": "exchange"
  },
  {
    "label": "Business Email Compromise",
    "value": "bec"
  },
  {
    "label": "Conditional Access",
    "value": "conditional-access"
  },
  {
    "label": "Devices",
    "value": "devices"
  }
]
```

#### 4. Edit Routes
Form pages for modifying existing entities:

```javascript
// /src/pages/identity/administration/users/user/edit.jsx
// Route: /identity/administration/users/user/edit?userId=123
import CippFormPage from "../../../../../components/CippFormPages/CippFormPage";
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { useRouter } from "next/router";
import { useForm } from "react-hook-form";
import { useSettings } from "../../../../../hooks/use-settings";

const EditUserPage = () => {
  const router = useRouter();
  const { userId } = router.query;
  const userSettingsDefaults = useSettings();
  
  const formControl = useForm({
    mode: "onBlur",
    defaultValues: {
      tenantFilter: userSettingsDefaults.currentTenant,
      userId: userId,
    },
  });
  
  return (
    <CippFormPage
      title="User"
      formPageType="Edit"
      formControl={formControl}
      postUrl="/api/EditUser"
      // ... form configuration
    />
  );
};

EditUserPage.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default EditUserPage;
```

## Navigation Patterns

### Programmatic Navigation

#### Using Next.js Router

```javascript
import { useRouter } from "next/router";

const Component = () => {
  const router = useRouter();

  // Simple navigation
  const handleNavigation = () => {
    router.push("/identity/administration/users");
  };

  // Navigation with query parameters
  const handleNavigationWithParams = () => {
    router.push({
      pathname: "/identity/administration/users/user",
      query: { userId: "123", tenantFilter: "domain.com" }
    });
  };

  // Replace route in history (no back button entry)
  const handleReplace = () => {
    router.replace("/dashboard");
  };

  // Go back
  const handleBack = () => {
    router.back();
  };

  return (
    <Box>
      <Button onClick={handleNavigation}>Go to Users</Button>
      <Button onClick={handleNavigationWithParams}>View User</Button>
      <Button onClick={handleBack}>Go Back</Button>
    </Box>
  );
};
```

#### Navigation Hooks

```javascript
import { useRouter } from "next/router";

const NavigationComponent = () => {
  const router = useRouter();

  // Check active route
  const isUsersPage = router.pathname.startsWith("/identity/administration/users");

  // Conditional navigation
  const handleConditionalNav = () => {
    if (isUsersPage) {
      router.push("/identity/administration/groups");
    } else {
      router.push("/identity/administration/users");
    }
  };

  return (
    <Button onClick={handleConditionalNav}>
      {isUsersPage ? "Go to Groups" : "Go to Users"}
    </Button>
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
      <Link
        href={{
          pathname: "/identity/administration/users/user",
          query: { userId: "123" }
        }}
      >
        <Button>View User</Button>
      </Link>

      {/* External link */}
      <Link 
        href="https://entra.microsoft.com" 
        target="_blank" 
        rel="noopener noreferrer"
      >
        <Button startIcon={<LaunchIcon />}>
          Open in Entra
        </Button>
      </Link>
    </Box>
  );
};
```

#### Permission-Gated Links

```javascript
import { PermissionButton } from "/src/utils/permissions";
import Link from "next/link";
import { PersonAdd, GroupAdd } from "@mui/icons-material";
import { Box } from "@mui/material";

const PermissionLinks = () => {
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

      <PermissionButton
        requiredPermissions={["Identity.Group.*"]}
        component={Link}
        href="/identity/administration/groups/add"
        startIcon={<GroupAdd />}
      >
        Add Group
      </PermissionButton>
    </Box>
  );
};
```

## Query Parameters and State

### URL Query Parameters

CIPP extensively uses query parameters for state management:

```javascript
// URL: /identity/administration/users/user?userId=123&tenantFilter=domain.com&tab=exchange

const UserPage = () => {
  const router = useRouter();
  const { userId, tenantFilter, tab } = router.query;

  // Access query parameters
  useEffect(() => {
    if (userId && tenantFilter) {
      // Load user data
      loadUserData(userId, tenantFilter);
    }
  }, [userId, tenantFilter]);

  // Update query parameters
  const handleTabChange = (newTab) => {
    router.push({
      pathname: router.pathname,
      query: { ...router.query, tab: newTab }
    }, undefined, { shallow: true }); // Shallow routing for client-side updates
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

### Common Query Parameter Patterns

#### Tenant Filtering
```javascript
// Most pages include tenantFilter in query
import { ApiGetCall } from "/src/api/ApiCall";

const apiCall = ApiGetCall({
  url: "/api/ListUsers",
  queryKey: "ListUsers",
  data: { 
    tenantFilter: router.query.tenantFilter || userSettingsDefaults.currentTenant 
  },
  waiting: !!userId
});
```

#### Entity Identification
```javascript
// Entity pages use ID parameters
const { userId, groupId, tenantId } = router.query;
```

#### View State Management
```javascript
// UI state via query parameters
const { 
  page = 1, 
  pageSize = 25, 
  sortBy = "displayName",
  filter = "",
  tab = "overview"
} = router.query;
```

#### Form Pre-population
```javascript
// Pre-populate forms from query parameters
const formControl = useForm({
  defaultValues: {
    displayName: router.query.displayName || "",
    email: router.query.email || "",
    department: router.query.department || ""
  }
});
```

## Permission-Based Routing

### Route Protection

CIPP implements route protection at multiple levels:

#### Layout-Level Protection
```javascript
// In DashboardLayout
useEffect(() => {
  if (currentRole.isSuccess && !currentRole.isFetching) {
    const userPermissions = currentRole.data?.permissions;
    
    // Filter navigation based on permissions
    const filteredMenu = filterItemsByRole(nativeMenuItems, userPermissions);
    setMenuItems(filteredMenu);
  }
}, [currentRole.data?.permissions]);
```

#### Page-Level Protection
```javascript
// Protect individual pages
import { usePermissions } from "/src/hooks/use-permissions.js";

const ProtectedPage = () => {
  const { userPermissions } = usePermissions();
  
  if (!userPermissions?.includes("Identity.User.*")) {
    return <UnauthorizedPage />;
  }

  return <UserManagementPage />;
};
```

#### Component-Level Protection
```javascript
// Protect UI components
import { PermissionButton } from "/src/utils/permissions";
import { Box } from "@mui/material";

const UserActions = ({ user }) => {
  return (
    <Box>
      <PermissionButton
        requiredPermissions={["Identity.User.*"]}
        onClick={() => handleEdit(user.id)}
      >
        Edit
      </PermissionButton>
      
      <PermissionButton
        requiredPermissions={["Identity.User.*"]}
        onClick={() => handleDelete(user.id)}
        color="error"
      >
        Delete
      </PermissionButton>
    </Box>
  );
};
```

### Navigation Menu Configuration

Navigation menus are configured with permission requirements:

```javascript
// /src/layouts/config.js
import { UsersIcon } from "@heroicons/react/24/outline";
import { SvgIcon } from "@mui/material";

export const nativeMenuItems = [
  {
    title: "Identity Management",
    type: "header",
    icon: (
      <SvgIcon>
        <UsersIcon />
      </SvgIcon>
    ),
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
          },
          {
            title: "Roles",
            path: "/identity/administration/roles",
            permissions: ["Identity.Role.*"]
          }
        ]
      },
      {
        title: "Reports",
        path: "/identity/reports",
        permissions: ["Identity.User.*", "Identity.Group.*", "Identity.Device.*", "Identity.Role.*", "Identity.AuditLog.*"],
        items: [
          {
            title: "MFA Report",
            path: "/identity/reports/mfa-report",
            permissions: ["Identity.User.*"]
          }
        ]
      }
    ]
  }
];
```

## Dynamic Routes

### Entity Detail Routes

CIPP uses query parameters instead of dynamic route segments for entity details:

```javascript
// Instead of: /users/[id].js
// CIPP uses: /users/user/index.jsx with ?userId=123

const UserDetailPage = () => {
  const router = useRouter();
  const { userId, tenantFilter } = router.query;

  const userData = ApiGetCall({
    url: `/api/ListUsers?UserId=${userId}&tenantFilter=${tenantFilter}`,
    queryKey: `ListUsers-${userId}`,
    waiting: !userId || !tenantFilter
  });

  return (
    <HeaderedTabbedLayout
      title={userData.data?.[0]?.displayName || "Loading..."}
      // ... configuration
    >
      {/* User details content */}
    </HeaderedTabbedLayout>
  );
};
```

### Tenant-Specific Routes

Many routes are tenant-aware:

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

  return (
    <PageComponent tenantFilter={tenantFilter} />
  );
};
```

## Route Transitions and Loading

### Loading States

```javascript
const NavigationWithLoading = () => {
  const router = useRouter();
  const [isNavigating, setIsNavigating] = useState(false);

  useEffect(() => {
    const handleStart = () => setIsNavigating(true);
    const handleComplete = () => setIsNavigating(false);

    router.events.on('routeChangeStart', handleStart);
    router.events.on('routeChangeComplete', handleComplete);
    router.events.on('routeChangeError', handleComplete);

    return () => {
      router.events.off('routeChangeStart', handleStart);
      router.events.off('routeChangeComplete', handleComplete);
      router.events.off('routeChangeError', handleComplete);
    };
  }, [router]);

  return (
    <Box>
      {isNavigating && <LinearProgress />}
      <NavigationButton onClick={() => router.push("/users")}>
        Go to Users
      </NavigationButton>
    </Box>
  );
};
```

### Progressive Enhancement

```javascript
// Graceful degradation for navigation
const EnhancedLink = ({ href, children, ...props }) => {
  const router = useRouter();
  
  const handleClick = (e) => {
    e.preventDefault();
    
    try {
      router.push(href);
    } catch (error) {
      // Fallback to regular navigation
      window.location.href = href;
    }
  };

  return (
    <Link href={href} onClick={handleClick} {...props}>
      {children}
    </Link>
  );
};
```

## Breadcrumbs and Navigation Context

### Automatic Breadcrumb Generation

```javascript
const useBreadcrumbs = () => {
  const pathname = usePathname();
  
  const generateBreadcrumbs = useCallback(() => {
    const segments = pathname.split('/').filter(Boolean);
    
    return segments.map((segment, index) => {
      const path = '/' + segments.slice(0, index + 1).join('/');
      const label = segment.charAt(0).toUpperCase() + segment.slice(1);
      
      return { label, path };
    });
  }, [pathname]);

  return generateBreadcrumbs();
};

const BreadcrumbNavigation = () => {
  const breadcrumbs = useBreadcrumbs();
  
  return (
    <Breadcrumbs>
      <Link href="/">Home</Link>
      {breadcrumbs.map((crumb, index) => (
        <Link key={index} href={crumb.path}>
          {crumb.label}
        </Link>
      ))}
    </Breadcrumbs>
  );
};
```

### Navigation History

```javascript
const useNavigationHistory = () => {
  const [history, setHistory] = useState([]);
  const router = useRouter();

  useEffect(() => {
    const handleRouteChange = (url) => {
      setHistory(prev => [...prev.slice(-9), url]); // Keep last 10 routes
    };

    router.events.on('routeChangeComplete', handleRouteChange);
    return () => router.events.off('routeChangeComplete', handleRouteChange);
  }, [router]);

  const goBack = () => {
    if (history.length > 1) {
      const previousRoute = history[history.length - 2];
      router.push(previousRoute);
    } else {
      router.back();
    }
  };

  return { history, goBack };
};
```

## SEO and Meta Information

### Dynamic Page Titles

```javascript
import Head from "next/head";

const UserDetailPage = () => {
  const { userId } = useRouter().query;
  const { data: userData } = ApiGetCall({
    url: `/api/ListUsers?UserId=${userId}`,
    queryKey: `ListUsers-${userId}`
  });

  const pageTitle = userData 
    ? `${userData.displayName} - User Management`
    : "User Management";

  return (
    <>
      <Head>
        <title>{pageTitle}</title>
        <meta name="description" content={`User details for ${userData?.displayName || 'user'}`} />
      </Head>
      
      <UserDetailInterface userData={userData} />
    </>
  );
};
```

### Canonical URLs

```javascript
const CanonicalPage = () => {
  const router = useRouter();
  
  // Remove query parameters for canonical URL
  const canonicalUrl = `${process.env.NEXT_PUBLIC_BASE_URL}${router.pathname}`;

  return (
    <Head>
      <link rel="canonical" href={canonicalUrl} />
    </Head>
  );
};
```

## Error Handling and Fallbacks

### 404 Pages

```javascript
// /src/pages/404.js
const NotFoundPage = () => {
  const router = useRouter();

  return (
    <Box textAlign="center" py={8}>
      <Typography variant="h1">404</Typography>
      <Typography variant="h4" gutterBottom>
        Page Not Found
      </Typography>
      <Typography color="text.secondary" paragraph>
        The page you're looking for doesn't exist.
      </Typography>
      <Button 
        variant="contained" 
        onClick={() => router.push("/")}
      >
        Go to Dashboard
      </Button>
    </Box>
  );
};

NotFoundPage.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default NotFoundPage;
```

### Route Error Boundaries

```javascript
const RouteErrorBoundary = ({ children }) => {
  return (
    <ErrorBoundary
      fallback={({ error, resetError }) => (
        <Box textAlign="center" py={4}>
          <Typography variant="h5" color="error" gutterBottom>
            Navigation Error
          </Typography>
          <Typography paragraph>
            {error.message}
          </Typography>
          <Button onClick={resetError}>
            Try Again
          </Button>
        </Box>
      )}
    >
      {children}
    </ErrorBoundary>
  );
};
```

## Performance Optimization

### Code Splitting

```javascript
// Automatic code splitting by route
import dynamic from "next/dynamic";

// Lazy load heavy pages
const HeavyReportPage = dynamic(
  () => import("./heavy-report"),
  { 
    ssr: false,
    loading: () => <PageSkeleton />
  }
);
```

### Prefetching

```javascript
// Prefetch likely navigation targets
const NavigationWithPrefetch = () => {
  const router = useRouter();

  useEffect(() => {
    // Prefetch likely next pages
    router.prefetch("/identity/administration/users");
    router.prefetch("/identity/administration/groups");
  }, [router]);

  return (
    <Navigation />
  );
};
```

### Shallow Routing

```javascript
// Use shallow routing for query parameter updates
const handleFilterChange = (newFilter) => {
  router.push({
    pathname: router.pathname,
    query: { ...router.query, filter: newFilter }
  }, undefined, { shallow: true }); // Prevents full page reload
};
```

## Best Practices

1. **Consistent URL Structure**: Follow hierarchical patterns for related pages
2. **Query Parameter Usage**: Use query params for state that should be shareable/bookmarkable
3. **Permission Integration**: Implement proper access controls at all levels
4. **Loading States**: Provide feedback during navigation transitions
5. **Error Handling**: Implement fallbacks for navigation failures
6. **SEO Optimization**: Use proper meta tags and canonical URLs
7. **Performance**: Leverage code splitting and prefetching appropriately
8. **Accessibility**: Ensure navigation is keyboard accessible
9. **Mobile Support**: Test navigation patterns on mobile devices
10. **State Preservation**: Maintain appropriate state across route changes

## Troubleshooting

### Common Issues

**Route Not Found**
- Check file naming and directory structure
- Verify the page component is exported as default
- Ensure `getLayout` is properly configured

**Query Parameters Not Working**
- Check `router.query` is being used correctly
- Verify query parameter names match expectations
- Ensure shallow routing is used for client-side updates

**Permission Issues**
- Check user permissions in browser dev tools
- Verify permission strings match configuration
- Test with different user roles

**Navigation Not Working**
- Check for JavaScript errors in console
- Verify router hooks are being used correctly
- Test fallback navigation methods

## Related Documentation

- [Page Types](./page-types/) - How routing works with different page patterns
- [Layout System](./layouts/) - How layouts integrate with routing
- [Main Layout](./layouts/main-layout.md) - Permission-based navigation
- [Authentication](../getting-started/README.md) - User authentication and routing