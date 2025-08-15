# Dashboard Pages - Custom Layout Pattern

Dashboard pages provide overview and summary information using card-based layouts. Unlike other page types that use standardized components, dashboard pages use custom layouts with various card components to create rich, interactive information displays.

## Core Concept

Dashboard pages are custom-built using:
- **Grid-based layouts** for responsive design
- **Card components** for information display
- **Interactive elements** like charts, buttons, and search
- **Real-time data** from multiple API sources

## Basic Structure

```javascript
import Head from "next/head";
import { Box, Container } from "@mui/material";
import { Grid } from "@mui/system";
import { Layout as DashboardLayout } from "../layouts/index.js";

const Page = () => {
  return (
    <>
      <Head>
        <title>Dashboard</title>
      </Head>
      <Box sx={{ flexGrow: 1, py: 4 }}>
        <Container maxWidth={false}>
          <Grid container spacing={3}>
            {/* Dashboard cards go here */}
          </Grid>
        </Container>
      </Box>
    </>
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default Page;
```

## Key Dashboard Components

### 1. CippInfoBar
Displays key information in a horizontal bar format.

```javascript
import { CippInfoBar } from "../components/CippCards/CippInfoBar";

const tenantInfo = [
  { name: "Tenant Name", data: organization.data?.displayName },
  { name: "Tenant ID", data: organization.data?.id },
  { name: "Default Domain", data: defaultDomain }
];

<Grid size={{ md: 12, xs: 12 }}>
  <CippInfoBar 
    data={tenantInfo} 
    isFetching={organization.isFetching} 
  />
</Grid>
```

### 2. CippChartCard
Interactive charts for data visualization.

```javascript
import { CippChartCard } from "../components/CippCards/CippChartCard";

<Grid size={{ md: 4, xs: 12 }}>
  <CippChartCard
    title="User Statistics"
    isFetching={dashboard.isFetching}
    chartType="pie"
    chartSeries={[
      Number(dashboard.data?.LicUsers || 0),
      Number(dashboard.data?.UnlicUsers || 0),
      Number(dashboard.data?.Guests || 0)
    ]}
    labels={["Licensed Users", "Unlicensed Users", "Guests"]}
  />
</Grid>
```

### 3. CippPropertyListCard
Displays properties in a list format with copy functionality.

```javascript
import { CippPropertyListCard } from "../components/CippCards/CippPropertyListCard";

<Grid size={{ md: 4, xs: 12 }}>
  <CippPropertyListCard
    title="Domain Names"
    showDivider={false}
    copyItems={true}
    isFetching={organization.isFetching}
    propertyItems={organization.data?.verifiedDomains?.map(domain => ({
      label: "",
      value: domain.name
    }))}
  />
</Grid>
```

### 4. CippUniversalSearch
Global search functionality for the dashboard.

```javascript
import { CippUniversalSearch } from "../components/CippCards/CippUniversalSearch";

<Box sx={{ flex: 1 }}>
  <CippUniversalSearch />
</Box>
```

## Real-World Example: Main Dashboard

Here's the actual main dashboard implementation from `/src/pages/index.js`:

```javascript
import Head from "next/head";
import { useEffect, useState } from "react";
import { Box, Container, Button, Card, CardContent, Tooltip } from "@mui/material";
import { Grid } from "@mui/system";
import { CippInfoBar } from "../components/CippCards/CippInfoBar";
import { CippChartCard } from "../components/CippCards/CippChartCard";
import { CippPropertyListCard } from "../components/CippCards/CippPropertyListCard";
import { Layout as DashboardLayout } from "../layouts/index.js";
import { useSettings } from "../hooks/use-settings";
import { ApiGetCall } from "../api/ApiCall.jsx";
import { ExecutiveReportButton } from "../components/ExecutiveReportButton.js";
import { CippUniversalSearch } from "../components/CippCards/CippUniversalSearch.jsx";

const Page = () => {
  const { currentTenant } = useSettings();
  const [domainVisible, setDomainVisible] = useState(false);

  // API calls for dashboard data
  const organization = ApiGetCall({
    url: "/api/ListOrg",
    queryKey: `${currentTenant}-ListOrg`,
    data: { tenantFilter: currentTenant },
  });

  const dashboard = ApiGetCall({
    url: "/api/ListuserCounts",
    data: { tenantFilter: currentTenant },
    queryKey: `${currentTenant}-ListuserCounts`,
  });

  const sharepoint = ApiGetCall({
    url: "/api/ListSharepointQuota",
    queryKey: `${currentTenant}-ListSharepointQuota`,
    data: { tenantFilter: currentTenant },
  });

  // Data processing for tenant info bar
  const tenantInfo = [
    { name: "Tenant Name", data: organization.data?.displayName },
    { name: "Tenant ID", data: organization.data?.id },
    { name: "Default Domain", data: defaultDomain },
    { name: "AD Sync Enabled", data: organization.data?.onPremisesSyncEnabled }
  ];

  return (
    <>
      <Head>
        <title>Dashboard</title>
      </Head>
      <Box sx={{ flexGrow: 1, py: 4 }}>
        <Container maxWidth={false}>
          <Grid container spacing={3}>
            {/* Header with actions and search */}
            <Grid size={{ md: 12, xs: 12 }}>
              <Card>
                <CardContent sx={{ display: "flex", alignItems: "center", gap: 2, p: 2 }}>
                  <ExecutiveReportButton
                    tenantName={organization.data?.displayName}
                    tenantId={organization.data?.id}
                    disabled={organization.isFetching}
                  />
                  <Box sx={{ flex: 1 }}>
                    <CippUniversalSearch />
                  </Box>
                </CardContent>
              </Card>
            </Grid>

            {/* Tenant information bar */}
            <Grid size={{ md: 12, xs: 12 }}>
              <CippInfoBar data={tenantInfo} isFetching={organization.isFetching} />
            </Grid>

            {/* User statistics chart */}
            <Grid size={{ md: 4, xs: 12 }}>
              <CippChartCard
                title="User Statistics"
                isFetching={dashboard.isFetching}
                chartType="pie"
                chartSeries={[
                  Number(dashboard.data?.LicUsers || 0),
                  Number(dashboard.data?.UnlicUsers || 0),
                  Number(dashboard.data?.Guests || 0)
                ]}
                labels={["Licensed Users", "Unlicensed Users", "Guests"]}
              />
            </Grid>

            {/* SharePoint quota chart */}
            <Grid size={{ md: 4, xs: 12 }}>
              <CippChartCard
                title="SharePoint Quota"
                isFetching={sharepoint.isFetching}
                chartType="donut"
                chartSeries={[
                  Number(sharepoint.data?.TenantStorageMB - sharepoint.data?.GeoUsedStorageMB) || 0,
                  Number(sharepoint.data?.GeoUsedStorageMB) || 0,
                ]}
                labels={[
                  `Free (${formatStorageSize(sharepoint.data?.TenantStorageMB - sharepoint.data?.GeoUsedStorageMB)})`,
                  `Used (${formatStorageSize(sharepoint.data?.GeoUsedStorageMB)})`
                ]}
              />
            </Grid>

            {/* Domain names list */}
            <Grid size={{ md: 4, xs: 12 }}>
              <CippPropertyListCard
                title="Domain Names"
                showDivider={false}
                copyItems={true}
                isFetching={organization.isFetching}
                propertyItems={organization.data?.verifiedDomains
                  ?.slice(0, domainVisible ? undefined : 3)
                  .map(domain => ({
                    label: "",
                    value: domain.name
                  }))}
                actionButton={
                  organization.data?.verifiedDomains?.length > 3 && (
                    <Button onClick={() => setDomainVisible(!domainVisible)}>
                      {domainVisible ? "See less" : "See more..."}
                    </Button>
                  )
                }
              />
            </Grid>

            {/* Partner relationships */}
            <Grid size={{ md: 4, xs: 12 }}>
              <CippPropertyListCard
                showDivider={false}
                copyItems={true}
                title="Partner Relationships"
                isFetching={partners.isFetching}
                propertyItems={partners.data?.Results.map(partner => ({
                  label: partner.TenantInfo?.displayName,
                  value: partner.TenantInfo?.defaultDomainName
                }))}
              />
            </Grid>

            {/* Tenant capabilities */}
            <Grid size={{ md: 4, xs: 12 }}>
              <CippPropertyListCard
                copyItems={true}
                showDivider={false}
                title="Tenant Capabilities"
                isFetching={organization.isFetching}
                propertyItems={[
                  {
                    label: "Services",
                    value: organization.data?.assignedPlans
                      ?.filter(plan => plan.capabilityStatus === "Enabled")
                      .map(plan => plan.service)
                      .join(", ")
                  }
                ]}
              />
            </Grid>
          </Grid>
        </Container>
      </Box>
    </>
  );
};

Page.getLayout = (page) => <DashboardLayout allTenantsSupport={false}>{page}</DashboardLayout>;
export default Page;
```

## Chart Types and Configuration

### Pie Charts
Perfect for showing proportional data.

```javascript
<CippChartCard
  title="License Distribution"
  chartType="pie"
  chartSeries={[30, 20, 15, 10]}
  labels={["Office 365 E3", "Office 365 E5", "Business Premium", "Business Basic"]}
  isFetching={loading}
/>
```

### Bar Charts
Good for comparing quantities across categories.

```javascript
<CippChartCard
  title="Monthly Signins"
  chartType="bar" 
  chartSeries={[120, 150, 98, 200, 180]}
  labels={["Jan", "Feb", "Mar", "Apr", "May"]}
  isFetching={loading}
/>
```

### Donut Charts
Similar to pie charts but with a hollow center.

```javascript
<CippChartCard
  title="Storage Usage"
  chartType="donut"
  chartSeries={[75, 25]}
  labels={["Used (750GB)", "Free (250GB)"]}
  isFetching={loading}
/>
```

### Line Charts
Best for showing trends over time.

```javascript
<CippChartCard
  title="User Growth"
  chartType="line"
  chartSeries={[
    {
      name: "Total Users",
      data: [100, 120, 140, 160, 180, 200]
    }
  ]}
  labels={["Jan", "Feb", "Mar", "Apr", "May", "Jun"]}
  isFetching={loading}
/>
```

## Interactive Elements

### Clickable Charts

```javascript
<Tooltip title="Click to view details">
  <CippChartCard
    title="Security Alerts"
    chartType="bar"
    chartSeries={[5, 12, 8, 3]}
    labels={["Critical", "High", "Medium", "Low"]}
    onClick={() => router.push("/security/alerts")}
    sx={{ cursor: "pointer" }}
  />
</Tooltip>
```

### Action Buttons

```javascript
import { BulkActionsMenu } from "../components/bulk-actions-menu.js";

<BulkActionsMenu
  buttonName="Quick Actions"
  actions={[
    { label: "User Management", link: "/identity/administration/users" },
    { label: "License Management", link: "/tenant/administration/licenses" },
    { label: "Security Reports", link: "/security/reports" }
  ]}
/>
```

### Executive Reports

```javascript
import { ExecutiveReportButton } from "../components/ExecutiveReportButton.js";

<ExecutiveReportButton
  tenantName={organization.data?.displayName}
  tenantId={organization.data?.id}
  userStats={{
    licensedUsers: dashboard.data?.LicUsers || 0,
    unlicensedUsers: dashboard.data?.UnlicUsers || 0,
    guests: dashboard.data?.Guests || 0
  }}
  organizationData={organization.data}
  disabled={organization.isFetching}
/>
```

## Data Management Patterns

### Multiple API Calls

```javascript
const Page = () => {
  const { currentTenant } = useSettings();

  // Multiple concurrent API calls
  const userStats = ApiGetCall({
    url: "/api/ListUserCounts",
    data: { tenantFilter: currentTenant },
    queryKey: `${currentTenant}-UserCounts`
  });

  const licenseInfo = ApiGetCall({
    url: "/api/ListLicenses", 
    data: { tenantFilter: currentTenant },
    queryKey: `${currentTenant}-Licenses`
  });

  const securityScore = ApiGetCall({
    url: "/api/GetSecureScore",
    data: { tenantFilter: currentTenant },
    queryKey: `${currentTenant}-SecureScore`
  });

  // Combine data for display
  const combinedData = useMemo(() => {
    if (!userStats.data || !licenseInfo.data) return null;
    
    return {
      totalUsers: userStats.data.total,
      availableLicenses: licenseInfo.data.available,
      securityScore: securityScore.data?.score || 0
    };
  }, [userStats.data, licenseInfo.data, securityScore.data]);

  return (
    // Dashboard layout using combinedData
  );
};
```

### Real-time Updates

```javascript
const Page = () => {
  const [lastUpdate, setLastUpdate] = useState(new Date());

  // Auto-refresh every 5 minutes
  useEffect(() => {
    const interval = setInterval(() => {
      setLastUpdate(new Date());
      // Trigger refetch of critical data
      queryClient.invalidateQueries(['dashboard-data']);
    }, 5 * 60 * 1000);

    return () => clearInterval(interval);
  }, []);

  return (
    <Box>
      <Typography variant="caption" color="text.secondary">
        Last updated: {lastUpdate.toLocaleTimeString()}
      </Typography>
      {/* Dashboard content */}
    </Box>
  );
};
```

### Error Handling

```javascript
const ErrorHandledCard = ({ apiCall, title, children }) => {
  if (apiCall.isError) {
    return (
      <Card>
        <CardContent>
          <Typography variant="h6" color="error">
            {title} - Error Loading Data
          </Typography>
          <Typography variant="body2" color="text.secondary">
            {apiCall.error?.message || "Unknown error occurred"}
          </Typography>
        </CardContent>
      </Card>
    );
  }

  return children;
};

// Usage
<ErrorHandledCard apiCall={userStats} title="User Statistics">
  <CippChartCard
    title="User Statistics"
    data={userStats.data}
    isFetching={userStats.isFetching}
  />
</ErrorHandledCard>
```

## Responsive Design

### Grid Breakpoints

```javascript
<Grid container spacing={3}>
  {/* Full width on mobile, half on tablet, third on desktop */}
  <Grid size={{ xs: 12, md: 6, lg: 4 }}>
    <CippChartCard title="Chart 1" />
  </Grid>
  
  {/* Full width on mobile/tablet, half on desktop */}
  <Grid size={{ xs: 12, lg: 6 }}>
    <CippPropertyListCard title="Properties" />
  </Grid>
  
  {/* Always full width */}
  <Grid size={12}>
    <CippInfoBar data={infoData} />
  </Grid>
</Grid>
```

### Conditional Rendering

```javascript
import { useMediaQuery } from "@mui/material";

const Page = () => {
  const isMobile = useMediaQuery((theme) => theme.breakpoints.down('md'));

  return (
    <Grid container spacing={3}>
      {!isMobile && (
        <Grid size={4}>
          <DetailedChart />
        </Grid>
      )}
      
      <Grid size={{ xs: 12, md: isMobile ? 12 : 8 }}>
        <MainContent />
      </Grid>
    </Grid>
  );
};
```

## Performance Optimization

### Lazy Loading

```javascript
import dynamic from "next/dynamic";

const LazyChart = dynamic(
  () => import("../components/CippCards/CippChartCard"),
  { 
    ssr: false,
    loading: () => <Skeleton variant="rectangular" height={300} />
  }
);
```

### Memoization

```javascript
const Page = () => {
  const processedData = useMemo(() => {
    if (!rawData) return null;
    
    return rawData.map(item => ({
      ...item,
      calculatedValue: expensiveCalculation(item)
    }));
  }, [rawData]);

  const chartSeries = useMemo(() => [
    processedData?.filter(item => item.type === 'A').length || 0,
    processedData?.filter(item => item.type === 'B').length || 0
  ], [processedData]);

  return (
    <CippChartCard
      title="Data Distribution"
      chartSeries={chartSeries}
      labels={["Type A", "Type B"]}
    />
  );
};
```

### Pagination for Large Datasets

```javascript
const Page = () => {
  const [page, setPage] = useState(1);
  const pageSize = 10;

  const data = ApiGetCall({
    url: "/api/GetPagedData",
    data: { 
      page,
      pageSize,
      tenantFilter: currentTenant 
    },
    queryKey: `PagedData-${currentTenant}-${page}`
  });

  return (
    <Box>
      <CippPropertyListCard
        title="Recent Items"
        propertyItems={data.data?.items}
        actionButton={
          <Button onClick={() => setPage(page + 1)}>
            Load More
          </Button>
        }
      />
    </Box>
  );
};
```

## Common Dashboard Patterns

### Executive Dashboard

```javascript
const ExecutiveDashboard = () => {
  return (
    <Grid container spacing={3}>
      {/* Key metrics */}
      <Grid size={12}>
        <CippInfoBar data={keyMetrics} />
      </Grid>
      
      {/* Charts row */}
      <Grid size={4}>
        <CippChartCard title="User Growth" chartType="line" />
      </Grid>
      <Grid size={4}>
        <CippChartCard title="License Usage" chartType="donut" />
      </Grid>
      <Grid size={4}>
        <CippChartCard title="Security Score" chartType="bar" />
      </Grid>
      
      {/* Details row */}
      <Grid size={6}>
        <CippPropertyListCard title="Top Issues" />
      </Grid>
      <Grid size={6}>
        <CippPropertyListCard title="Recent Changes" />
      </Grid>
    </Grid>
  );
};
```

### Operational Dashboard

```javascript
const OperationalDashboard = () => {
  return (
    <Grid container spacing={3}>
      {/* Alert summary */}
      <Grid size={12}>
        <AlertSummaryCard />
      </Grid>
      
      {/* System health */}
      <Grid size={6}>
        <CippChartCard title="System Health" chartType="pie" />
      </Grid>
      <Grid size={6}>
        <CippChartCard title="Response Times" chartType="line" />
      </Grid>
      
      {/* Activity logs */}
      <Grid size={12}>
        <RecentActivityTable />
      </Grid>
    </Grid>
  );
};
```

### Tenant Overview Dashboard

```javascript
const TenantOverviewDashboard = () => {
  return (
    <Grid container spacing={3}>
      {/* Tenant info bar */}
      <Grid size={12}>
        <CippInfoBar data={tenantInfo} />
      </Grid>
      
      {/* User and license metrics */}
      <Grid size={4}>
        <CippChartCard title="Users" chartType="pie" />
      </Grid>
      <Grid size={4}>
        <CippChartCard title="Licenses" chartType="bar" />
      </Grid>
      <Grid size={4}>
        <CippChartCard title="Security" chartType="donut" />
      </Grid>
      
      {/* Configuration details */}
      <Grid size={6}>
        <CippPropertyListCard title="Domains" />
      </Grid>
      <Grid size={6}>
        <CippPropertyListCard title="Applications" />
      </Grid>
    </Grid>
  );
};
```

## Best Practices

1. **Responsive Design**: Use proper grid breakpoints for all screen sizes
2. **Loading States**: Show skeleton loaders during data fetching
3. **Error Handling**: Provide meaningful error messages and recovery options
4. **Performance**: Optimize API calls and use memoization for expensive operations
5. **User Experience**: Keep dashboards focused and avoid information overload
6. **Accessibility**: Ensure charts and interactive elements are accessible
7. **Data Freshness**: Implement appropriate refresh strategies
8. **Visual Hierarchy**: Use consistent spacing and typography
9. **Color Coding**: Use meaningful colors for status and category indication
10. **Interactive Elements**: Provide clear affordances for clickable items

## Troubleshooting

### Common Issues

**Charts Not Displaying**
- Verify chart data format matches expected structure
- Check for null/undefined data values
- Ensure chart series and labels arrays have matching lengths

**Performance Issues**
- Implement proper memoization for calculated values
- Use lazy loading for heavy components
- Optimize API calls to fetch only necessary data

**Responsive Layout Problems**
- Check grid breakpoint configurations
- Test on various screen sizes
- Use browser dev tools to debug layout issues

**Data Not Updating**
- Verify React Query cache configuration
- Check API call dependencies and triggers
- Ensure proper queryKey patterns for cache invalidation

## Related Components

- [`CippInfoBar`](../components/cipp-cards/README.md) - Information bar component
- [`CippChartCard`](../components/cipp-cards/README.md) - Chart display component
- [`CippPropertyListCard`](../components/cipp-cards/README.md) - Property list component
- [`CippUniversalSearch`](../components/cipp-cards/README.md) - Global search component
- [`ExecutiveReportButton`](../components/README.md) - Report generation component