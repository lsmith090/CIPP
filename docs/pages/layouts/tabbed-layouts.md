# Tabbed Layouts

CIPP provides two tabbed layout components for organizing content into multiple sections or views. These layouts enable clean navigation between related content areas while maintaining context and state.

## Layout Components

### 1. HeaderedTabbedLayout
A full-featured tabbed layout with header, breadcrumbs, actions, and navigation.

**Location**: `/src/layouts/HeaderedTabbedLayout.jsx`

### 2. TabbedLayout  
A simple tabbed interface for basic multi-section pages.

**Location**: `/src/layouts/TabbedLayout.jsx`

## HeaderedTabbedLayout

### Overview

The `HeaderedTabbedLayout` provides a complete page structure with:
- Page title and subtitle display
- Breadcrumb navigation with back button
- Action buttons in the header
- Tab-based content organization
- Mobile-responsive design
- URL-based tab routing

### Basic Usage

```javascript
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { HeaderedTabbedLayout } from "/src/layouts/HeaderedTabbedLayout.jsx";
import tabOptions from "./tabOptions.json";

const Page = () => {
  return (
    <HeaderedTabbedLayout
      tabOptions={tabOptions}
      title="User Management"
      subtitle={[
        { icon: <PersonIcon />, text: "John Doe" },
        { icon: <EmailIcon />, text: "john.doe@example.com" }
      ]}
      actions={[
        { label: "Edit", handler: () => handleEdit() },
        { label: "Delete", handler: () => handleDelete() }
      ]}
    >
      {/* Tab content goes here */}
    </HeaderedTabbedLayout>
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default Page;
```

### Props

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `tabOptions` | `array` | Yes | Array of tab configuration objects |
| `title` | `string` | Yes | Main page title |
| `children` | `ReactNode` | Yes | Content to render in tab area |
| `subtitle` | `array` | No | Array of subtitle items with icons |
| `actions` | `array` | No | Array of action button configurations |
| `actionsData` | `any` | No | Data passed to action handlers |
| `isFetching` | `boolean` | No | Loading state for title/subtitle |
| `backUrl` | `string` | No | Custom back navigation URL |

### Tab Configuration

Tab options are typically defined in a `tabOptions.json` file:

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
  },
  {
    "label": "Conditional Access",
    "path": "/identity/administration/users/user/conditional-access"
  }
]
```

### Subtitle Configuration

Subtitles display contextual information with icons:

```javascript
const subtitle = [
  {
    icon: <Mail />,
    text: <CippCopyToClipBoard type="chip" text="user@domain.com" />
  },
  {
    icon: <Fingerprint />,
    text: <CippCopyToClipBoard type="chip" text="user-id-123" />
  },
  {
    icon: <CalendarIcon />,
    text: <>Created: <CippTimeAgo data="2023-01-01T00:00:00Z" /></>
  }
];
```

### Actions Configuration

Actions provide interactive buttons in the header:

```javascript
const actions = [
  {
    label: "Edit User",
    handler: (data) => router.push(`/users/edit?id=${data.id}`),
    icon: <EditIcon />,
    color: "primary"
  },
  {
    label: "Reset Password", 
    handler: (data) => handlePasswordReset(data.id),
    icon: <KeyIcon />,
    color: "warning",
    confirmText: "Reset this user's password?"
  },
  {
    label: "Delete User",
    handler: (data) => handleDelete(data.id),
    icon: <DeleteIcon />,
    color: "error",
    confirmText: "Are you sure you want to delete this user?"
  }
];
```

### Real-World Example

Here's the actual implementation from `/src/pages/identity/administration/users/user/index.jsx`:

```javascript
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { HeaderedTabbedLayout } from "../../../../../layouts/HeaderedTabbedLayout";
import tabOptions from "./tabOptions";
import { CippCopyToClipBoard } from "../../../../../components/CippComponents/CippCopyToClipboard";
import { Mail, Fingerprint, Launch } from "@mui/icons-material";
import CalendarIcon from "@heroicons/react/24/outline/CalendarIcon";
import { CippTimeAgo } from "../../../../../components/CippComponents/CippTimeAgo";
import CippUserActions from "/src/components/CippComponents/CippUserActions";

const Page = () => {
  const userSettingsDefaults = useSettings();
  const router = useRouter();
  const { userId } = router.query;

  const userRequest = ApiGetCall({
    url: `/api/ListUsers?UserId=${userId}&tenantFilter=${router.query.tenantFilter ?? userSettingsDefaults.currentTenant}`,
    queryKey: `ListUsers-${userId}`,
    waiting: !!userId,
  });

  const title = userRequest.isSuccess ? 
    <>{userRequest.data?.[0]?.displayName}</> : 
    "Loading...";

  const subtitle = userRequest.isSuccess ? [
    {
      icon: <Mail />,
      text: <CippCopyToClipBoard type="chip" text={userRequest.data?.[0]?.userPrincipalName} />
    },
    {
      icon: <Fingerprint />,
      text: <CippCopyToClipBoard type="chip" text={userRequest.data?.[0]?.id} />
    },
    {
      icon: <CalendarIcon />,
      text: <>Created: <CippTimeAgo data={userRequest.data?.[0]?.createdDateTime} /></>
    },
    {
      icon: <Launch style={{ color: "#667085" }} />,
      text: (
        <Button
          color="muted"
          size="small"
          href={`https://entra.microsoft.com/${userSettingsDefaults.currentTenant}/#view/Microsoft_AAD_UsersAndTenants/UserProfileMenuBlade/~/overview/userId/${userId}`}
          target="_blank"
        >
          View in Entra
        </Button>
      )
    }
  ] : [];

  return (
    <HeaderedTabbedLayout
      tabOptions={tabOptions}
      title={title}
      actions={CippUserActions()}
      actionsData={userRequest.data?.[0]}
      subtitle={subtitle}
      isFetching={userRequest.isLoading}
    >
      {/* Tab content rendered here */}
      {userRequest.isLoading && <CippFormSkeleton layout={[2, 1, 2, 2]} />}
      {userRequest.isSuccess && (
        <Box sx={{ flexGrow: 1, py: 4 }}>
          {/* User details content */}
        </Box>
      )}
    </HeaderedTabbedLayout>
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default Page;
```

## TabbedLayout

### Overview

The `TabbedLayout` provides a simpler tabbed interface with:
- Tab navigation only
- Minimal visual overhead
- Route-based tab switching
- Mobile-responsive tabs

### Basic Usage

```javascript
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { TabbedLayout } from "/src/layouts/TabbedLayout.jsx";

const tabOptions = [
  { label: "Overview", path: "/section/overview" },
  { label: "Details", path: "/section/details" },
  { label: "Settings", path: "/section/settings" }
];

const Page = () => {
  return (
    <TabbedLayout tabOptions={tabOptions}>
      {/* Tab content */}
    </TabbedLayout>
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default Page;
```

### Props

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `tabOptions` | `array` | Yes | Array of tab configuration objects |
| `children` | `ReactNode` | Yes | Content to render in tab area |

## Tab Configuration Patterns

### File-Based Tab Options

Tab configurations are typically stored in `tabOptions.json` files alongside the pages:

```
/pages/section/
  ├── tabOptions.json          # Tab configuration
  ├── index.js                 # Default tab page
  ├── overview.js              # Tab 1 content
  ├── details.js               # Tab 2 content
  └── settings.js              # Tab 3 content
```

### Dynamic Tab Options

For dynamic tab configurations:

```javascript
const Page = () => {
  const { data: userRoles } = ApiGetCall({
    url: "/api/GetUserRoles",
    queryKey: "userRoles"
  });

  const tabOptions = useMemo(() => {
    const baseTabs = [
      { label: "Overview", path: "/user/overview" },
      { label: "Profile", path: "/user/profile" }
    ];

    // Add admin-only tabs
    if (userRoles?.includes("Admin")) {
      baseTabs.push(
        { label: "Permissions", path: "/user/permissions" },
        { label: "Audit Log", path: "/user/audit" }
      );
    }

    return baseTabs;
  }, [userRoles]);

  return (
    <HeaderedTabbedLayout tabOptions={tabOptions}>
      {children}
    </HeaderedTabbedLayout>
  );
};
```

### Conditional Tab Display

```javascript
const tabOptions = [
  { label: "Basic Info", path: "/entity/basic" },
  { 
    label: "Advanced", 
    path: "/entity/advanced",
    condition: (user) => user.hasAdvancedAccess 
  },
  {
    label: "Admin Panel",
    path: "/entity/admin", 
    condition: (user) => user.roles.includes("Admin")
  }
];

// Filter tabs based on conditions
const filteredTabs = tabOptions.filter(tab => 
  !tab.condition || tab.condition(currentUser)
);
```

## Navigation Integration

### URL-Based Tab Routing

Both tabbed layouts use Next.js routing for tab navigation:

```javascript
// Current tab determined by pathname
const currentTab = tabOptions.find(option => option.path === pathname);

// Tab change navigates to new route
const handleTabsChange = useCallback((event, value) => {
  router.push({
    pathname: value,
    query: queryParams,  // Preserve query parameters
  }, undefined, { shallow: true });
}, [router, queryParams]);
```

### Query Parameter Preservation

Query parameters are maintained during tab navigation:

```javascript
// URL: /users/user?userId=123&tenantFilter=domain.com
// Tab navigation preserves: userId=123&tenantFilter=domain.com

router.push({
  pathname: "/users/user/edit",
  query: router.query,  // Preserves existing query params
});
```

### Deep Linking

Tabs support direct URL access:

```javascript
// Direct navigation to specific tab
router.push("/users/user/exchange?userId=123");

// Tab automatically becomes active based on pathname
```

## Responsive Design

### Mobile Optimization

```javascript
// Mobile-responsive tab variants
<Tabs 
  onChange={handleTabsChange} 
  value={currentTab?.path} 
  variant="scrollable"  // Allows horizontal scrolling on mobile
  scrollButtons="auto"  // Shows scroll buttons when needed
>
  {tabOptions.map((option) => (
    <Tab key={option.path} label={option.label} value={option.path} />
  ))}
</Tabs>
```

### Responsive Breakpoints

```javascript
const mdDown = useMediaQuery((theme) => theme.breakpoints.down("md"));

// Adjust layout for mobile
<Box
  sx={!mdDown && {
    flexGrow: 1,
    overflow: "auto",
    height: "calc(100vh - 400px)",  // Fixed height on desktop
  }}
>
  {children}
</Box>
```

## State Management

### Tab State Persistence

Tab state is automatically managed through URL routing:

```javascript
// No manual state management needed
// Current tab is derived from URL pathname
const currentTab = tabOptions.find(option => option.path === pathname);
```

### Cross-Tab Data Sharing

Data can be shared across tabs using:

#### URL Query Parameters
```javascript
// Share data via query params
router.push({
  pathname: "/users/user/edit",
  query: {
    ...router.query,
    mode: "advanced",
    returnTo: pathname
  }
});
```

#### React Query Cache
```javascript
// Share data via React Query cache
const sharedData = ApiGetCall({
  url: "/api/GetSharedData",
  queryKey: "sharedData",  // Same key across tabs
});
```

#### Context Providers
```javascript
// Share data via React Context
const TabDataProvider = ({ children }) => {
  const [sharedState, setSharedState] = useState({});
  
  return (
    <TabDataContext.Provider value={{ sharedState, setSharedState }}>
      {children}
    </TabDataContext.Provider>
  );
};
```

## Loading and Error States

### Loading States

```javascript
const Page = () => {
  const dataLoading = userData.isLoading;

  return (
    <HeaderedTabbedLayout
      title={dataLoading ? "Loading..." : userData.data?.name}
      isFetching={dataLoading}  // Shows skeleton loaders
      subtitle={dataLoading ? [] : subtitleData}
    >
      {dataLoading ? (
        <CippFormSkeleton layout={[2, 1, 2]} />
      ) : (
        <TabContent data={userData.data} />
      )}
    </HeaderedTabbedLayout>
  );
};
```

### Error States

```javascript
const Page = () => {
  if (userData.isError) {
    return (
      <HeaderedTabbedLayout
        tabOptions={tabOptions}
        title="Error Loading Data"
      >
        <Alert severity="error">
          Failed to load user data: {userData.error?.message}
        </Alert>
      </HeaderedTabbedLayout>
    );
  }

  return (
    <HeaderedTabbedLayout tabOptions={tabOptions}>
      {/* Normal content */}
    </HeaderedTabbedLayout>
  );
};
```

## Accessibility

### Keyboard Navigation

```javascript
// Tabs automatically support keyboard navigation
// Arrow keys navigate between tabs
// Enter/Space activate tabs

// Custom keyboard shortcuts
useEffect(() => {
  const handleKeyDown = (event) => {
    if (event.ctrlKey && event.key >= '1' && event.key <= '9') {
      const tabIndex = parseInt(event.key) - 1;
      if (tabOptions[tabIndex]) {
        router.push(tabOptions[tabIndex].path);
      }
    }
  };

  document.addEventListener('keydown', handleKeyDown);
  return () => document.removeEventListener('keydown', handleKeyDown);
}, [tabOptions, router]);
```

### Screen Reader Support

```javascript
// Proper ARIA attributes for tabs
<Tabs
  onChange={handleTabsChange}
  value={currentTab?.path}
  aria-label="User management sections"
  role="tablist"
>
  {tabOptions.map((option) => (
    <Tab
      key={option.path}
      label={option.label}
      value={option.path}
      aria-controls={`tabpanel-${option.path}`}
      id={`tab-${option.path}`}
    />
  ))}
</Tabs>

<Box
  role="tabpanel"
  id={`tabpanel-${currentTab?.path}`}
  aria-labelledby={`tab-${currentTab?.path}`}
>
  {children}
</Box>
```

## Performance Optimization

### Lazy Tab Loading

```javascript
import dynamic from "next/dynamic";

// Lazy load heavy tab components
const LazyAdvancedTab = dynamic(
  () => import("./AdvancedTab"),
  { 
    ssr: false,
    loading: () => <Skeleton variant="rectangular" height={400} />
  }
);

const Page = () => {
  const currentPath = usePathname();
  
  return (
    <HeaderedTabbedLayout tabOptions={tabOptions}>
      {currentPath === "/advanced" && <LazyAdvancedTab />}
      {currentPath === "/basic" && <BasicTab />}
    </HeaderedTabbedLayout>
  );
};
```

### Memoization

```javascript
// Memoize expensive tab calculations
const processedTabData = useMemo(() => {
  return rawData?.map(item => ({
    ...item,
    calculatedValue: expensiveCalculation(item)
  }));
}, [rawData]);

// Memoize tab options
const memoizedTabOptions = useMemo(() => {
  return filterTabsByPermissions(baseTabOptions, userPermissions);
}, [baseTabOptions, userPermissions]);
```

## Common Use Cases

### Entity Detail Pages

```javascript
// User detail page with multiple views
const tabOptions = [
  { label: "Overview", path: "/users/user" },
  { label: "Profile", path: "/users/user/edit" },
  { label: "Licenses", path: "/users/user/licenses" },
  { label: "Groups", path: "/users/user/groups" },
  { label: "Devices", path: "/users/user/devices" }
];
```

### Settings Pages

```javascript
// Application settings with categories
const tabOptions = [
  { label: "General", path: "/settings" },
  { label: "Security", path: "/settings/security" },
  { label: "Integrations", path: "/settings/integrations" },
  { label: "Notifications", path: "/settings/notifications" }
];
```

### Administrative Sections

```javascript
// Admin section with multiple management areas
const tabOptions = [
  { label: "Users", path: "/admin/users" },
  { label: "Roles", path: "/admin/roles" },
  { label: "Permissions", path: "/admin/permissions" },
  { label: "Audit Logs", path: "/admin/audit" }
];
```

## Best Practices

1. **Consistent Navigation**: Use standard tab patterns across similar pages
2. **Logical Grouping**: Group related functionality in tabs
3. **Clear Labels**: Use descriptive, concise tab labels
4. **Loading States**: Provide feedback during data loading
5. **Error Handling**: Handle errors gracefully with meaningful messages
6. **Accessibility**: Ensure keyboard navigation and screen reader support
7. **Performance**: Lazy load heavy tab content when possible
8. **Mobile Design**: Test tab usability on mobile devices
9. **State Management**: Use appropriate patterns for cross-tab data sharing
10. **URL Structure**: Create logical, bookmarkable URLs for tabs

## Troubleshooting

### Common Issues

**Tabs Not Switching**
- Check that `tabOptions` paths match actual route paths
- Verify router navigation is working correctly
- Check for JavaScript errors in console

**Content Not Loading**
- Verify data fetching is triggered for each tab
- Check API endpoints and query keys
- Ensure proper loading state handling

**Mobile Navigation Issues**
- Test tab scrolling on small screens
- Check responsive breakpoint handling
- Verify touch interactions work correctly

**Performance Problems**
- Implement lazy loading for heavy components
- Check for unnecessary re-renders
- Optimize data fetching strategies

## Related Components

- [DashboardLayout](./main-layout.md) - Main application layout
- [Page Types](../page-types/) - How tabbed layouts work with different page patterns
- [Navigation Components](../components/README.md) - Tab and navigation components