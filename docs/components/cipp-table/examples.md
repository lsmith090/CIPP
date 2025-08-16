# CippTable Examples

Real-world examples of using CIPP table components for different data management scenarios.

## Basic User Management Table

Simple user table with common actions:

```jsx
import { CippTablePage } from '/src/components/CippComponents/CippTablePage.jsx';
import { Chip, Avatar, Stack, Typography } from '@mui/material';
import { UserIcon, KeyIcon, XMarkIcon, EditIcon } from '@heroicons/react/24/outline';

const UsersTable = () => {
  const userColumns = [
    {
      accessorKey: 'displayName',
      header: 'Display Name',
      cell: ({ getValue, row }) => (
        <Stack direction="row" alignItems="center" spacing={2}>
          <Avatar sx={{ width: 32, height: 32 }}>
            {getValue()?.charAt(0)}
          </Avatar>
          <div>
            <Typography variant="body2" fontWeight="medium">
              {getValue()}
            </Typography>
            <Typography variant="caption" color="text.secondary">
              {row.original.jobTitle}
            </Typography>
          </div>
        </Stack>
      ),
    },
    {
      accessorKey: 'mail',
      header: 'Email',
      enableClickToCopy: true,
    },
    {
      accessorKey: 'department',
      header: 'Department',
      filterVariant: 'select',
    },
    {
      accessorKey: 'accountEnabled',
      header: 'Status',
      cell: ({ getValue }) => (
        <Chip
          label={getValue() ? 'Enabled' : 'Disabled'}
          color={getValue() ? 'success' : 'error'}
          size="small"
        />
      ),
      filterVariant: 'checkbox',
    },
    {
      accessorKey: 'lastSignInDateTime',
      header: 'Last Sign In',
      cell: ({ getValue }) => 
        getValue() ? new Date(getValue()).toLocaleDateString() : 'Never',
      sortingFn: 'dateTimeNullsLast',
    },
  ];

  const userActions = [
    {
      label: 'Edit User',
      icon: <EditIcon />,
      customFunction: (row) => {
        router.push(`/identity/administration/users/edit?userId=${row.id}`);
      },
      noConfirm: true,
    },
    {
      label: 'Reset Password',
      icon: <KeyIcon />,
      api: {
        url: '/api/ExecResetPass',
        method: 'POST',
        data: { ID: '{id}' },
      },
      fields: [
        {
          name: 'mustChangePass',
          label: 'User must change password at next logon',
          type: 'checkbox',
          defaultValue: true,
        },
        {
          name: 'sendEmail',
          label: 'Send password via email',
          type: 'checkbox',
          defaultValue: false,
        }
      ],
    },
    {
      label: 'Enable Account',
      icon: <UserIcon />,
      api: {
        url: '/api/ExecDisableUser',
        method: 'POST',
        data: { ID: '{id}', AccountEnabled: true },
      },
      condition: (row) => !row.accountEnabled,
      color: 'success',
    },
    {
      label: 'Disable Account',
      icon: <XMarkIcon />,
      api: {
        url: '/api/ExecDisableUser',
        method: 'POST',
        data: { ID: '{id}', AccountEnabled: false },
      },
      condition: (row) => row.accountEnabled,
      color: 'error',
    },
  ];

  return (
    <CippTablePage
      title="User Management"
      apiUrl="/api/ListUsers"
      apiData={{ $top: 999 }}
      actions={userActions}
      columns={userColumns}
      queryKey="users"
      offCanvas={{
        extendedInfoFields: [
          'id', 'userPrincipalName', 'displayName', 'mail', 
          'jobTitle', 'department', 'manager', 'accountEnabled',
          'createdDateTime', 'lastPasswordChangeDateTime'
        ]
      }}
      cardButton={
        <Button
          variant="contained"
          startIcon={<PlusIcon />}
          onClick={() => router.push('/identity/administration/users/add')}
        >
          Add User
        </Button>
      }
    />
  );
};
```

## Advanced License Management Table

Complex table with nested data and custom rendering:

```jsx
import { CippDataTable } from '/src/components/CippTable/CippDataTable.jsx';
import { useState } from 'react';

const LicenseManagementTable = () => {
  const [selectedUsers, setSelectedUsers] = useState([]);
  
  const licenseColumns = [
    {
      accessorKey: 'displayName',
      header: 'User',
      size: 200,
    },
    {
      accessorKey: 'userPrincipalName',
      header: 'UPN',
      enableClickToCopy: true,
    },
    {
      accessorKey: 'assignedLicenses',
      header: 'Assigned Licenses',
      cell: ({ getValue }) => {
        const licenses = getValue() || [];
        return (
          <Stack direction="row" spacing={1} flexWrap="wrap">
            {licenses.map((license, index) => (
              <Chip
                key={index}
                label={license.skuPartNumber}
                size="small"
                variant="outlined"
                sx={{ fontSize: '0.75rem' }}
              />
            ))}
          </Stack>
        );
      },
      enableColumnFilter: false,
      enableSorting: false,
    },
    {
      accessorKey: 'licenseCount',
      header: 'License Count',
      cell: ({ row }) => {
        const count = row.original.assignedLicenses?.length || 0;
        return (
          <Chip
            label={count}
            color={count > 0 ? 'primary' : 'default'}
            size="small"
          />
        );
      },
      filterVariant: 'range',
    },
    {
      accessorKey: 'lastSignInDateTime',
      header: 'Last Activity',
      cell: ({ getValue }) => {
        const date = getValue();
        if (!date) return <Chip label="Never" color="error" size="small" />;
        
        const daysSince = Math.floor((new Date() - new Date(date)) / (1000 * 60 * 60 * 24));
        const color = daysSince > 30 ? 'error' : daysSince > 7 ? 'warning' : 'success';
        
        return (
          <Chip
            label={`${daysSince} days ago`}
            color={color}
            size="small"
          />
        );
      },
    },
  ];

  const licenseActions = [
    {
      label: 'Assign License',
      icon: <PlusIcon />,
      customFunction: (row) => {
        setAssignDialogOpen(true);
        setSelectedUser(row);
      },
      noConfirm: true,
    },
    {
      label: 'Remove All Licenses',
      icon: <MinusIcon />,
      api: {
        url: '/api/ExecLicenseAssignment',
        method: 'POST',
        data: {
          UserID: '{id}',
          RemoveAllLicenses: true,
        },
      },
      condition: (row) => row.assignedLicenses?.length > 0,
      color: 'error',
    },
    {
      label: 'Copy Licenses From User',
      icon: <DocumentDuplicateIcon />,
      fields: [
        {
          name: 'sourceUser',
          label: 'Source User',
          type: 'user-selector',
          required: true,
        }
      ],
      api: {
        url: '/api/ExecLicenseAssignment',
        method: 'POST',
        data: {
          UserID: '{id}',
          CopyFrom: '{sourceUser}',
        },
      },
    },
  ];

  const handleBulkLicenseAssignment = () => {
    if (selectedUsers.length === 0) return;
    
    setBulkAssignDialogOpen(true);
  };

  return (
    <>
      <CippDataTable
        queryKey="license-management"
        title="License Management"
        api={{
          url: '/api/ListUsers',
          data: { 
            tenantFilter: currentTenant,
            includeLicenseInfo: true 
          },
        }}
        columns={licenseColumns}
        actions={licenseActions}
        enableRowSelection={true}
        onChange={setSelectedUsers}
        renderTopToolbarCustomActions={({ table }) => (
          <Stack direction="row" spacing={1}>
            <Button
              variant="contained"
              startIcon={<PlusIcon />}
              disabled={selectedUsers.length === 0}
              onClick={handleBulkLicenseAssignment}
            >
              Bulk Assign ({selectedUsers.length})
            </Button>
            <Button
              variant="outlined"
              startIcon={<DocumentArrowDownIcon />}
              onClick={() => exportLicenseReport(selectedUsers)}
            >
              Export Report
            </Button>
          </Stack>
        )}
        initialState={{
          columnFilters: [
            { id: 'licenseCount', value: [1, 10] }
          ],
          sorting: [
            { id: 'lastSignInDateTime', desc: true }
          ]
        }}
      />

      {/* Bulk assignment dialog */}
      <BulkLicenseAssignmentDialog
        open={bulkAssignDialogOpen}
        onClose={() => setBulkAssignDialogOpen(false)}
        users={selectedUsers}
      />
    </>
  );
};
```

## Security Alerts Table

Table with priority-based styling and real-time updates:

```jsx
import { CippDataTable } from '/src/components/CippTable/CippDataTable.jsx';
import { useEffect, useState } from 'react';

const SecurityAlertsTable = () => {
  const [realTimeUpdates, setRealTimeUpdates] = useState(true);

  const alertColumns = [
    {
      accessorKey: 'title',
      header: 'Alert',
      cell: ({ getValue, row }) => (
        <Stack>
          <Typography variant="body2" fontWeight="medium">
            {getValue()}
          </Typography>
          <Typography variant="caption" color="text.secondary">
            {row.original.description}
          </Typography>
        </Stack>
      ),
    },
    {
      accessorKey: 'severity',
      header: 'Severity',
      cell: ({ getValue }) => {
        const severity = getValue()?.toLowerCase();
        const colors = {
          critical: 'error',
          high: 'warning', 
          medium: 'info',
          low: 'success'
        };
        
        return (
          <Chip
            label={getValue()}
            color={colors[severity] || 'default'}
            size="small"
            sx={{ fontWeight: 'medium' }}
          />
        );
      },
      filterVariant: 'multi-select',
      filterSelectOptions: ['Critical', 'High', 'Medium', 'Low'],
    },
    {
      accessorKey: 'status',
      header: 'Status',
      cell: ({ getValue }) => (
        <Chip
          label={getValue()}
          color={getValue() === 'Active' ? 'error' : 'success'}
          variant={getValue() === 'Active' ? 'filled' : 'outlined'}
          size="small"
        />
      ),
      filterVariant: 'select',
    },
    {
      accessorKey: 'affectedUsers',
      header: 'Affected Users',
      cell: ({ getValue }) => (
        <Chip
          label={getValue()?.length || 0}
          color={getValue()?.length > 10 ? 'error' : 'primary'}
          size="small"
        />
      ),
    },
    {
      accessorKey: 'detectedDateTime',
      header: 'Detected',
      cell: ({ getValue }) => {
        const date = new Date(getValue());
        const now = new Date();
        const diffHours = Math.floor((now - date) / (1000 * 60 * 60));
        
        return (
          <Stack>
            <Typography variant="body2">
              {date.toLocaleDateString()}
            </Typography>
            <Typography 
              variant="caption" 
              color={diffHours < 24 ? 'error.main' : 'text.secondary'}
            >
              {diffHours < 1 ? 'Just now' : `${diffHours}h ago`}
            </Typography>
          </Stack>
        );
      },
      sortingFn: 'datetime',
    },
  ];

  const alertActions = [
    {
      label: 'Investigate',
      icon: <MagnifyingGlassIcon />,
      customFunction: (row) => {
        router.push(`/security/incidents/alert-details?alertId=${row.id}`);
      },
      noConfirm: true,
    },
    {
      label: 'Assign to Me',
      icon: <UserIcon />,
      api: {
        url: '/api/AssignAlert',
        method: 'POST',
        data: { 
          alertId: '{id}',
          assignee: currentUser.mail 
        },
      },
      condition: (row) => !row.assignedTo,
    },
    {
      label: 'Mark as Resolved',
      icon: <CheckIcon />,
      api: {
        url: '/api/ResolveAlert',
        method: 'POST',
        data: { alertId: '{id}' },
      },
      fields: [
        {
          name: 'resolution',
          label: 'Resolution Notes',
          type: 'textarea',
          required: true,
        }
      ],
      condition: (row) => row.status === 'Active',
      color: 'success',
    },
    {
      label: 'False Positive',
      icon: <XMarkIcon />,
      api: {
        url: '/api/DismissAlert',
        method: 'POST',
        data: { 
          alertId: '{id}',
          reason: 'false_positive' 
        },
      },
      condition: (row) => row.status === 'Active',
      color: 'error',
    },
  ];

  // Auto-refresh for real-time updates
  useEffect(() => {
    if (!realTimeUpdates) return;
    
    const interval = setInterval(() => {
      queryClient.invalidateQueries(['security-alerts']);
    }, 30000); // Refresh every 30 seconds

    return () => clearInterval(interval);
  }, [realTimeUpdates]);

  return (
    <CippDataTable
      queryKey="security-alerts"
      title="Security Alerts"
      api={{
        url: '/api/ListSecurityAlerts',
        data: { tenantFilter: currentTenant },
      }}
      columns={alertColumns}
      actions={alertActions}
      defaultSorting={[
        { id: 'detectedDateTime', desc: true },
        { id: 'severity', desc: true }
      ]}
      filters={[
        { id: 'status', value: 'Active' }
      ]}
      renderTopToolbarCustomActions={() => (
        <Stack direction="row" spacing={1} alignItems="center">
          <IconButton
            onClick={() => queryClient.invalidateQueries(['security-alerts'])}
            size="small"
            title="Refresh"
          >
            <ArrowPathIcon />
          </IconButton>
          <FormControlLabel
            control={
              <Switch
                checked={realTimeUpdates}
                onChange={(e) => setRealTimeUpdates(e.target.checked)}
                size="small"
              />
            }
            label="Real-time"
            sx={{ ml: 1 }}
          />
        </Stack>
      )}
      // Custom row styling based on severity
      muiTableBodyRowProps={({ row }) => ({
        sx: {
          backgroundColor: 
            row.original.severity === 'Critical' 
              ? 'error.lighter'
              : row.original.severity === 'High'
              ? 'warning.lighter'
              : 'inherit',
        }
      })}
    />
  );
};
```

## Audit Log Table

Complex table with JSON data and advanced filtering:

```jsx
import { CippDataTable } from '/src/components/CippTable/CippDataTable.jsx';
import { CippCodeBlock } from '/src/components/CippComponents/CippCodeBlock.jsx';

const AuditLogTable = () => {
  const auditColumns = [
    {
      accessorKey: 'CreationTime',
      header: 'Timestamp',
      cell: ({ getValue }) => new Date(getValue()).toLocaleString(),
      sortingFn: 'datetime',
      size: 150,
    },
    {
      accessorKey: 'UserIds',
      header: 'User',
      cell: ({ getValue }) => getValue()?.[0] || 'System',
      filterVariant: 'text',
    },
    {
      accessorKey: 'Operations',
      header: 'Operation',
      cell: ({ getValue }) => (
        <Chip
          label={getValue()?.[0]}
          size="small"
          variant="outlined"
        />
      ),
      filterVariant: 'multi-select',
    },
    {
      accessorKey: 'Workload',
      header: 'Service',
      cell: ({ getValue }) => {
        const workloadIcons = {
          'Exchange': <MailIcon />,
          'SharePoint': <FolderIcon />,
          'AzureActiveDirectory': <UserIcon />,
          'OneDrive': <CloudIcon />,
        };
        
        return (
          <Stack direction="row" alignItems="center" spacing={1}>
            {workloadIcons[getValue()]}
            <Typography variant="body2">
              {getValue()}
            </Typography>
          </Stack>
        );
      },
      filterVariant: 'select',
    },
    {
      accessorKey: 'ResultStatus',
      header: 'Result',
      cell: ({ getValue }) => (
        <Chip
          label={getValue()}
          color={getValue() === 'Success' ? 'success' : 'error'}
          size="small"
        />
      ),
    },
  ];

  const auditActions = [
    {
      label: 'View Details',
      icon: <EyeIcon />,
      customFunction: (row) => {
        setDetailDialogOpen(true);
        setSelectedAuditLog(row);
      },
      noConfirm: true,
    },
    {
      label: 'Export Event',
      icon: <DocumentArrowDownIcon />,
      customFunction: (row) => {
        const dataStr = JSON.stringify(row, null, 2);
        const dataBlob = new Blob([dataStr], { type: 'application/json' });
        const url = URL.createObjectURL(dataBlob);
        const link = document.createElement('a');
        link.href = url;
        link.download = `audit-log-${row.Id}.json`;
        link.click();
      },
      noConfirm: true,
    },
  ];

  return (
    <>
      <CippDataTable
        queryKey="audit-logs"
        title="Audit Logs"
        api={{
          url: '/api/ListAuditLog',
          data: { 
            tenantFilter: currentTenant,
            StartTime: startDate.toISOString(),
            EndTime: endDate.toISOString(),
          },
        }}
        columns={auditColumns}
        actions={auditActions}
        enableRowVirtualization={true}
        maxHeightOffset="300px"
        offCanvas={{
          children: ({ data }) => (
            <Stack spacing={2}>
              <Typography variant="h6">Audit Log Details</Typography>
              <CippCodeBlock 
                code={JSON.stringify(data, null, 2)}
                language="json"
                showLineNumbers={true}
              />
            </Stack>
          )
        }}
        renderTopToolbar={({ table }) => (
          <Box sx={{ p: 2, borderBottom: 1, borderColor: 'divider' }}>
            <Stack direction="row" spacing={2} alignItems="center">
              <DateTimePicker
                label="Start Date"
                value={startDate}
                onChange={setStartDate}
                size="small"
              />
              <DateTimePicker
                label="End Date"
                value={endDate}
                onChange={setEndDate}
                size="small"
              />
              <Button
                variant="contained"
                onClick={() => table.resetColumnFilters()}
              >
                Apply Filter
              </Button>
            </Stack>
          </Box>
        )}
        globalFilterFn="contains"
        enableGlobalFilter={true}
        initialState={{
          density: 'compact',
          columnVisibility: {
            Id: false,
            OrganizationId: false,
          }
        }}
      />

      {/* Detail dialog */}
      <Dialog
        open={detailDialogOpen}
        onClose={() => setDetailDialogOpen(false)}
        maxWidth="md"
        fullWidth
      >
        <DialogTitle>Audit Log Details</DialogTitle>
        <DialogContent>
          {selectedAuditLog && (
            <CippCodeBlock 
              code={JSON.stringify(selectedAuditLog, null, 2)}
              language="json"
              showLineNumbers={true}
            />
          )}
        </DialogContent>
      </Dialog>
    </>
  );
};
```

## Mobile-Optimized Table

Table that adapts well to mobile devices:

```jsx
import { useMediaQuery, useTheme } from '@mui/material';

const MobileOptimizedTable = () => {
  const theme = useTheme();
  const isMobile = useMediaQuery(theme.breakpoints.down('md'));

  const mobileColumns = [
    {
      accessorKey: 'displayName',
      header: 'User',
      cell: ({ getValue, row }) => (
        <Stack>
          <Typography variant="body2" fontWeight="medium">
            {getValue()}
          </Typography>
          <Typography variant="caption" color="text.secondary">
            {row.original.mail}
          </Typography>
          <Chip
            label={row.original.accountEnabled ? 'Active' : 'Disabled'}
            size="small"
            color={row.original.accountEnabled ? 'success' : 'error'}
            sx={{ width: 'fit-content', mt: 0.5 }}
          />
        </Stack>
      ),
      size: isMobile ? 300 : 200,
    },
    // Hide columns on mobile
    ...(!isMobile ? [
      {
        accessorKey: 'department',
        header: 'Department',
      },
      {
        accessorKey: 'lastSignInDateTime',
        header: 'Last Sign In',
        cell: ({ getValue }) => 
          getValue() ? new Date(getValue()).toLocaleDateString() : 'Never',
      },
    ] : []),
  ];

  return (
    <CippDataTable
      queryKey="mobile-users"
      title="Users"
      api={{
        url: '/api/ListUsers',
        data: { tenantFilter: currentTenant },
      }}
      columns={mobileColumns}
      actions={userActions}
      // Simpler layout on mobile
      initialState={{
        density: isMobile ? 'compact' : 'comfortable',
        pagination: {
          pageSize: isMobile ? 10 : 25,
        },
      }}
      // Reduce functionality on mobile
      enableColumnResizing={!isMobile}
      enableColumnOrdering={!isMobile}
      enablePinning={!isMobile}
      renderRowActions={isMobile ? undefined : ({ row, table }) => (
        <IconButton
          onClick={() => handleRowAction(row)}
          size="small"
        >
          <MoreVertIcon />
        </IconButton>
      )}
      // Use off-canvas for details on mobile
      offCanvas={isMobile ? {
        extendedInfoFields: ['mail', 'department', 'jobTitle', 'accountEnabled'],
      } : undefined}
    />
  );
};
```

## Performance-Optimized Large Dataset Table

Table optimized for handling thousands of rows:

```jsx
import { useMemo, useCallback } from 'react';

const LargeDatasetTable = () => {
  // Memoize column definitions
  const columns = useMemo(() => [
    {
      accessorKey: 'displayName',
      header: 'Name',
      size: 200,
    },
    {
      accessorKey: 'mail',
      header: 'Email',
      size: 250,
    },
    {
      accessorKey: 'department',
      header: 'Department',
      size: 150,
    },
  ], []);

  // Memoize action handlers
  const handleEdit = useCallback((row) => {
    router.push(`/users/edit/${row.id}`);
  }, [router]);

  const actions = useMemo(() => [
    {
      label: 'Edit',
      icon: <EditIcon />,
      customFunction: handleEdit,
      noConfirm: true,
    },
  ], [handleEdit]);

  return (
    <CippDataTable
      queryKey="large-dataset"
      title="All Users"
      api={{
        url: '/api/ListUsers',
        data: { 
          tenantFilter: currentTenant,
          $top: 5000 // Large dataset
        },
      }}
      columns={columns}
      actions={actions}
      // Enable virtualization
      enableRowVirtualization={true}
      enableColumnVirtualization={true}
      // Optimize for performance
      enableRowSelection={false}
      enableExpandAll={false}
      enableGlobalFilter={false}
      // Reduce initial page size
      initialState={{
        pagination: { pageSize: 50 },
        density: 'compact',
      }}
      // Custom virtualization settings
      rowVirtualizerProps={{
        estimateSize: 45,
        overscan: 20,
      }}
      columnVirtualizerProps={{
        estimateSize: 200,
        overscan: 5,
      }}
    />
  );
};
```