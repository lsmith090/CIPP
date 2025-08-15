# Routing & Navigation Architecture

## Overview

CIPP implements a sophisticated routing system built on Next.js file-based routing, enhanced with custom layout composition, permission-based navigation filtering, and Azure Static Web Apps integration. The architecture supports complex multi-tenant scenarios while maintaining excellent performance and security.

## Next.js Routing Foundation

CIPP leverages Next.js 15+ file-based routing with custom layout patterns for consistent user experience across all Microsoft 365 partner management features.

### File Structure

The routing structure follows Next.js conventions in `/src/pages/`:

```
src/pages/
├── _app.js                 # Global app wrapper with providers
├── index.js                # Dashboard homepage
├── identity/
│   ├── administration/
│   │   ├── users/
│   │   │   └── index.js    # /identity/administration/users
│   │   ├── groups/
│   │   └── devices/
│   └── reports/
├── tenant/
│   ├── administration/
│   ├── standards/
│   └── conditional/
├── email/
├── security/
├── endpoint/
├── teams-share/
├── tools/
└── cipp/
```

### App Wrapper Implementation

The `/src/pages/_app.js` file orchestrates the entire application routing system:

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
                {(settings) => {
                  const theme = createTheme({
                    colorPreset: "orange",
                    direction: settings.direction,
                    paletteMode: settings.currentTheme?.value !== "browser"
                      ? settings.currentTheme?.value
                      : preferredTheme,
                    contrast: "high",
                  });

                  return (
                    <ThemeProvider theme={theme}>
                      <RTL direction={settings.direction}>
                        <CssBaseline />
                        <ErrorBoundary FallbackComponent={Error500}>
                          <PrivateRoute>
                            {getLayout(<Component {...pageProps} />)}
                          </PrivateRoute>
                        </ErrorBoundary>
                      </RTL>
                    </ThemeProvider>
                  );
                }}
              </SettingsConsumer>
            </LocalizationProvider>
          </SettingsProvider>
        </QueryClientProvider>
      </ReduxProvider>
    </CacheProvider>
  );
};
```

## Layout System

CIPP uses a flexible layout composition system that allows pages to define their own layout requirements through the `getLayout` pattern.

### Layout Types

#### 1. DashboardLayout (Default)

The primary layout located in `/src/layouts/index.js` provides the full dashboard experience:

```javascript
export const Layout = (props) => {
  const { children, allTenantsSupport = true } = props;
  const mdDown = useMediaQuery((theme) => theme.breakpoints.down("md"));
  const settings = useSettings();
  const mobileNav = useMobileNav();

  return (
    <>
      {hideSidebar === false && (
        <>
          <TopNav onNavOpen={mobileNav.handleOpen} openNav={mobileNav.open} />
          {mdDown && (
            <MobileNav items={menuItems} onClose={mobileNav.handleClose} open={mobileNav.open} />
          )}
          {!mdDown && <SideNav items={menuItems} onPin={handleNavPin} pinned={!!settings.pinNav} />}
        </>
      )}
      <LayoutRoot
        sx={{
          pl: {
            md: (hideSidebar ? "0" : offset) + "px",
          },
        }}
      >
        <LayoutContainer>
          {(currentTenant === "AllTenants" || !currentTenant) && !allTenantsSupport ? (
            <Box sx={{ flexGrow: 1, py: 4 }}>
              <Container maxWidth={false}>
                <Grid container spacing={3}>
                  <Grid size={6}>
                    <CippImageCard
                      title="Not supported"
                      imageUrl="/assets/illustrations/undraw_website_ij0l.svg"
                      text="The page does not support all Tenants, please select a different tenant using the tenant selector."
                    />
                  </Grid>
                </Grid>
              </Container>
            </Box>
          ) : (
            <>{children}</>
          )}
          <Footer />
        </LayoutContainer>
      </LayoutRoot>
    </>
  );
};
```

**Key Features:**
- Responsive sidebar with pin/unpin functionality
- Mobile navigation drawer
- Tenant context validation
- Conditional content rendering based on tenant support

#### 2. TabbedLayout

For pages requiring tab navigation (`/src/layouts/TabbedLayout.jsx`):

```javascript
export const TabbedLayout = (props) => {
  const { tabOptions, children } = props;
  const router = useRouter();
  const pathname = usePathname();

  const handleTabsChange = (event, value) => {
    router.push(value);
  };

  const currentTab = tabOptions.find((option) => option.path === pathname);

  return (
    <Box sx={{ flexGrow: 1, py: 4 }}>
      <Stack spacing={2}>
        <div>
          <Tabs onChange={handleTabsChange} value={currentTab?.path} variant="scrollable">
            {tabOptions.map((option) => (
              <Tab key={option.path} label={option.label} value={option.path} />
            ))}
          </Tabs>
          <Divider />
        </div>
      </Stack>
      {children}
    </Box>
  );
};
```

#### 3. HeaderedTabbedLayout

Combination of header content with tabbed navigation (`/src/layouts/HeaderedTabbedLayout.jsx`).

### Layout Composition Pattern

Pages define their layout through the `getLayout` static method:

```javascript
// Example page component
const UsersPage = () => {
  return (
    <Container maxWidth={false}>
      {/* Page content */}
    </Container>
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

## Permission-Based Navigation

CIPP implements a sophisticated permission system that filters navigation items based on user roles and permissions.

### Navigation Configuration

Navigation structure is defined in `/src/layouts/config.js`:

```javascript
export const nativeMenuItems = [
  {
    title: "Dashboard",
    path: "/",
    icon: <SvgIcon><HomeIcon /></SvgIcon>,
    permissions: ["CIPP.Core.*"],
  },
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
            permissions: ["Identity.User.*"],
          },
          {
            title: "Groups",
            path: "/identity/administration/groups",
            permissions: ["Identity.Group.*"],
          },
          // ... more items
        ],
      },
    ],
  },
  // ... more sections
];
```

### Permission Filtering Logic

The navigation filtering is implemented in the main Layout component:

```javascript
useEffect(() => {
  if (currentRole.isSuccess && !currentRole.isFetching) {
    const userRoles = currentRole.data?.clientPrincipal?.userRoles;
    const userPermissions = currentRole.data?.permissions;
    
    if (!userRoles) {
      setMenuItems([]);
      setHideSidebar(true);
      return;
    }
    
    const filterItemsByRole = (items) => {
      return items
        .map((item) => {
          // Check roles
          if (item.roles && item.roles.length > 0) {
            const hasRole = item.roles.some((requiredRole) => 
              userRoles.includes(requiredRole)
            );
            if (!hasRole) return null;
          }

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
          } else {
            return null;
          }
          
          // Recursively filter sub-items
          if (item.items && item.items.length > 0) {
            const filteredSubItems = filterItemsByRole(item.items).filter(Boolean);
            return { ...item, items: filteredSubItems };
          }

          return item;
        })
        .filter(Boolean);
    };
    
    const filteredMenu = filterItemsByRole(nativeMenuItems);
    setMenuItems(filteredMenu);
  }
}, [
  currentRole.isSuccess,
  currentRole.data?.clientPrincipal?.userRoles,
  currentRole.data?.permissions,
  currentRole.isFetching,
]);
```

### Permission Pattern Matching

CIPP supports wildcard permissions for flexible access control:

- `CIPP.Core.*` - Matches any CIPP core permission
- `Identity.*` - Matches any identity management permission
- `Exchange.Mailbox.*` - Matches any mailbox-related permission

## Navigation Components

### SideNav Component

The main navigation sidebar (`/src/layouts/side-nav.js`):

**Features:**
- Hierarchical menu structure
- Pinnable/collapsible design
- Permission-filtered items
- Active state management
- Material-UI integration

### MobileNav Component

Mobile-optimized navigation drawer (`/src/layouts/mobile-nav.js`):

**Features:**
- Responsive breakpoint activation
- Touch-friendly interface
- Same permission filtering as desktop
- Auto-close on route change

### TopNav Component

Application header (`/src/layouts/top-nav.js`):

**Features:**
- Tenant selector integration
- User profile management
- Organization context
- Mobile menu trigger

## Azure Static Web Apps Integration

CIPP integrates with Azure Static Web Apps for authentication and route protection.

### Route Configuration

The `staticwebapp.config.json` defines route-level security:

```json
{
  "routes": [
    {
      "route": "_next/static/*",
      "headers": {
        "cache-control": "must-revalidate, max-age=15770000"
      }
    },
    {
      "route": "/login",
      "rewrite": "/.auth/login/aad"
    },
    {
      "route": "/logout",
      "redirect": "/.auth/logout?post_logout_redirect_uri=/LogoutRedirect"
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
  "navigationFallback": {
    "rewrite": "index.html",
    "exclude": [
      "_next/static/*",
      "/css/*",
      "public/*",
      "assets/*",
      "favicon.ico",
      "robots.txt",
      "sitemap.xml",
      "manifest.json"
    ]
  },
  "responseOverrides": {
    "401": {
      "redirect": "/.auth/login/aad?post_login_redirect_uri=.referrer",
      "statusCode": 302
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

### Authentication Flow Integration

The routing system integrates with Azure SWA authentication:

1. **Unauthenticated Access** → Redirect to Azure AD login
2. **Authentication Check** → PrivateRoute component validation
3. **Permission Loading** → React Query caches user permissions
4. **Navigation Filtering** → Menu items filtered by permissions
5. **Route Protection** → Per-page role/permission checks

## Dynamic Routing Patterns

### Tenant Context Routing

Many routes support tenant-specific contexts:

```javascript
// URL patterns
/identity/administration/users                    // All tenants view
/identity/administration/users?tenantFilter=123   // Specific tenant

// Component handling
const currentTenant = settings?.currentTenant;
const allTenantsSupport = props.allTenantsSupport ?? true;

if ((currentTenant === "AllTenants" || !currentTenant) && !allTenantsSupport) {
  return <TenantNotSupportedMessage />;
}
```

### Entity-Specific Routes

CIPP supports various entity-specific routing patterns:

```javascript
// User management
/identity/administration/users/[userId]          // User details
/identity/administration/users/[userId]/edit     // User editing

// Tenant administration
/tenant/administration/tenants/[tenantId]        // Tenant details
/tenant/conditional/policies/[policyId]          // Policy details

// Dynamic route handling in Next.js
export async function getServerSideProps({ params }) {
  return {
    props: {
      entityId: params.entityId,
    },
  };
}
```

### Tabbed Route Integration

Complex features use tabbed layouts with route synchronization:

```javascript
const tabOptions = [
  {
    label: "Users",
    path: "/identity/administration/users",
  },
  {
    label: "Groups", 
    path: "/identity/administration/groups",
  },
  {
    label: "Devices",
    path: "/identity/administration/devices",
  },
];

// Tab change triggers router navigation
const handleTabsChange = (event, value) => {
  router.push(value);
};
```

### Query Parameter Handling

CIPP routing supports complex query parameter patterns:

```javascript
// Search and filtering
/identity/administration/users?search=john&status=active&role=admin

// Pagination
/tenant/administration/tenants?page=2&limit=50

// Multi-tenant operations
/email/administration/mailboxes?tenantFilter=contoso.com&type=shared
```

## Route Protection

### Page-Level Protection

Individual pages can specify protection requirements:

```javascript
// Role-based protection
const AdminPage = () => {
  return <AdminContent />;
};

AdminPage.getLayout = (page) => (
  <Layout>
    <PrivateRoute routeType="admin">
      {page}
    </PrivateRoute>
  </Layout>
);
```

### Component-Level Protection

Components can use permission hooks for fine-grained control:

```javascript
import { usePermissions } from "/src/hooks/use-permissions";

const UserActions = ({ user }) => {
  const { checkPermissions } = usePermissions();
  
  const canEditUsers = checkPermissions(["Identity.User.Edit"]);
  const canDeleteUsers = checkPermissions(["Identity.User.Delete"]);
  
  return (
    <Stack direction="row" spacing={1}>
      {canEditUsers && (
        <Button onClick={() => handleEdit(user)}>Edit</Button>
      )}
      {canDeleteUsers && (
        <Button onClick={() => handleDelete(user)}>Delete</Button>
      )}
    </Stack>
  );
};
```

## Performance Optimizations

### Route-Level Code Splitting

CIPP leverages Next.js automatic code splitting:

```javascript
// Automatic splitting per page
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(
  () => import('../components/HeavyComponent'),
  { loading: () => <LoadingSkeleton /> }
);
```

### Navigation Caching

Navigation state and permissions are cached via React Query:

- Navigation menu items cached after permission calculation
- User permissions cached with 2-minute stale time
- Route transitions don't refetch unless data is stale

### Responsive Route Handling

Different layouts activate based on screen size:

```javascript
const mdDown = useMediaQuery((theme) => theme.breakpoints.down("md"));

return (
  <>
    {mdDown && (
      <MobileNav items={menuItems} onClose={mobileNav.handleClose} open={mobileNav.open} />
    )}
    {!mdDown && (
      <SideNav items={menuItems} onPin={handleNavPin} pinned={!!settings.pinNav} />
    )}
  </>
);
```

## Best Practices

### Layout Assignment

```javascript
// Consistent layout assignment pattern
PageComponent.getLayout = (page) => (
  <Layout allTenantsSupport={false}>
    <TabbedLayout tabOptions={tabOptions}>
      {page}
    </TabbedLayout>
  </Layout>
);
```

### Permission Declaration

```javascript
// Clear permission requirements in navigation config
{
  title: "User Management",
  path: "/identity/administration/users",
  permissions: ["Identity.User.Read", "Identity.User.List"],
  roles: ["admin", "editor"], // Optional role-based access
}
```

### Route Naming Conventions

```javascript
// Follow hierarchical naming patterns
/feature/category/action     // Standard pattern
/identity/administration/users     // ✓ Good
/users-admin-identity             // ✗ Avoid
```

### Error Handling

```javascript
// Comprehensive error boundaries
<ErrorBoundary FallbackComponent={Error500}>
  <PrivateRoute routeType={routeType}>
    {getLayout(<Component {...pageProps} />)}
  </PrivateRoute>
</ErrorBoundary>
```

This routing architecture provides CIPP with the flexibility to handle complex Microsoft 365 partner management scenarios while maintaining security, performance, and excellent user experience across desktop and mobile devices.