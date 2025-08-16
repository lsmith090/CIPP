# Creating New Pages in CIPP

This guide covers the complete workflow for adding new pages to the CIPP application, including file structure, layout selection, routing, and navigation integration.

## Page Creation Workflow

### 1. File and Directory Structure

CIPP uses Next.js file-based routing. Pages are organized under `/src/pages/` following a logical hierarchy:

```
/src/pages/
├── identity/
│   ├── administration/
│   │   ├── users/
│   │   │   ├── index.js          # List users page
│   │   │   ├── add.jsx           # Add user form
│   │   │   └── user/             # User detail pages
│   │   │       ├── index.jsx     # View user
│   │   │       ├── edit.jsx      # Edit user
│   │   │       └── tabOptions.json # Tab configuration
│   │   └── groups/
│   │       ├── index.js          # List groups
│   │       └── add.jsx           # Add group
│   └── reports/
│       └── mfa-report/
│           └── index.js          # MFA report page
├── tenant/
│   └── administration/
│       └── tenants/
│           ├── index.js          # List tenants
│           └── add.js            # Add tenant
└── email/
    └── administration/
        └── mailboxes/
            └── index.js          # List mailboxes
```

### 2. File Naming Conventions

| File Type | Naming | Example | Purpose |
|-----------|--------|---------|----------|
| List pages | `index.js` | `users/index.js` | Main listing/table page |
| Form pages | `add.jsx` | `users/add.jsx` | Create new item form |
| Edit pages | `edit.jsx` | `users/edit.jsx` | Edit existing item form |
| Detail pages | `index.jsx` | `user/index.jsx` | View item details |
| Tab config | `tabOptions.json` | `user/tabOptions.json` | Tab navigation config |

### 3. Basic Page Template

Create a new page using this template:

```javascript
// /src/pages/example/new-feature/index.js
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { CippTablePage } from "/src/components/CippComponents/CippTablePage.jsx";
import { useSettings } from "/src/hooks/use-settings.js";
import { ApiGetCall } from "/src/api/ApiCall.jsx";

const Page = () => {
  const pageTitle = "New Feature";
  const { currentTenant } = useSettings();

  // For table pages, use CippTablePage component
  return (
    <CippTablePage
      title={pageTitle}
      apiUrl="/api/ListNewFeature"
      apiData={{ tenantFilter: currentTenant }}
      queryKey="newFeature"
      columns={[
        { field: "displayName", headerName: "Display Name", flex: 1 },
        { field: "status", headerName: "Status", flex: 1 },
        { field: "lastModified", headerName: "Last Modified", flex: 1 }
      ]}
    />
  );
};

// Required: Set the layout
Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;

export default Page;
```

### 4. Custom Page Template

For pages that need custom layout and functionality:

```javascript
// /src/pages/example/custom-page/index.js
import { useState } from "react";
import {
  Box,
  Container,
  Card,
  CardContent,
  Typography,
  Button,
  Stack
} from "@mui/material";
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { CippHead } from "/src/components/CippComponents/CippHead.jsx";
import { ApiGetCall, ApiPostCall } from "/src/api/ApiCall.jsx";
import { useSettings } from "/src/hooks/use-settings.js";

const Page = () => {
  const pageTitle = "Custom Feature";
  const { currentTenant } = useSettings();
  const [selectedItem, setSelectedItem] = useState(null);

  // Fetch data
  const { data, isLoading, isError } = ApiGetCall({
    url: "/api/GetCustomData",
    queryKey: "customData",
    data: { tenantFilter: currentTenant },
    waiting: !!currentTenant
  });

  // Mutation for actions
  const updateMutation = ApiPostCall({
    relatedQueryKeys: ["customData"],
    onResult: (result) => {
      console.log("Update completed:", result);
    }
  });

  const handleAction = () => {
    updateMutation.mutate({
      url: "/api/UpdateCustomData",
      data: { tenantFilter: currentTenant, itemId: selectedItem }
    });
  };

  if (isLoading) return <div>Loading...</div>;
  if (isError) return <div>Error loading data</div>;

  return (
    <>
      <CippHead title={pageTitle} />
      <Box sx={{ flexGrow: 1, py: 4 }}>
        <Container maxWidth={false}>
          <Stack spacing={3}>
            <Typography variant="h4">{pageTitle}</Typography>
            
            <Card>
              <CardContent>
                <Typography gutterBottom>
                  Custom page content goes here
                </Typography>
                
                <Button
                  variant="contained"
                  onClick={handleAction}
                  disabled={updateMutation.isPending}
                >
                  {updateMutation.isPending ? "Processing..." : "Take Action"}
                </Button>
              </CardContent>
            </Card>
          </Stack>
        </Container>
      </Box>
    </>
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;

export default Page;
```

## Layout Selection

CIPP provides several layout options depending on your page needs:

### 1. DashboardLayout (Standard)

For most pages, use the standard dashboard layout:

```javascript
import { Layout as DashboardLayout } from "/src/layouts/index.js";

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
```

**Use for:**
- List/table pages
- Form pages
- Most content pages
- Single-purpose pages

### 2. TabbedLayout

For pages that need simple tab navigation:

```javascript
import { TabbedLayout } from "/src/layouts/TabbedLayout.jsx";
import tabOptions from "./tabOptions.json";

Page.getLayout = (page) => (
  <DashboardLayout>
    <TabbedLayout tabOptions={tabOptions}>
      {page}
    </TabbedLayout>
  </DashboardLayout>
);
```

**Use for:**
- Related pages grouped by functionality
- Settings sections
- Configuration areas

**Tab Configuration (`tabOptions.json`):**
```json
[
  {
    "label": "General",
    "path": "/example/settings"
  },
  {
    "label": "Advanced",
    "path": "/example/settings/advanced"
  },
  {
    "label": "Permissions",
    "path": "/example/settings/permissions"
  }
]
```

### 3. HeaderedTabbedLayout

For detail pages with header information and tab navigation:

```javascript
import { HeaderedTabbedLayout } from "/src/layouts/HeaderedTabbedLayout.jsx";
import tabOptions from "./tabOptions.json";
import { useRouter } from "next/router";
import { ApiGetCall } from "/src/api/ApiCall.jsx";

const Page = () => {
  const router = useRouter();
  const { userId } = router.query;

  const { data: user, isLoading } = ApiGetCall({
    url: "/api/GetUser",
    queryKey: ["user", userId],
    data: { userId },
    waiting: !!userId
  });

  return (
    <HeaderedTabbedLayout
      title={user?.displayName || "Loading..."}
      subtitle={[
        { icon: <EmailIcon />, text: user?.userPrincipalName },
        { icon: <BuildingIcon />, text: user?.department }
      ]}
      tabOptions={tabOptions}
      isFetching={isLoading}
      backUrl="/identity/administration/users"
      actions={[
        {
          label: "Edit User",
          handler: () => router.push(`/identity/administration/users/user/edit?userId=${userId}`)
        },
        {
          label: "Reset Password",
          handler: () => {/* Reset password logic */}
        }
      ]}
      actionsData={user}
    >
      {/* Page content */}
    </HeaderedTabbedLayout>
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
```

**Use for:**
- Detail pages (user, tenant, application details)
- Multi-tab entity views
- Pages with contextual actions

## Navigation Integration

### 1. Adding to Navigation Menu

Navigation is configured in `/src/layouts/config.js`. Add your new page to the appropriate section:

```javascript
// In /src/layouts/config.js
{
  title: "Example Section",
  items: [
    {
      title: "New Feature",
      path: "/example/new-feature",
      icon: <CubeIcon />,
      permissions: ["Example.Read"], // Optional permissions
    },
    {
      title: "Custom Page",
      path: "/example/custom-page",
      icon: <DocumentIcon />,
      permissions: ["Example.Admin"],
    }
  ]
}
```

### 2. Permission Integration

Control page access using permissions:

```javascript
// In the page component
import { usePermissions } from "/src/hooks/use-permissions.js";

const Page = () => {
  const { checkPermissions } = usePermissions();
  
  if (!checkPermissions(["Example.Read"])) {
    return <div>Access denied</div>;
  }

  // Page content...
};
```

### 3. Breadcrumb Integration

Breadcrumbs are automatically generated from the URL path. Ensure your page paths follow the logical hierarchy:

```
/identity/administration/users        → Identity > Administration > Users
/tenant/administration/applications   → Tenant > Administration > Applications
```

## Common Page Patterns

### 1. Table/List Pages

Use `CippTablePage` for data listing pages:

```javascript
const Page = () => {
  const { currentTenant } = useSettings();
  
  return (
    <CippTablePage
      title="Users"
      apiUrl="/api/ListUsers"
      apiData={{ tenantFilter: currentTenant }}
      queryKey="users"
      columns={[
        { field: "displayName", headerName: "Display Name", flex: 1 },
        { field: "userPrincipalName", headerName: "UPN", flex: 1 },
        { field: "jobTitle", headerName: "Job Title", flex: 1 }
      ]}
      filters={[
        {
          filterName: "Account Enabled",
          value: [{ id: "accountEnabled", value: "Yes" }],
          type: "column"
        }
      ]}
      offCanvas={{
        extendedInfoFields: ["id", "createdDateTime", "lastModified"],
        actions: UserActions()
      }}
    />
  );
};
```

### 2. Form Pages

Create dedicated form pages for data entry:

```javascript
import { CippFormPage } from "/src/components/CippFormPages/CippFormPage.jsx";

const Page = () => {
  const router = useRouter();
  const { currentTenant } = useSettings();

  const formSchema = {
    displayName: { type: "text", label: "Display Name", required: true },
    email: { type: "email", label: "Email Address", required: true },
    department: { type: "text", label: "Department" }
  };

  const handleSubmit = (data) => {
    // Submit logic
    mutation.mutate({
      url: "/api/CreateUser",
      data: { ...data, tenantFilter: currentTenant }
    });
  };

  return (
    <CippFormPage
      title="Add New User"
      schema={formSchema}
      onSubmit={handleSubmit}
      backUrl="/identity/administration/users"
    />
  );
};
```

### 3. Dashboard Pages

For dashboard-style layouts with cards and widgets:

```javascript
import { Grid } from "@mui/system";
import { CippInfoCard } from "/src/components/CippCards/CippInfoCard.jsx";

const Page = () => {
  const { data } = ApiGetCall({
    url: "/api/GetDashboardStats",
    queryKey: "dashboardStats"
  });

  return (
    <>
      <CippHead title="Dashboard" />
      <Box sx={{ flexGrow: 1, py: 4 }}>
        <Container maxWidth={false}>
          <Grid container spacing={3}>
            <Grid size={{ xs: 12, md: 3 }}>
              <CippInfoCard
                label="Total Users"
                value={data?.userCount || 0}
                icon={<UsersIcon />}
              />
            </Grid>
            <Grid size={{ xs: 12, md: 3 }}>
              <CippInfoCard
                label="Active Licenses"
                value={data?.licenseCount || 0}
                icon={<LicenseIcon />}
              />
            </Grid>
            {/* More cards... */}
          </Grid>
        </Container>
      </Box>
    </>
  );
};
```

### 4. Wizard Pages

For multi-step processes:

```javascript
import { CippWizard } from "/src/components/CippWizard/CippWizard.jsx";

const Page = () => {
  const steps = [
    {
      title: "Basic Information",
      component: BasicInfoStep
    },
    {
      title: "Configuration",
      component: ConfigurationStep
    },
    {
      title: "Review",
      component: ReviewStep
    }
  ];

  return (
    <CippWizard
      title="Setup Wizard"
      steps={steps}
      onComplete={(data) => {
        // Handle completion
      }}
    />
  );
};
```

## Best Practices

### 1. Page Structure
- Use consistent naming conventions
- Follow the established directory hierarchy
- Include proper TypeScript/PropTypes definitions
- Add appropriate error boundaries

### 2. Data Loading
- Use appropriate loading states
- Handle error states gracefully
- Implement proper caching strategies
- Consider pagination for large datasets

### 3. User Experience
- Provide clear page titles and breadcrumbs
- Show loading indicators for async operations
- Include helpful error messages
- Ensure responsive design

### 4. Performance
- Lazy load heavy components
- Optimize API calls
- Use proper memoization
- Implement efficient re-renders

### 5. Accessibility
- Use semantic HTML elements
- Provide proper ARIA labels
- Ensure keyboard navigation
- Maintain color contrast standards

### 6. Testing
- Test different screen sizes
- Verify permission-based access
- Test API error scenarios
- Validate form inputs

## Common Anti-Patterns to Avoid

1. **Hard-coded tenant filters**: Always use `currentTenant` from settings
2. **Missing error handling**: Always handle API errors appropriately
3. **Inconsistent layouts**: Use the established layout patterns
4. **Poor navigation**: Ensure proper back/breadcrumb navigation
5. **Missing permissions**: Always check user permissions for sensitive operations
6. **Inefficient API calls**: Avoid unnecessary or redundant API requests
7. **Poor mobile experience**: Test on different screen sizes

## File Checklist

When creating a new page, ensure you have:

- [ ] Created the page file in the correct directory
- [ ] Set the appropriate layout using `getLayout`
- [ ] Added navigation menu entries (if needed)
- [ ] Created `tabOptions.json` for tabbed layouts
- [ ] Implemented proper permission checks
- [ ] Added appropriate API integration
- [ ] Tested on different screen sizes
- [ ] Verified error handling
- [ ] Updated any related documentation

This guide provides the foundation for creating consistent, functional pages in CIPP. For specific page types, refer to existing examples in the codebase and the component documentation.