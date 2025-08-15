# Layout System Overview

CIPP uses a hierarchical layout system that provides consistent navigation, authentication, and structure across all pages. The layout system is built on Next.js layout patterns with Material-UI components for responsive design.

## Layout Hierarchy

```
DashboardLayout (Main)
├── HeaderedTabbedLayout
├── TabbedLayout
└── Custom Page Layouts
```

## Core Layout Components

### 1. DashboardLayout (Main Layout)
The foundation layout that wraps all pages in the application.

**Location**: `/src/layouts/index.js`

**Features**:
- Side navigation with collapsible/pinned states
- Top navigation with tenant selector
- Authentication and user management
- Permission-based menu filtering
- Mobile-responsive navigation
- Loading states and error handling

### 2. HeaderedTabbedLayout
Enhanced tabbed layout with header, breadcrumbs, and actions.

**Location**: `/src/layouts/HeaderedTabbedLayout.jsx`

**Features**:
- Page title and subtitle display
- Breadcrumb navigation
- Tab-based content organization
- Action buttons in header
- Back navigation
- Mobile-optimized tabs

### 3. TabbedLayout
Simple tabbed interface for basic multi-section pages.

**Location**: `/src/layouts/TabbedLayout.jsx`

**Features**:
- Tab navigation only
- Minimal overhead
- Content area management
- Route-based tab switching

## Layout Pattern Usage

### Basic Page Layout

Every page in CIPP uses the `getLayout` pattern:

```javascript
import { Layout as DashboardLayout } from "/src/layouts/index.js";

const Page = () => {
  return (
    <div>
      {/* Page content */}
    </div>
  );
};

// This is the key pattern - every page needs this
Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;

export default Page;
```

### Tabbed Page Layout

For pages with multiple sections or views:

```javascript
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { HeaderedTabbedLayout } from "/src/layouts/HeaderedTabbedLayout.jsx";
import tabOptions from "./tabOptions.json";

const Page = () => {
  return (
    <HeaderedTabbedLayout
      tabOptions={tabOptions}
      title="Page Title"
      subtitle={[
        { icon: <Icon />, text: "Subtitle text" }
      ]}
      actions={[
        { label: "Action", handler: () => {} }
      ]}
    >
      {/* Tab content */}
    </HeaderedTabbedLayout>
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default Page;
```

## Authentication & Permissions

### Authentication Flow

The main layout handles authentication:

1. **Authentication Check**: Validates user session via `/.auth/me`
2. **Role Retrieval**: Gets user roles and permissions via `/api/me`
3. **Menu Filtering**: Filters navigation based on permissions
4. **Access Control**: Shows/hides UI elements based on roles

### Permission-Based Navigation

Navigation items are filtered by user permissions:

```javascript
// In layout configuration
const menuItem = {
  title: "Users",
  path: "/identity/administration/users",
  permissions: ["Identity.User.Read"], // Required permissions
  roles: ["Administrator"],            // Required roles (optional)
  items: [...] // Sub-items (also filtered)
};
```

### Permission Pattern Matching

Supports wildcard patterns for flexible permission checking:

```javascript
// User has permission: "Identity.User.ReadWrite"
// Required permission: "Identity.User.*"
// Result: Access granted (wildcard match)

// User has permission: "Identity.User.Read"  
// Required permission: "Identity.User.ReadWrite"
// Result: Access denied (exact match required)
```

## Responsive Design

### Breakpoint System

The layout uses Material-UI breakpoints:

- **xs**: 0px - 599px (Mobile)
- **sm**: 600px - 899px (Small tablet)
- **md**: 900px - 1199px (Tablet/Small desktop)
- **lg**: 1200px - 1535px (Desktop)
- **xl**: 1536px+ (Large desktop)

### Navigation Adaptation

```javascript
// Desktop: Side navigation (collapsible)
{!mdDown && (
  <SideNav 
    items={menuItems} 
    onPin={handleNavPin} 
    pinned={!!settings.pinNav} 
  />
)}

// Mobile: Drawer navigation
{mdDown && (
  <MobileNav 
    items={menuItems} 
    onClose={mobileNav.handleClose} 
    open={mobileNav.open} 
  />
)}
```

### Layout Responsiveness

```javascript
// Layout root adapts to navigation state
const LayoutRoot = styled("div")(({ theme }) => ({
  backgroundColor: theme.palette.background.default,
  display: "flex",
  flex: "1 1 auto",
  maxWidth: "100%",
  paddingTop: TOP_NAV_HEIGHT,
  [theme.breakpoints.up("lg")]: {
    paddingLeft: SIDE_NAV_WIDTH, // Desktop offset
  },
}));
```

## Navigation Configuration

### Menu Structure

Navigation is defined in `/src/layouts/config.js`:

```javascript
export const nativeMenuItems = [
  {
    title: "Dashboard",
    path: "/",
    icon: <DashboardIcon />,
    permissions: ["Dashboard.Read"]
  },
  {
    title: "Identity",
    icon: <IdentityIcon />,
    permissions: ["Identity.*"],
    items: [
      {
        title: "Administration",
        items: [
          {
            title: "Users",
            path: "/identity/administration/users",
            permissions: ["Identity.User.Read"]
          },
          {
            title: "Groups", 
            path: "/identity/administration/groups",
            permissions: ["Identity.Group.Read"]
          }
        ]
      }
    ]
  }
];
```

### Tab Configuration

For tabbed layouts, tabs are defined in `tabOptions.json` files:

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

## State Management

### Settings Context

The layout integrates with the global settings context:

```javascript
const settings = useSettings();

// Available settings
settings.currentTenant    // Selected tenant
settings.pinNav          // Navigation pin state
settings.theme          // Theme preferences
settings.bookmarks      // User bookmarks
```

### Tenant Management

Tenant selection is managed in the layout:

```javascript
// Tenant selector in top navigation
<TenantSelector
  currentTenant={settings.currentTenant}
  onTenantChange={settings.handleUpdate}
  tenants={availableTenants}
/>
```

## Layout Customization

### Custom Layouts

Create custom layouts for specific needs:

```javascript
// /src/layouts/CustomLayout.jsx
export const CustomLayout = ({ children, customProp }) => {
  return (
    <Box sx={{ display: "flex", flexDirection: "column" }}>
      <CustomHeader customProp={customProp} />
      <Box component="main" sx={{ flexGrow: 1 }}>
        {children}
      </Box>
      <CustomFooter />
    </Box>
  );
};

// Usage in page
Page.getLayout = (page) => (
  <DashboardLayout>
    <CustomLayout customProp="value">
      {page}
    </CustomLayout>
  </DashboardLayout>
);
```

### Layout Props

Layouts can accept props for customization:

```javascript
// Layout with custom props
Page.getLayout = (page) => (
  <DashboardLayout 
    allTenantsSupport={false}  // Disable all-tenants mode
    hideNavigation={false}     // Show/hide navigation
  >
    {page}
  </DashboardLayout>
);
```

### Conditional Layouts

Apply different layouts based on conditions:

```javascript
const getPageLayout = (page) => {
  // Check if user is authenticated
  if (!isAuthenticated) {
    return <AuthLayout>{page}</AuthLayout>;
  }
  
  // Check if setup is complete
  if (!setupComplete) {
    return <OnboardingLayout>{page}</OnboardingLayout>;
  }
  
  // Default authenticated layout
  return <DashboardLayout>{page}</DashboardLayout>;
};

Page.getLayout = getPageLayout;
```

## Performance Considerations

### Layout Optimization

```javascript
// Memoize navigation items to prevent re-renders
const memoizedMenuItems = useMemo(() => {
  return filterItemsByPermissions(nativeMenuItems, userPermissions);
}, [userPermissions]);

// Lazy load heavy navigation components
const LazyNavigationPanel = dynamic(
  () => import('./NavigationPanel'),
  { ssr: false }
);
```

### Bundle Splitting

Layouts are automatically code-split by Next.js:

```javascript
// Each layout is in its own chunk
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { HeaderedTabbedLayout } from "/src/layouts/HeaderedTabbedLayout.jsx";
```

## Error Boundaries

### Layout Error Handling

```javascript
const LayoutErrorBoundary = ({ children }) => {
  return (
    <ErrorBoundary
      fallback={
        <Box p={4}>
          <Typography variant="h4">Layout Error</Typography>
          <Typography>
            There was an error loading the layout. Please refresh the page.
          </Typography>
        </Box>
      }
    >
      {children}
    </ErrorBoundary>
  );
};

// Wrap layouts in error boundaries
Page.getLayout = (page) => (
  <LayoutErrorBoundary>
    <DashboardLayout>{page}</DashboardLayout>
  </LayoutErrorBoundary>
);
```

## Accessibility

### Keyboard Navigation

```javascript
// Layout supports keyboard navigation
const handleKeyDown = (event) => {
  if (event.key === 'Tab' && event.ctrlKey) {
    // Navigate between main sections
    focusNextSection();
  }
};

// ARIA landmarks
<Box component="nav" role="navigation" aria-label="Main navigation">
  <SideNav />
</Box>
```

### Screen Reader Support

```javascript
// Proper heading hierarchy
<Typography variant="h1" sx={{ sr: { only: true } }}>
  CIPP Application
</Typography>

// Skip links
<Button
  sx={{ position: 'absolute', left: '-9999px' }}
  onFocus={(e) => e.target.style.left = '0'}
>
  Skip to main content
</Button>
```

## Common Patterns

### Loading States

```javascript
// Layout shows loading while authentication resolves
{authStatus.isLoading && <FullPageSpinner />}

// Content loads with skeleton
{isLoading ? <LayoutSkeleton /> : children}
```

### Error States

```javascript
// Authentication error
if (authStatus.isError) {
  return <UnauthenticatedLayout />;
}

// Permission error  
if (!hasRequiredPermissions) {
  return <UnauthorizedLayout />;
}
```

### Theme Integration

```javascript
// Layout responds to theme changes
const theme = useTheme();
const isDark = theme.palette.mode === 'dark';

<Box
  sx={{
    backgroundColor: isDark ? 'grey.900' : 'grey.50',
    transition: 'background-color 0.3s'
  }}
>
  {children}
</Box>
```

## Best Practices

1. **Consistent Structure**: Always use the `getLayout` pattern
2. **Permission Integration**: Implement proper access controls
3. **Responsive Design**: Test layouts across all breakpoints
4. **Performance**: Optimize navigation rendering and state updates
5. **Accessibility**: Ensure proper heading hierarchy and keyboard navigation
6. **Error Handling**: Implement error boundaries for layout failures
7. **Loading States**: Provide clear loading feedback
8. **Theme Support**: Make layouts work with all theme variants
9. **Mobile First**: Design layouts for mobile and enhance for desktop
10. **SEO Consideration**: Use proper meta tags and structured markup

## Troubleshooting

### Common Issues

**Layout Not Applying**
- Verify `getLayout` function is exported with the page
- Check that layout component is properly imported
- Ensure layout wrapper is returning the page component

**Navigation Not Showing**
- Check user authentication status
- Verify user has required permissions
- Check menu configuration and permission matching

**Responsive Issues**
- Test with browser dev tools at various screen sizes
- Verify breakpoint usage in styled components
- Check Grid component size props

**Performance Problems**
- Implement proper memoization for expensive calculations
- Check for unnecessary re-renders in navigation
- Optimize permission checking logic

## Related Documentation

- [Main Layout](./main-layout.md) - Detailed DashboardLayout documentation
- [Tabbed Layouts](./tabbed-layouts.md) - HeaderedTabbedLayout and TabbedLayout
- [Page Types](../page-types/) - How layouts work with different page patterns
- [Routing](../routing.md) - Next.js routing integration with layouts