# CippForms Examples

Real-world examples of using CIPP form components for various data entry and management scenarios.

## User Creation Form

Complete user creation form with validation and conditional fields:

```jsx
import CippFormPage from '../CippFormPages/CippFormPage';
import { CippFormSection } from '../CippFormPages/CippFormSection';
import { CippFormComponent } from '../CippComponents/CippFormComponent';
import { CippFormTenantSelector } from '../CippComponents/CippFormTenantSelector';
import { useForm } from 'react-hook-form';

const AddUserForm = () => {
  const router = useRouter();
  const { currentTenant } = useSettings();
  
  const formControl = useForm({
    mode: 'onChange',
    defaultValues: {
      displayName: '',
      givenName: '',
      surname: '',
      mail: '',
      userPrincipalName: '',
      jobTitle: '',
      department: '',
      officeLocation: '',
      phoneNumber: '',
      manager: '',
      accountEnabled: true,
      mustChangePassword: true,
      licenses: [],
      groups: [],
    }
  });

  // Watch fields for dynamic updates
  const watchGivenName = formControl.watch('givenName');
  const watchSurname = formControl.watch('surname');
  const watchMail = formControl.watch('mail');

  // Auto-generate display name and UPN
  useEffect(() => {
    if (watchGivenName && watchSurname) {
      const displayName = `${watchGivenName} ${watchSurname}`;
      formControl.setValue('displayName', displayName);
    }
  }, [watchGivenName, watchSurname]);

  useEffect(() => {
    if (watchMail && currentTenant) {
      const username = watchMail.split('@')[0];
      const upn = `${username}@${currentTenant}`;
      formControl.setValue('userPrincipalName', upn);
    }
  }, [watchMail, currentTenant]);

  const customDataFormatter = (data) => {
    return {
      ...data,
      mailNickname: data.mail.split('@')[0],
      usageLocation: 'US', // Default usage location
      passwordProfile: {
        forceChangePasswordNextSignIn: data.mustChangePassword,
        password: generateSecurePassword(),
      },
    };
  };

  return (
    <CippFormPage
      title="User"
      formPageType="Add"
      formControl={formControl}
      postUrl="/api/AddUser"
      queryKey={['users', 'list']}
      customDataformatter={customDataFormatter}
      backButtonTitle="Back to Users"
    >
      <Stack spacing={3}>
        {/* Basic Information */}
        <CippFormSection
          title="Basic Information"
          description="Core user details and contact information"
          defaultExpanded={true}
          required={true}
        >
          <Grid container spacing={3}>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="textField"
                name="givenName"
                label="First Name"
                formControl={formControl}
                validators={{ required: "First name is required" }}
                fullWidth
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="textField"
                name="surname"
                label="Last Name"
                formControl={formControl}
                validators={{ required: "Last name is required" }}
                fullWidth
              />
            </Grid>
            <Grid item xs={12}>
              <CippFormComponent
                type="textField"
                name="displayName"
                label="Display Name"
                formControl={formControl}
                validators={{ required: "Display name is required" }}
                fullWidth
                helperText="Auto-generated from first and last name"
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="emailField"
                name="mail"
                label="Email Address"
                formControl={formControl}
                validators={{
                  required: "Email is required",
                  pattern: {
                    value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
                    message: "Invalid email format"
                  }
                }}
                fullWidth
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="textField"
                name="userPrincipalName"
                label="User Principal Name"
                formControl={formControl}
                validators={{ required: "User principal name is required" }}
                fullWidth
                helperText="Auto-generated from email"
              />
            </Grid>
          </Grid>
        </CippFormSection>

        {/* Job Information */}
        <CippFormSection
          title="Job Information"
          description="Employment and organizational details"
        >
          <Grid container spacing={3}>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="textField"
                name="jobTitle"
                label="Job Title"
                formControl={formControl}
                fullWidth
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="select"
                name="department"
                label="Department"
                formControl={formControl}
                options={[
                  { value: 'IT', label: 'Information Technology' },
                  { value: 'HR', label: 'Human Resources' },
                  { value: 'Finance', label: 'Finance' },
                  { value: 'Sales', label: 'Sales' },
                  { value: 'Marketing', label: 'Marketing' },
                  { value: 'Operations', label: 'Operations' },
                ]}
                fullWidth
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="textField"
                name="officeLocation"
                label="Office Location"
                formControl={formControl}
                fullWidth
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="phoneField"
                name="phoneNumber"
                label="Phone Number"
                formControl={formControl}
                fullWidth
              />
            </Grid>
            <Grid item xs={12}>
              <CippFormUserSelector
                name="manager"
                label="Manager"
                formControl={formControl}
                userType="users"
              />
            </Grid>
          </Grid>
        </CippFormSection>

        {/* Account Settings */}
        <CippFormSection
          title="Account Settings"
          description="Security and access configuration"
        >
          <Grid container spacing={3}>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="switch"
                name="accountEnabled"
                label="Account Enabled"
                formControl={formControl}
                helperText="Enable user account for sign-in"
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="switch"
                name="mustChangePassword"
                label="Must Change Password"
                formControl={formControl}
                helperText="Force password change on first sign-in"
              />
            </Grid>
          </Grid>
        </CippFormSection>

        {/* License Assignment */}
        <CippFormSection
          title="License Assignment"
          description="Assign licenses to the user"
        >
          <CippFormLicenseSelector
            name="licenses"
            label="Licenses"
            formControl={formControl}
            multiple={true}
            tenant={currentTenant}
          />
        </CippFormSection>

        {/* Group Membership */}
        <CippFormSection
          title="Group Membership"
          description="Add user to security and distribution groups"
        >
          <CippFormGroupSelector
            name="groups"
            label="Groups"
            formControl={formControl}
            multiple={true}
            groupTypes={['security', 'distribution', 'office365']}
          />
        </CippFormSection>
      </Stack>
    </CippFormPage>
  );
};
```

## Conditional Access Policy Form

Complex form with conditional logic and dynamic fields:

```jsx
import { useFieldArray } from 'react-hook-form';


const ConditionalAccessPolicyForm = () => {
  const formControl = useForm({
    mode: 'onChange',
    defaultValues: {
      displayName: '',
      state: 'disabled',
      conditions: {
        users: {
          includeUsers: [],
          excludeUsers: [],
        },
        applications: {
          includeApplications: ['All'],
        },
        locations: {
          includeLocations: ['All'],
          excludeLocations: [],
        },
        platforms: {
          includePlatforms: [],
          excludePlatforms: [],
        },
        clientAppTypes: ['all'],
        deviceStates: {
          includeStates: [],
          excludeStates: [],
        },
      },
      grantControls: {
        operator: 'OR',
        builtInControls: [],
        customAuthenticationFactors: [],
        termsOfUse: [],
      },
    }
  });

  const watchIncludeApplications = formControl.watch('conditions.applications.includeApplications');
  const watchIncludeLocations = formControl.watch('conditions.locations.includeLocations');

  const { fields: riskLevels, append: addRiskLevel, remove: removeRiskLevel } = useFieldArray({
    control: formControl.control,
    name: 'conditions.signInRiskLevels',
  });

  return (
    <CippFormPage
      title="Conditional Access Policy"
      formPageType="Add"
      formControl={formControl}
      postUrl="/api/AddConditionalAccessPolicy"
      queryKey={['conditional-access-policies']}
    >
      <Stack spacing={3}>
        {/* Basic Settings */}
        <CippFormSection
          title="Basic Settings"
          description="Policy identification and state"
          defaultExpanded={true}
          required={true}
        >
          <Grid container spacing={3}>
            <Grid item xs={12} md={8}>
              <CippFormComponent
                type="textField"
                name="displayName"
                label="Policy Name"
                formControl={formControl}
                validators={{ required: "Policy name is required" }}
                fullWidth
                helperText="Enter a descriptive name for this policy"
              />
            </Grid>
            <Grid item xs={12} md={4}>
              <CippFormComponent
                type="select"
                name="state"
                label="Policy State"
                formControl={formControl}
                validators={{ required: "Policy state is required" }}
                options={[
                  { value: 'disabled', label: 'Report-only' },
                  { value: 'enabled', label: 'On' },
                  { value: 'enabledForReportingButNotEnforced', label: 'Report-only' },
                ]}
                fullWidth
              />
            </Grid>
          </Grid>
        </CippFormSection>

        {/* Assignments */}
        <CippFormSection
          title="Assignments"
          description="Users, groups, and applications this policy applies to"
          required={true}
        >
          <Stack spacing={3}>
            {/* Users and Groups */}
            <Typography variant="h6">Users and Groups</Typography>
            <Grid container spacing={3}>
              <Grid item xs={12} md={6}>
                <CippFormUserSelector
                  name="conditions.users.includeUsers"
                  label="Include Users"
                  formControl={formControl}
                  multiple={true}
                  includeGroups={true}
                  validators={{ required: "At least one user must be included" }}
                />
              </Grid>
              <Grid item xs={12} md={6}>
                <CippFormUserSelector
                  name="conditions.users.excludeUsers"
                  label="Exclude Users"
                  formControl={formControl}
                  multiple={true}
                  includeGroups={true}
                />
              </Grid>
            </Grid>

            {/* Cloud Applications */}
            <Typography variant="h6">Cloud Applications</Typography>
            <Grid container spacing={3}>
              <Grid item xs={12}>
                <CippFormComponent
                  type="select"
                  name="conditions.applications.includeApplications"
                  label="Include Applications"
                  formControl={formControl}
                  multiple={true}
                  options={[
                    { value: 'All', label: 'All cloud apps' },
                    { value: 'Office365', label: 'Office 365' },
                    { value: 'MicrosoftAdminPortals', label: 'Microsoft Admin Portals' },
                  ]}
                  validators={{ required: "At least one application must be included" }}
                />
              </Grid>
              {!watchIncludeApplications?.includes('All') && (
                <Grid item xs={12}>
                  <CippFormComponent
                    type="autoComplete"
                    name="conditions.applications.specificApplications"
                    label="Specific Applications"
                    formControl={formControl}
                    multiple={true}
                    apiUrl="/api/ListApplications"
                    optionKey="appId"
                    optionLabel="displayName"
                  />
                </Grid>
              )}
            </Grid>

            {/* Conditions */}
            <Typography variant="h6">Conditions</Typography>
            <Grid container spacing={3}>
              <Grid item xs={12} md={6}>
                <CippFormComponent
                  type="select"
                  name="conditions.locations.includeLocations"
                  label="Include Locations"
                  formControl={formControl}
                  multiple={true}
                  options={[
                    { value: 'All', label: 'Any location' },
                    { value: 'AllTrusted', label: 'All trusted locations' },
                  ]}
                />
              </Grid>
              <Grid item xs={12} md={6}>
                <CippFormComponent
                  type="select"
                  name="conditions.platforms.includePlatforms"
                  label="Device Platforms"
                  formControl={formControl}
                  multiple={true}
                  options={[
                    { value: 'all', label: 'Any device' },
                    { value: 'android', label: 'Android' },
                    { value: 'iOS', label: 'iOS' },
                    { value: 'windows', label: 'Windows' },
                    { value: 'macOS', label: 'macOS' },
                    { value: 'linux', label: 'Linux' },
                  ]}
                />
              </Grid>
              <Grid item xs={12}>
                <CippFormComponent
                  type="select"
                  name="conditions.clientAppTypes"
                  label="Client Apps"
                  formControl={formControl}
                  multiple={true}
                  options={[
                    { value: 'all', label: 'Browser and mobile apps' },
                    { value: 'browser', label: 'Browser' },
                    { value: 'mobileAppsAndDesktopClients', label: 'Mobile apps and desktop clients' },
                    { value: 'exchangeActiveSync', label: 'Exchange ActiveSync clients' },
                    { value: 'other', label: 'Other clients' },
                  ]}
                />
              </Grid>
            </Grid>
          </Stack>
        </CippFormSection>

        {/* Access Controls */}
        <CippFormSection
          title="Access Controls"
          description="Grant or block access based on conditions"
          required={true}
        >
          <Grid container spacing={3}>
            <Grid item xs={12} md={4}>
              <CippFormComponent
                type="select"
                name="grantControls.operator"
                label="Require"
                formControl={formControl}
                options={[
                  { value: 'OR', label: 'One of the selected controls' },
                  { value: 'AND', label: 'All the selected controls' },
                ]}
                validators={{ required: "Grant operator is required" }}
              />
            </Grid>
            <Grid item xs={12} md={8}>
              <CippFormComponent
                type="select"
                name="grantControls.builtInControls"
                label="Grant Controls"
                formControl={formControl}
                multiple={true}
                options={[
                  { value: 'block', label: 'Block access' },
                  { value: 'mfa', label: 'Require multi-factor authentication' },
                  { value: 'compliantDevice', label: 'Require device to be marked as compliant' },
                  { value: 'domainJoinedDevice', label: 'Require Hybrid Azure AD joined device' },
                  { value: 'approvedApplication', label: 'Require approved client app' },
                  { value: 'appProtectionPolicy', label: 'Require app protection policy' },
                ]}
                validators={{ required: "At least one control must be selected" }}
              />
            </Grid>
          </Grid>
        </CippFormSection>

        {/* Session Controls */}
        <CippFormSection
          title="Session Controls"
          description="Control the user experience within applications"
        >
          <Grid container spacing={3}>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="select"
                name="sessionControls.applicationEnforcedRestrictions"
                label="Use app enforced restrictions"
                formControl={formControl}
                options={[
                  { value: null, label: 'Not configured' },
                  { value: true, label: 'Enabled' },
                ]}
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="select"
                name="sessionControls.cloudAppSecurity"
                label="Use Conditional Access App Control"
                formControl={formControl}
                options={[
                  { value: null, label: 'Not configured' },
                  { value: 'mcasConfigured', label: 'Use custom policy' },
                  { value: 'monitorOnly', label: 'Monitor only' },
                  { value: 'blockDownloads', label: 'Block downloads' },
                ]}
              />
            </Grid>
            <Grid item xs={12}>
              <CippFormComponent
                type="numberField"
                name="sessionControls.signInFrequency.value"
                label="Sign-in frequency (hours)"
                formControl={formControl}
                helperText="Require users to sign in again after this period"
                min={1}
                max={8760}
              />
            </Grid>
          </Grid>
        </CippFormSection>
      </Stack>
    </CippFormPage>
  );
};
```

## Bulk User Import Form

Form with file upload and dynamic preview:

```jsx
import { useState } from 'react';
import { CippFormComponent } from '../CippComponents/CippFormComponent';
import { CippFormPage } from '../CippFormPages/CippFormPage';

const BulkUserImportForm = () => {
  const [csvData, setCsvData] = useState([]);
  const [importPreview, setImportPreview] = useState([]);
  const [validationErrors, setValidationErrors] = useState([]);

  const formControl = useForm({
    defaultValues: {
      csvFile: null,
      defaultDomain: '',
      defaultPassword: '',
      sendWelcomeEmail: true,
      mustChangePassword: true,
      accountEnabled: true,
      defaultLicenses: [],
      defaultGroups: [],
    }
  });

  const handleFileUpload = async (file) => {
    if (!file) return;

    const text = await file.text();
    const lines = text.split('\n');
    const headers = lines[0].split(',').map(h => h.trim());
    
    const data = lines.slice(1).map((line, index) => {
      const values = line.split(',').map(v => v.trim());
      const row = {};
      headers.forEach((header, i) => {
        row[header] = values[i] || '';
      });
      row._rowIndex = index + 1;
      return row;
    }).filter(row => Object.values(row).some(val => val !== ''));

    setCsvData(data);
    validateImportData(data);
  };

  const validateImportData = (data) => {
    const errors = [];
    const requiredFields = ['firstName', 'lastName', 'email'];

    data.forEach((row, index) => {
      requiredFields.forEach(field => {
        if (!row[field]) {
          errors.push({
            row: index + 1,
            field,
            message: `${field} is required`,
          });
        }
      });

      // Validate email format
      if (row.email && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(row.email)) {
        errors.push({
          row: index + 1,
          field: 'email',
          message: 'Invalid email format',
        });
      }
    });

    setValidationErrors(errors);
    setImportPreview(data);
  };

  const customDataFormatter = (formData) => {
    return {
      ...formData,
      users: importPreview.map(user => ({
        ...user,
        userPrincipalName: `${user.email.split('@')[0]}@${formData.defaultDomain}`,
        passwordProfile: {
          password: formData.defaultPassword || generateRandomPassword(),
          forceChangePasswordNextSignIn: formData.mustChangePassword,
        },
        accountEnabled: formData.accountEnabled,
        licenses: formData.defaultLicenses,
        groups: formData.defaultGroups,
      })),
    };
  };

  return (
    <CippFormPage
      title="Bulk User Import"
      formPageType="Import"
      formControl={formControl}
      postUrl="/api/BulkAddUsers"
      queryKey={['users', 'bulk-import']}
      customDataformatter={customDataFormatter}
      allowResubmit={true}
    >
      <Stack spacing={3}>
        {/* File Upload */}
        <CippFormSection
          title="CSV File Upload"
          description="Upload a CSV file with user data"
          defaultExpanded={true}
          required={true}
        >
          <Grid container spacing={3}>
            <Grid item xs={12}>
              <CippFormComponent
                type="filePicker"
                name="csvFile"
                label="CSV File"
                formControl={formControl}
                accept=".csv"
                onFileSelect={handleFileUpload}
                helperText="CSV should include columns: firstName, lastName, email, department, jobTitle"
                required
              />
            </Grid>
            <Grid item xs={12}>
              <Alert severity="info">
                <Typography variant="body2">
                  <strong>Required columns:</strong> firstName, lastName, email<br />
                  <strong>Optional columns:</strong> department, jobTitle, manager, officeLocation, phoneNumber
                </Typography>
              </Alert>
            </Grid>
          </Grid>
        </CippFormSection>

        {/* Default Settings */}
        <CippFormSection
          title="Default Settings"
          description="Default values applied to all imported users"
        >
          <Grid container spacing={3}>
            <Grid item xs={12} md={6}>
              <CippFormTenantSelector
                name="defaultDomain"
                label="Default Domain"
                formControl={formControl}
                required
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="passwordField"
                name="defaultPassword"
                label="Default Password"
                formControl={formControl}
                helperText="Leave blank to generate random passwords"
              />
            </Grid>
            <Grid item xs={12} md={4}>
              <CippFormComponent
                type="switch"
                name="accountEnabled"
                label="Account Enabled"
                formControl={formControl}
              />
            </Grid>
            <Grid item xs={12} md={4}>
              <CippFormComponent
                type="switch"
                name="mustChangePassword"
                label="Must Change Password"
                formControl={formControl}
              />
            </Grid>
            <Grid item xs={12} md={4}>
              <CippFormComponent
                type="switch"
                name="sendWelcomeEmail"
                label="Send Welcome Email"
                formControl={formControl}
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormLicenseSelector
                name="defaultLicenses"
                label="Default Licenses"
                formControl={formControl}
                multiple={true}
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormGroupSelector
                name="defaultGroups"
                label="Default Groups"
                formControl={formControl}
                multiple={true}
              />
            </Grid>
          </Grid>
        </CippFormSection>

        {/* Import Preview */}
        {importPreview.length > 0 && (
          <CippFormSection
            title="Import Preview"
            description={`${importPreview.length} users ready for import`}
          >
            {validationErrors.length > 0 && (
              <Alert severity="error" sx={{ mb: 2 }}>
                <Typography variant="body2" fontWeight="medium">
                  Validation Errors ({validationErrors.length}):
                </Typography>
                <ul style={{ margin: 0, paddingLeft: '20px' }}>
                  {validationErrors.slice(0, 10).map((error, index) => (
                    <li key={index}>
                      Row {error.row}, {error.field}: {error.message}
                    </li>
                  ))}
                  {validationErrors.length > 10 && (
                    <li>... and {validationErrors.length - 10} more errors</li>
                  )}
                </ul>
              </Alert>
            )}

            <TableContainer component={Paper} sx={{ maxHeight: 400 }}>
              <Table stickyHeader size="small">
                <TableHead>
                  <TableRow>
                    <TableCell>First Name</TableCell>
                    <TableCell>Last Name</TableCell>
                    <TableCell>Email</TableCell>
                    <TableCell>Department</TableCell>
                    <TableCell>Job Title</TableCell>
                    <TableCell>Status</TableCell>
                  </TableRow>
                </TableHead>
                <TableBody>
                  {importPreview.slice(0, 100).map((user, index) => {
                    const rowErrors = validationErrors.filter(e => e.row === index + 1);
                    const hasErrors = rowErrors.length > 0;
                    
                    return (
                      <TableRow 
                        key={index}
                        sx={{ backgroundColor: hasErrors ? 'error.lighter' : 'inherit' }}
                      >
                        <TableCell>{user.firstName}</TableCell>
                        <TableCell>{user.lastName}</TableCell>
                        <TableCell>{user.email}</TableCell>
                        <TableCell>{user.department}</TableCell>
                        <TableCell>{user.jobTitle}</TableCell>
                        <TableCell>
                          {hasErrors ? (
                            <Chip label="Error" color="error" size="small" />
                          ) : (
                            <Chip label="Ready" color="success" size="small" />
                          )}
                        </TableCell>
                      </TableRow>
                    );
                  })}
                </TableBody>
              </Table>
            </TableContainer>

            {importPreview.length > 100 && (
              <Typography variant="body2" color="text.secondary" sx={{ mt: 1 }}>
                Showing first 100 users. Total: {importPreview.length}
              </Typography>
            )}
          </CippFormSection>
        )}
      </Stack>
    </CippFormPage>
  );
};
```

## Settings Configuration Form

Complex settings form with nested configurations:

```jsx

const SettingsForm = () => {
  const { data: currentSettings, isLoading } = ApiGetCall({
    url: '/api/GetSettings',
    queryKey: 'settings',
  });

  const formControl = useForm({
    mode: 'onChange',
    defaultValues: {
      general: {
        organizationName: '',
        supportEmail: '',
        logoUrl: '',
        timezone: 'UTC',
      },
      security: {
        passwordPolicy: {
          minimumLength: 8,
          requireUppercase: true,
          requireLowercase: true,
          requireNumbers: true,
          requireSymbols: false,
          passwordHistory: 12,
          maxPasswordAge: 90,
        },
        accountLockout: {
          enabled: true,
          threshold: 5,
          duration: 30,
          resetCounter: 30,
        },
      },
      notifications: {
        emailNotifications: true,
        smsNotifications: false,
        webhookUrl: '',
        alertTypes: [],
      },
    }
  });

  // Load current settings when available
  useEffect(() => {
    if (currentSettings && !isLoading) {
      formControl.reset(currentSettings);
    }
  }, [currentSettings, isLoading]);

  return (
    <CippFormPage
      title="Settings"
      formPageType="Configure"
      formControl={formControl}
      postUrl="/api/UpdateSettings"
      queryKey="settings"
      hideBackButton={true}
      allowResubmit={true}
    >
      <Stack spacing={3}>
        {/* General Settings */}
        <CippFormSection
          title="General Settings"
          description="Basic organization information"
          defaultExpanded={true}
        >
          <Grid container spacing={3}>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="textField"
                name="general.organizationName"
                label="Organization Name"
                formControl={formControl}
                validators={{ required: "Organization name is required" }}
                fullWidth
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="emailField"
                name="general.supportEmail"
                label="Support Email"
                formControl={formControl}
                fullWidth
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="textField"
                name="general.logoUrl"
                label="Logo URL"
                formControl={formControl}
                validators={{
                  pattern: {
                    value: /^https?:\/\/.+/,
                    message: "Invalid URL format"
                  }
                }}
                fullWidth
                helperText="URL to your organization logo"
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="select"
                name="general.timezone"
                label="Default Timezone"
                formControl={formControl}
                options={timezoneOptions}
                fullWidth
              />
            </Grid>
          </Grid>
        </CippFormSection>

        {/* Security Settings */}
        <CippFormSection
          title="Security Settings"
          description="Password and account security policies"
        >
          <Stack spacing={3}>
            {/* Password Policy */}
            <Typography variant="h6">Password Policy</Typography>
            <Grid container spacing={3}>
              <Grid item xs={12} md={6}>
                <CippFormComponent
                  type="numberField"
                  name="security.passwordPolicy.minimumLength"
                  label="Minimum Length"
                  formControl={formControl}
                  validators={{
                    min: { value: 8, message: "Minimum length must be at least 8" },
                    max: { value: 128, message: "Maximum length cannot exceed 128" }
                  }}
                  fullWidth
                />
              </Grid>
              <Grid item xs={12} md={6}>
                <CippFormComponent
                  type="numberField"
                  name="security.passwordPolicy.passwordHistory"
                  label="Password History"
                  formControl={formControl}
                  validators={{
                    min: { value: 0, message: "Cannot be negative" },
                    max: { value: 24, message: "Cannot exceed 24" }
                  }}
                  fullWidth
                  helperText="Number of previous passwords to remember"
                />
              </Grid>
              <Grid item xs={12} md={3}>
                <CippFormComponent
                  type="switch"
                  name="security.passwordPolicy.requireUppercase"
                  label="Require Uppercase"
                  formControl={formControl}
                />
              </Grid>
              <Grid item xs={12} md={3}>
                <CippFormComponent
                  type="switch"
                  name="security.passwordPolicy.requireLowercase"
                  label="Require Lowercase"
                  formControl={formControl}
                />
              </Grid>
              <Grid item xs={12} md={3}>
                <CippFormComponent
                  type="switch"
                  name="security.passwordPolicy.requireNumbers"
                  label="Require Numbers"
                  formControl={formControl}
                />
              </Grid>
              <Grid item xs={12} md={3}>
                <CippFormComponent
                  type="switch"
                  name="security.passwordPolicy.requireSymbols"
                  label="Require Symbols"
                  formControl={formControl}
                />
              </Grid>
            </Grid>

            {/* Account Lockout */}
            <Typography variant="h6">Account Lockout</Typography>
            <Grid container spacing={3}>
              <Grid item xs={12}>
                <CippFormComponent
                  type="switch"
                  name="security.accountLockout.enabled"
                  label="Enable Account Lockout"
                  formControl={formControl}
                />
              </Grid>
              <Grid item xs={12} md={4}>
                <CippFormComponent
                  type="numberField"
                  name="security.accountLockout.threshold"
                  label="Lockout Threshold"
                  formControl={formControl}
                  validators={{
                    min: { value: 1, message: "Must be at least 1" },
                    max: { value: 999, message: "Cannot exceed 999" }
                  }}
                  fullWidth
                  helperText="Failed attempts before lockout"
                />
              </Grid>
              <Grid item xs={12} md={4}>
                <CippFormComponent
                  type="numberField"
                  name="security.accountLockout.duration"
                  label="Lockout Duration (minutes)"
                  formControl={formControl}
                  min={1}
                  max={99999}
                  fullWidth
                />
              </Grid>
              <Grid item xs={12} md={4}>
                <CippFormComponent
                  type="numberField"
                  name="security.accountLockout.resetCounter"
                  label="Reset Counter After (minutes)"
                  formControl={formControl}
                  min={1}
                  max={99999}
                  fullWidth
                />
              </Grid>
            </Grid>
          </Stack>
        </CippFormSection>

        {/* Notification Settings */}
        <CippFormSection
          title="Notification Settings"
          description="Configure alert and notification preferences"
        >
          <Grid container spacing={3}>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="switch"
                name="notifications.emailNotifications"
                label="Email Notifications"
                formControl={formControl}
              />
            </Grid>
            <Grid item xs={12} md={6}>
              <CippFormComponent
                type="switch"
                name="notifications.smsNotifications"
                label="SMS Notifications"
                formControl={formControl}
              />
            </Grid>
            <Grid item xs={12}>
              <CippFormComponent
                type="textField"
                name="notifications.webhookUrl"
                label="Webhook URL"
                formControl={formControl}
                fullWidth
                helperText="URL to send webhook notifications"
              />
            </Grid>
            <Grid item xs={12}>
              <CippFormComponent
                type="select"
                name="notifications.alertTypes"
                label="Alert Types"
                formControl={formControl}
                multiple={true}
                options={[
                  { value: 'security', label: 'Security Alerts' },
                  { value: 'system', label: 'System Alerts' },
                  { value: 'user', label: 'User Events' },
                  { value: 'compliance', label: 'Compliance Alerts' },
                ]}
                fullWidth
              />
            </Grid>
          </Grid>
        </CippFormSection>
      </Stack>
    </CippFormPage>
  );
};
```