# Main Layout - DashboardLayout

The `DashboardLayout` is the foundational layout component that wraps all pages in CIPP. It provides the core application structure including navigation, authentication, tenant management, and responsive design patterns.

## Component Overview

**Location**: `/src/layouts/index.js`

**Primary Responsibilities**:
- Application-wide navigation (side nav, top nav, mobile nav)
- User authentication and session management
- Tenant selection and context management
- Permission-based UI filtering
- Responsive layout adaptation
- Global state management integration

## Basic Usage

```javascript
import { Layout as DashboardLayout } from "/src/layouts/index.js";

const Page = () => {
  return (
    <div>
      {/* Your page content */}
    </div>
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default Page;
```

## Props and Configuration

### Layout Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | Page content to render |
| `allTenantsSupport` | `boolean` | `true` | Whether page supports "All Tenants" mode |

### Usage Examples

```javascript
// Standard layout (supports all tenants)
Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;

// Single tenant only
Page.getLayout = (page) => (
  <DashboardLayout allTenantsSupport={false}>
    {page}
  </DashboardLayout>
);
```

## Architecture Components

### 1. Navigation System

The layout includes three navigation components:

#### TopNav
- **Location**: `/src/layouts/top-nav.js`
- **Features**: 
  - Tenant selector dropdown
  - User account menu
  - Mobile navigation trigger
  - Notifications panel

#### SideNav (Desktop)
- **Location**: `/src/layouts/side-nav.js`
- **Features**:
  - Hierarchical menu structure
  - Permission-based filtering
  - Collapsible/pinnable states
  - Active state management

#### MobileNav (Mobile)
- **Location**: `/src/layouts/mobile-nav.js`
- **Features**:
  - Drawer-based navigation
  - Touch-optimized interface
  - Same menu structure as desktop

### 2. Authentication Flow

```javascript
// Authentication check via Static Web Apps
const swaStatus = ApiGetCall({
  url: "/.auth/me",
  queryKey: "authmeswa",
  staleTime: 120000,
  refetchOnWindowFocus: true,
});

// User role and permission retrieval
const currentRole = ApiGetCall({
  url: "/api/me",
  queryKey: "authmecipp",
  waiting: !swaStatus.isSuccess || swaStatus.data?.clientPrincipal === null,
});
```

### 3. Permission System

The layout implements sophisticated permission filtering:

```javascript
const filterItemsByRole = (items) => {
  return items
    .map((item) => {
      // Role-based filtering
      if (item.roles && item.roles.length > 0) {
        const hasRole = item.roles.some((requiredRole) => 
          userRoles.includes(requiredRole)
        );
        if (!hasRole) return null;
      }

      // Permission-based filtering with pattern matching
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
        const filteredSubItems = filterItemsByRole(item.items).filter(Boolean);
        return { ...item, items: filteredSubItems };
      }

      return item;
    })
    .filter(Boolean);
};
```

## Responsive Behavior

### Breakpoint Constants

```javascript
const SIDE_NAV_WIDTH = 270;           // Desktop side nav width
const SIDE_NAV_PINNED_WIDTH = 50;     // Pinned side nav width  
const TOP_NAV_HEIGHT = 50;            // Top navigation height
```

### Layout Adaptation

```javascript
const LayoutRoot = styled("div")(({ theme }) => ({
  backgroundColor: theme.palette.background.default,
  display: "flex",
  flex: "1 1 auto",
  maxWidth: "100%",
  paddingTop: TOP_NAV_HEIGHT,
  [theme.breakpoints.up("lg")]: {
    paddingLeft: SIDE_NAV_WIDTH,      // Desktop offset
  },
}));
```

### Mobile Navigation

```javascript
const useMobileNav = () => {
  const pathname = usePathname();
  const [open, setOpen] = useState(false);

  // Auto-close on route change
  useEffect(() => {
    if (open) setOpen(false);
  }, [pathname]);

  return { handleClose, handleOpen, open };
};
```

## State Management Integration

### Settings Context

The layout integrates with the global settings context:

```javascript
const settings = useSettings();

// Available settings
const {
  currentTenant,     // Selected tenant
  pinNav,           // Navigation pin state
  theme,            // Theme preferences
  bookmarks,        // User bookmarks
  showDevtools     // Developer tools visibility
} = settings;
```

### User Settings Sync

```javascript
const userSettingsAPI = ApiGetCall({
  url: "/api/ListUserSettings",
  queryKey: "userSettings",
});

useEffect(() => {
  if (userSettingsAPI.isSuccess && !userSettingsAPI.isFetching) {
    // Clean up problematic settings
    if (userSettingsAPI.data.offboardingDefaults?.user) {
      delete userSettingsAPI.data.offboardingDefaults.user;
    }
    
    // Merge with current settings
    settings.handleUpdate({
      ...userSettingsAPI.data,
      bookmarks: settings.bookmarks,      // Preserve local bookmarks
      showDevtools: settings.showDevtools // Preserve dev settings
    });
  }
}, [userSettingsAPI.isSuccess, userSettingsAPI.data]);
```

## Tenant Management

### Tenant Context

The layout provides tenant context to all child pages:

```javascript
const { currentTenant } = useSettings();

// Tenant validation for single-tenant pages
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
```

### Tenant Selector Integration

The tenant selector is embedded in the top navigation:

```javascript
// In TopNav component
<TenantSelector
  currentTenant={settings.currentTenant}
  onTenantChange={settings.handleUpdate}
  available={tenantOptions}
/>
```

## Alert and Notification System

### Global Alert Handling

```javascript
const alertsAPI = ApiGetCall({
  url: `/api/GetCippAlerts?localversion=${version?.data?.version}`,
  queryKey: "alertsDashboard",
  waiting: false,
  refetchOnMount: false,
  refetchOnReconnect: false,
  keepPreviousData: true,
});

useEffect(() => {
  if (alertsAPI.isSuccess && !alertsAPI.isFetching) {
    if (alertsAPI.data.length > 0) {
      alertsAPI.data.forEach((alert) => {
        dispatch(showToast({
          message: alert.Alert,
          title: alert.title,
          toastError: alert,
        }));
      });
    }
  }
}, [alertsAPI.isSuccess]);
```

### Setup Wizard Integration

```javascript
const [setupCompleted, setSetupCompleted] = useState(true);
const createDialog = useDialog();

// Check for incomplete setup
useEffect(() => {
  if (alertsAPI.isSuccess && !alertsAPI.isFetching) {
    const setupCompleted = alertsAPI.data.find(alert => alert.setupCompleted === false);
    if (setupCompleted) {
      setSetupCompleted(false);
    }
  }
}, [alertsAPI.isSuccess]);

// Setup wizard dialog
<Dialog
  fullWidth
  maxWidth="lg"
  onClose={createDialog.handleClose}
  open={createDialog.open}
>
  <DialogTitle>Setup Wizard</DialogTitle>
  <DialogContent>
    <Page /> {/* Onboarding wizard */}
  </DialogContent>
</Dialog>
```

## Performance Optimization

### Navigation Memoization

```javascript
// Memoize menu filtering to prevent unnecessary re-calculations
const filteredMenu = useMemo(() => {
  return filterItemsByRole(nativeMenuItems);
}, [
  currentRole.data?.clientPrincipal?.userRoles,
  currentRole.data?.permissions
]);
```

### Conditional Rendering

```javascript
// Only render navigation when authentication is complete
{hideSidebar === false && (
  <>
    <TopNav onNavOpen={mobileNav.handleOpen} openNav={mobileNav.open} />
    {mdDown && (
      <MobileNav items={menuItems} onClose={mobileNav.handleClose} open={mobileNav.open} />
    )}
    {!mdDown && <SideNav items={menuItems} onPin={handleNavPin} pinned={!!settings.pinNav} />}
  </>
)}
```

### Lazy Loading

Heavy components can be lazy loaded:

```javascript
import dynamic from "next/dynamic";

const LazySetupWizard = dynamic(
  () => import("../pages/onboardingv2"),
  { ssr: false }
);
```

## Error Handling

### Authentication Errors

```javascript
// Handle authentication failures
if (swaStatus.isError || currentRole.isError) {
  return <UnauthenticatedLayout />;
}

// Handle missing permissions
if (currentRole.isSuccess && !currentRole.data?.permissions) {
  return <UnauthorizedLayout />;
}
```

### Network Error Recovery

```javascript
// Retry mechanism for critical API calls
const retryAuthCall = useCallback(() => {
  swaStatus.refetch();
  currentRole.refetch();
}, [swaStatus, currentRole]);

// Error boundary for layout failures
<ErrorBoundary
  fallback={<LayoutErrorFallback onRetry={retryAuthCall} />}
  onError={(error) => console.error("Layout error:", error)}
>
  {children}
</ErrorBoundary>
```

## Customization Examples

### Custom Footer

```javascript
// Add custom footer to layout
<LayoutContainer>
  {children}
  <Footer />
  <CustomFooter additionalInfo="Custom content" />
</LayoutContainer>
```

### Navigation Customization

```javascript
// Custom navigation handler
const handleNavPin = useCallback(() => {
  settings.handleUpdate({
    pinNav: !settings.pinNav,
  });
  
  // Custom logic
  analytics.track('navigation_pinned', {
    pinned: !settings.pinNav
  });
}, [settings]);
```

### Theme Integration

```javascript
// Custom theme handling in layout
const customTheme = useMemo(() => {
  return createTheme({
    ...baseTheme,
    palette: {
      ...baseTheme.palette,
      primary: settings.customPrimaryColor || baseTheme.palette.primary
    }
  });
}, [settings.customPrimaryColor]);

<ThemeProvider theme={customTheme}>
  <LayoutRoot>
    {children}
  </LayoutRoot>
</ThemeProvider>
```

## Accessibility Features

### Keyboard Navigation

```javascript
// Skip link for keyboard users
<Button
  sx={{ 
    position: 'absolute', 
    left: '-9999px',
    '&:focus': { left: '6px', top: '6px' }
  }}
  onClick={() => document.getElementById('main-content')?.focus()}
>
  Skip to main content
</Button>

// Main content area
<Box id="main-content" tabIndex={-1}>
  {children}
</Box>
```

### Screen Reader Support

```javascript
// Proper ARIA landmarks
<Box component="nav" role="navigation" aria-label="Main navigation">
  <SideNav items={menuItems} />
</Box>

<Box component="main" role="main" aria-label="Main content">
  {children}
</Box>
```

### Focus Management

```javascript
// Focus management on navigation changes
useEffect(() => {
  if (pathname) {
    // Announce route changes to screen readers
    const announcement = `Navigated to ${document.title}`;
    announceToScreenReader(announcement);
  }
}, [pathname]);
```

## Development Tools Integration

### Debug Information

```javascript
{process.env.NODE_ENV === 'development' && settings.showDevtools && (
  <Box sx={{ position: 'fixed', bottom: 0, right: 0, p: 2 }}>
    <Card>
      <CardContent>
        <Typography variant="caption">
          Tenant: {settings.currentTenant}<br/>
          User: {currentRole.data?.clientPrincipal?.userDetails}<br/>
          Permissions: {currentRole.data?.permissions?.length || 0}
        </Typography>
      </CardContent>
    </Card>
  </Box>
)}
```

### Performance Monitoring

```javascript
// Performance timing for layout rendering
useEffect(() => {
  if (process.env.NODE_ENV === 'development') {
    const startTime = performance.now();
    
    return () => {
      const endTime = performance.now();
      console.log(`Layout render time: ${endTime - startTime}ms`);
    };
  }
}, []);
```

## Common Use Cases

### Public Pages

```javascript
// Layout for unauthenticated pages
const PublicPage = () => {
  return <div>Public content</div>;
};

PublicPage.getLayout = (page) => (
  <PublicLayout>
    {page}
  </PublicLayout>
);
```

### Admin-Only Pages

```javascript
// Layout with admin requirement
const AdminPage = () => {
  return <div>Admin content</div>;
};

AdminPage.getLayout = (page) => (
  <DashboardLayout>
    <RequirePermissions permissions={["Admin.Access"]}>
      {page}
    </RequirePermissions>
  </DashboardLayout>
);
```

### Modal Pages

```javascript
// Layout for modal-style pages
const ModalPage = () => {
  return <div>Modal content</div>;
};

ModalPage.getLayout = (page) => (
  <DashboardLayout>
    <ModalContainer>
      {page}
    </ModalContainer>
  </DashboardLayout>
);
```

## Best Practices

1. **Consistent Usage**: Always use the `getLayout` pattern
2. **Permission Checking**: Implement proper access controls at layout level
3. **Performance**: Optimize expensive operations with memoization
4. **Error Handling**: Provide fallbacks for authentication and network errors
5. **Accessibility**: Ensure proper navigation structure and ARIA labels
6. **Mobile Support**: Test responsive behavior across all breakpoints
7. **State Management**: Use layout for global state that affects navigation
8. **Loading States**: Provide clear feedback during authentication and data loading
9. **SEO**: Use proper page titles and meta information
10. **Analytics**: Track navigation and user interactions appropriately

## Troubleshooting

### Common Issues

**Layout Not Rendering**
- Check if `getLayout` is properly exported with the page
- Verify layout component import path
- Ensure authentication API endpoints are accessible

**Navigation Not Showing**
- Check user authentication status in browser dev tools
- Verify user has required permissions for menu items
- Check menu configuration in `/src/layouts/config.js`

**Mobile Navigation Problems**
- Test responsive breakpoints with browser dev tools
- Check touch event handling on mobile devices
- Verify mobile navigation state management

**Performance Issues**
- Monitor navigation re-renders with React DevTools
- Check for expensive permission calculations
- Optimize API calls with proper caching

**Authentication Loops**
- Check Static Web Apps authentication configuration
- Verify API endpoint accessibility
- Review authentication token expiration handling

## Related Components

- [HeaderedTabbedLayout](./tabbed-layouts.md) - Enhanced tabbed layout
- [TabbedLayout](./tabbed-layouts.md) - Simple tabbed layout
- [Navigation Components](../components/README.md) - SideNav, TopNav, MobileNav
- [Settings Context](../getting-started/README.md) - Global settings management