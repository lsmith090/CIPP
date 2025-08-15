# CippWizard Examples

Comprehensive examples of using CIPP wizard components for complex multi-step workflows.

## User Onboarding Wizard

Complete user onboarding workflow with conditional steps:

```jsx
import { CippWizard, CippWizardPage, CippWizardStepButtons } from '../CippWizard';
import { CippFormComponent } from '../CippComponents/CippFormComponent';
import { useForm } from 'react-hook-form';
import * as yup from 'yup';

// Validation schemas for each step
const userInfoSchema = yup.object({
  firstName: yup.string().required('First name is required'),
  lastName: yup.string().required('Last name is required'),
  email: yup.string().email('Invalid email').required('Email is required'),
  department: yup.string().required('Department is required'),
  jobTitle: yup.string().required('Job title is required'),
});

const accessSchema = yup.object({
  userType: yup.string().required('User type is required'),
  manager: yup.string().when('userType', {
    is: 'employee',
    then: yup.string().required('Manager is required for employees'),
  }),
  licenses: yup.array().min(1, 'At least one license must be selected'),
});

const securitySchema = yup.object({
  mfaRequired: yup.boolean(),
  startDate: yup.date().required('Start date is required'),
  endDate: yup.date().when('userType', {
    is: 'contractor',
    then: yup.date().required('End date is required for contractors'),
  }),
});

// Step Components
const UserInfoStep = ({ formControl, title, onNextStep, onPreviousStep, currentStep, lastStep }) => {
  const departments = [
    { value: 'IT', label: 'Information Technology' },
    { value: 'HR', label: 'Human Resources' },
    { value: 'Finance', label: 'Finance' },
    { value: 'Sales', label: 'Sales' },
    { value: 'Marketing', label: 'Marketing' },
  ];

  return (
    <CippWizardPage
      title={title}
      subtitle="Enter the new user's basic information"
      helpText="This information will be used to create the user account and set up their profile"
    >
      <Grid container spacing={3}>
        <Grid item xs={12} md={6}>
          <CippFormComponent
            type="textField"
            name="firstName"
            label="First Name"
            formControl={formControl}
            required
            fullWidth
          />
        </Grid>
        <Grid item xs={12} md={6}>
          <CippFormComponent
            type="textField"
            name="lastName"
            label="Last Name"
            formControl={formControl}
            required
            fullWidth
          />
        </Grid>
        <Grid item xs={12}>
          <CippFormComponent
            type="emailField"
            name="email"
            label="Email Address"
            formControl={formControl}
            required
            fullWidth
            helperText="This will be the user's primary email address"
          />
        </Grid>
        <Grid item xs={12} md={6}>
          <CippFormComponent
            type="select"
            name="department"
            label="Department"
            formControl={formControl}
            options={departments}
            required
            fullWidth
          />
        </Grid>
        <Grid item xs={12} md={6}>
          <CippFormComponent
            type="textField"
            name="jobTitle"
            label="Job Title"
            formControl={formControl}
            required
            fullWidth
          />
        </Grid>
        <Grid item xs={12}>
          <CippFormComponent
            type="textFieldMultiline"
            name="notes"
            label="Additional Notes"
            formControl={formControl}
            rows={3}
            fullWidth
            helperText="Any additional information about this user"
          />
        </Grid>
      </Grid>

      <CippWizardStepButtons
        currentStep={currentStep}
        lastStep={lastStep}
        onNextStep={onNextStep}
        onPreviousStep={onPreviousStep}
      />
    </CippWizardPage>
  );
};

const AccessRightsStep = ({ formControl, title, onNextStep, onPreviousStep, currentStep, lastStep }) => {
  const userType = formControl.watch('userType');

  return (
    <CippWizardPage
      title={title}
      subtitle="Configure user access rights and permissions"
    >
      <Grid container spacing={3}>
        <Grid item xs={12}>
          <CippFormComponent
            type="select"
            name="userType"
            label="User Type"
            formControl={formControl}
            options={[
              { value: 'employee', label: 'Employee' },
              { value: 'contractor', label: 'Contractor' },
              { value: 'partner', label: 'Partner' },
              { value: 'guest', label: 'Guest' },
            ]}
            required
            fullWidth
          />
        </Grid>

        {userType === 'employee' && (
          <Grid item xs={12}>
            <CippFormUserSelector
              name="manager"
              label="Manager"
              formControl={formControl}
              userType="users"
              required
            />
          </Grid>
        )}

        <Grid item xs={12}>
          <CippFormLicenseSelector
            name="licenses"
            label="License Assignment"
            formControl={formControl}
            multiple={true}
            required
          />
        </Grid>

        <Grid item xs={12}>
          <CippFormGroupSelector
            name="groups"
            label="Group Membership"
            formControl={formControl}
            multiple={true}
            groupTypes={['security', 'distribution']}
          />
        </Grid>

        {(userType === 'contractor' || userType === 'partner') && (
          <Grid item xs={12}>
            <Alert severity="info">
              External users will have limited access and require manager approval for certain actions.
            </Alert>
          </Grid>
        )}
      </Grid>

      <CippWizardStepButtons
        currentStep={currentStep}
        lastStep={lastStep}
        onNextStep={onNextStep}
        onPreviousStep={onPreviousStep}
      />
    </CippWizardPage>
  );
};

const SecuritySettingsStep = ({ formControl, title, onNextStep, onPreviousStep, currentStep, lastStep }) => {
  const userType = formControl.watch('userType');
  const mfaRequired = formControl.watch('mfaRequired');

  return (
    <CippWizardPage
      title={title}
      subtitle="Configure security settings and access duration"
    >
      <Grid container spacing={3}>
        <Grid item xs={12}>
          <FormControlLabel
            control={
              <CippFormComponent
                type="switch"
                name="mfaRequired"
                formControl={formControl}
              />
            }
            label="Require Multi-Factor Authentication"
          />
          <Typography variant="caption" color="text.secondary" display="block">
            Highly recommended for all users with administrative access
          </Typography>
        </Grid>

        {mfaRequired && (
          <Grid item xs={12}>
            <CippFormComponent
              type="select"
              name="mfaMethod"
              label="Preferred MFA Method"
              formControl={formControl}
              options={[
                { value: 'authenticator', label: 'Authenticator App' },
                { value: 'phone', label: 'Phone Call' },
                { value: 'sms', label: 'SMS Text' },
              ]}
              fullWidth
            />
          </Grid>
        )}

        <Grid item xs={12} md={6}>
          <CippFormComponent
            type="datePicker"
            name="startDate"
            label="Access Start Date"
            formControl={formControl}
            required
            fullWidth
            disablePast={true}
          />
        </Grid>

        {(userType === 'contractor' || userType === 'partner') && (
          <Grid item xs={12} md={6}>
            <CippFormComponent
              type="datePicker"
              name="endDate"
              label="Access End Date"
              formControl={formControl}
              required
              fullWidth
              disablePast={true}
            />
          </Grid>
        )}

        <Grid item xs={12}>
          <CippFormComponent
            type="switch"
            name="mustChangePassword"
            label="Must Change Password on First Login"
            formControl={formControl}
          />
        </Grid>

        <Grid item xs={12}>
          <CippFormComponent
            type="switch"
            name="sendWelcomeEmail"
            label="Send Welcome Email"
            formControl={formControl}
          />
        </Grid>
      </Grid>

      <CippWizardStepButtons
        currentStep={currentStep}
        lastStep={lastStep}
        onNextStep={onNextStep}
        onPreviousStep={onPreviousStep}
      />
    </CippWizardPage>
  );
};

const ReviewStep = ({ formControl, onPreviousStep, currentStep, onComplete, isSubmitting }) => {
  const formData = formControl.getValues();

  const reviewSections = [
    {
      title: 'User Information',
      data: [
        { label: 'Name', value: `${formData.firstName} ${formData.lastName}` },
        { label: 'Email', value: formData.email },
        { label: 'Department', value: formData.department },
        { label: 'Job Title', value: formData.jobTitle },
      ],
    },
    {
      title: 'Access Rights',
      data: [
        { label: 'User Type', value: formData.userType },
        { label: 'Manager', value: formData.manager?.displayName || 'Not set' },
        { label: 'Licenses', value: formData.licenses?.map(l => l.skuPartNumber).join(', ') || 'None' },
        { label: 'Groups', value: formData.groups?.map(g => g.displayName).join(', ') || 'None' },
      ],
    },
    {
      title: 'Security Settings',
      data: [
        { label: 'MFA Required', value: formData.mfaRequired ? 'Yes' : 'No' },
        { label: 'Start Date', value: formData.startDate ? new Date(formData.startDate).toLocaleDateString() : 'Not set' },
        { label: 'End Date', value: formData.endDate ? new Date(formData.endDate).toLocaleDateString() : 'No expiration' },
        { label: 'Must Change Password', value: formData.mustChangePassword ? 'Yes' : 'No' },
      ],
    },
  ];

  return (
    <CippWizardPage
      title="Review User Configuration"
      subtitle="Please review the user details before creating the account"
    >
      <Stack spacing={3}>
        {reviewSections.map((section, index) => (
          <Card key={index} variant="outlined">
            <CardHeader title={section.title} />
            <CardContent>
              <Grid container spacing={2}>
                {section.data.map((item, itemIndex) => (
                  <Grid item xs={12} md={6} key={itemIndex}>
                    <Typography variant="subtitle2" color="text.secondary">
                      {item.label}
                    </Typography>
                    <Typography variant="body1">
                      {item.value || 'Not specified'}
                    </Typography>
                  </Grid>
                ))}
              </Grid>
            </CardContent>
          </Card>
        ))}

        <Alert severity="info">
          <Typography variant="body2">
            After creating this user account, they will receive setup instructions via email 
            {formData.sendWelcomeEmail ? ' (welcome email enabled)' : ' (welcome email disabled)'}.
          </Typography>
        </Alert>
      </Stack>

      <Box sx={{ display: 'flex', justifyContent: 'space-between', mt: 3 }}>
        <Button
          onClick={onPreviousStep}
          disabled={isSubmitting}
        >
          Previous
        </Button>
        <Button
          variant="contained"
          onClick={onComplete}
          disabled={isSubmitting}
          startIcon={isSubmitting ? <CircularProgress size={20} /> : <CheckIcon />}
        >
          {isSubmitting ? 'Creating User...' : 'Create User'}
        </Button>
      </Box>
    </CippWizardPage>
  );
};

// Main Wizard Component
const UserOnboardingWizard = () => {
  const router = useRouter();

  const wizardSteps = [
    {
      label: 'User Information',
      description: 'Basic user details',
      component: UserInfoStep,
      componentProps: {
        title: 'User Information',
      },
      validation: userInfoSchema,
      required: true,
    },
    {
      label: 'Access Rights',
      description: 'Permissions and licenses',
      component: AccessRightsStep,
      componentProps: {
        title: 'Access Rights',
      },
      validation: accessSchema,
      required: true,
    },
    {
      label: 'Security Settings',
      description: 'Security configuration',
      component: SecuritySettingsStep,
      componentProps: {
        title: 'Security Settings',
      },
      validation: securitySchema,
    },
    {
      label: 'Review & Create',
      description: 'Review and complete',
      component: ReviewStep,
      required: true,
    },
  ];

  const initialState = {
    firstName: '',
    lastName: '',
    email: '',
    department: '',
    jobTitle: '',
    notes: '',
    userType: 'employee',
    manager: null,
    licenses: [],
    groups: [],
    mfaRequired: true,
    mfaMethod: 'authenticator',
    startDate: new Date(),
    endDate: null,
    mustChangePassword: true,
    sendWelcomeEmail: true,
  };

  const handleComplete = (result) => {
    showNotification('User created successfully!');
    router.push('/identity/administration/users');
  };

  return (
    <Box sx={{ maxWidth: 1200, mx: 'auto', p: 3 }}>
      <Typography variant="h4" gutterBottom>
        New User Onboarding
      </Typography>
      <Typography variant="body1" color="text.secondary" sx={{ mb: 4 }}>
        Follow the steps below to create a new user account and configure their access.
      </Typography>

      <CippWizard
        steps={wizardSteps}
        postUrl="/api/AddUser"
        orientation="horizontal"
        initialState={initialState}
        onComplete={handleComplete}
      />
    </Box>
  );
};
```

## Tenant Setup Wizard

Multi-tenant configuration wizard with branching logic:

```jsx
import { useState } from 'react';

const TenantSetupWizard = () => {
  const [selectedTenants, setSelectedTenants] = useState([]);
  const [setupType, setSetupType] = useState('');

  const TenantSelectionStep = ({ formControl, onNextStep, currentStep, lastStep }) => {
    const { data: tenants, isLoading } = ApiGetCall({
      url: '/api/ListTenants',
      queryKey: 'tenants-list',
    });

    return (
      <CippWizardPage
        title="Select Tenants"
        subtitle="Choose which tenants to configure"
        loading={isLoading}
      >
        <Grid container spacing={3}>
          <Grid item xs={12}>
            <CippFormTenantSelector
              name="targetTenants"
              label="Target Tenants"
              formControl={formControl}
              multiple={true}
              required
            />
          </Grid>
          <Grid item xs={12}>
            <FormControlLabel
              control={
                <Checkbox
                  checked={formControl.watch('selectAll')}
                  onChange={(e) => {
                    formControl.setValue('selectAll', e.target.checked);
                    if (e.target.checked) {
                      formControl.setValue('targetTenants', tenants?.map(t => t.id) || []);
                    }
                  }}
                />
              }
              label="Select all tenants"
            />
          </Grid>
        </Grid>

        <CippWizardStepButtons
          currentStep={currentStep}
          lastStep={lastStep}
          onNextStep={onNextStep}
          isValid={formControl.watch('targetTenants')?.length > 0}
        />
      </CippWizardPage>
    );
  };

  const SetupTypeStep = ({ formControl, onNextStep, onPreviousStep, currentStep, lastStep }) => {
    const setupOptions = [
      {
        value: 'basic',
        label: 'Basic Setup',
        description: 'Configure essential security settings and policies',
        icon: <ShieldCheckIcon />,
        features: ['Password Policy', 'MFA Settings', 'Basic Conditional Access'],
      },
      {
        value: 'comprehensive',
        label: 'Comprehensive Setup',
        description: 'Full security and compliance configuration',
        icon: <CogIcon />,
        features: ['All Basic Features', 'Advanced Conditional Access', 'Compliance Policies', 'Data Loss Prevention'],
      },
      {
        value: 'custom',
        label: 'Custom Setup',
        description: 'Select specific features to configure',
        icon: <WrenchIcon />,
        features: ['Choose Individual Features', 'Advanced Configuration Options'],
      },
    ];

    return (
      <CippWizardPage
        title="Setup Type"
        subtitle="Choose your configuration approach"
      >
        <Grid container spacing={3}>
          {setupOptions.map((option) => (
            <Grid item xs={12} md={4} key={option.value}>
              <Card
                sx={{
                  cursor: 'pointer',
                  border: 2,
                  borderColor: formControl.watch('setupType') === option.value ? 'primary.main' : 'transparent',
                  '&:hover': { borderColor: 'primary.light' },
                }}
                onClick={() => formControl.setValue('setupType', option.value)}
              >
                <CardContent sx={{ textAlign: 'center', p: 3 }}>
                  <Box sx={{ mb: 2, color: 'primary.main' }}>
                    {option.icon}
                  </Box>
                  <Typography variant="h6" gutterBottom>
                    {option.label}
                  </Typography>
                  <Typography variant="body2" color="text.secondary" sx={{ mb: 2 }}>
                    {option.description}
                  </Typography>
                  <Divider sx={{ my: 2 }} />
                  <Typography variant="subtitle2" gutterBottom>
                    Features:
                  </Typography>
                  <List dense>
                    {option.features.map((feature, index) => (
                      <ListItem key={index} sx={{ py: 0.5 }}>
                        <ListItemIcon sx={{ minWidth: 20 }}>
                          <CheckIcon fontSize="small" color="success" />
                        </ListItemIcon>
                        <ListItemText
                          primary={feature}
                          primaryTypographyProps={{ variant: 'body2' }}
                        />
                      </ListItem>
                    ))}
                  </List>
                </CardContent>
              </Card>
            </Grid>
          ))}
        </Grid>

        <CippWizardStepButtons
          currentStep={currentStep}
          lastStep={lastStep}
          onNextStep={onNextStep}
          onPreviousStep={onPreviousStep}
          isValid={!!formControl.watch('setupType')}
        />
      </CippWizardPage>
    );
  };

  const SecurityPolicyStep = ({ formControl, onNextStep, onPreviousStep, currentStep, lastStep }) => {
    return (
      <CippWizardPage
        title="Security Policies"
        subtitle="Configure password and authentication policies"
      >
        <Stack spacing={3}>
          {/* Password Policy */}
          <Card variant="outlined">
            <CardHeader title="Password Policy" />
            <CardContent>
              <Grid container spacing={3}>
                <Grid item xs={12} md={6}>
                  <CippFormComponent
                    type="numberField"
                    name="passwordPolicy.minimumLength"
                    label="Minimum Length"
                    formControl={formControl}
                    min={8}
                    max={128}
                    fullWidth
                  />
                </Grid>
                <Grid item xs={12} md={6}>
                  <CippFormComponent
                    type="numberField"
                    name="passwordPolicy.passwordHistory"
                    label="Password History"
                    formControl={formControl}
                    min={0}
                    max={24}
                    fullWidth
                  />
                </Grid>
                <Grid item xs={12} md={3}>
                  <CippFormComponent
                    type="switch"
                    name="passwordPolicy.requireUppercase"
                    label="Require Uppercase"
                    formControl={formControl}
                  />
                </Grid>
                <Grid item xs={12} md={3}>
                  <CippFormComponent
                    type="switch"
                    name="passwordPolicy.requireLowercase"
                    label="Require Lowercase"
                    formControl={formControl}
                  />
                </Grid>
                <Grid item xs={12} md={3}>
                  <CippFormComponent
                    type="switch"
                    name="passwordPolicy.requireNumbers"
                    label="Require Numbers"
                    formControl={formControl}
                  />
                </Grid>
                <Grid item xs={12} md={3}>
                  <CippFormComponent
                    type="switch"
                    name="passwordPolicy.requireSymbols"
                    label="Require Symbols"
                    formControl={formControl}
                  />
                </Grid>
              </Grid>
            </CardContent>
          </Card>

          {/* MFA Settings */}
          <Card variant="outlined">
            <CardHeader title="Multi-Factor Authentication" />
            <CardContent>
              <Grid container spacing={3}>
                <Grid item xs={12}>
                  <CippFormComponent
                    type="switch"
                    name="mfaSettings.enableMFA"
                    label="Enable MFA for All Users"
                    formControl={formControl}
                  />
                </Grid>
                <Grid item xs={12}>
                  <CippFormComponent
                    type="select"
                    name="mfaSettings.allowedMethods"
                    label="Allowed MFA Methods"
                    formControl={formControl}
                    multiple={true}
                    options={[
                      { value: 'authenticator', label: 'Authenticator App' },
                      { value: 'phone', label: 'Phone Call' },
                      { value: 'sms', label: 'SMS Text' },
                      { value: 'email', label: 'Email' },
                    ]}
                    fullWidth
                  />
                </Grid>
                <Grid item xs={12} md={6}>
                  <CippFormComponent
                    type="numberField"
                    name="mfaSettings.rememberDeviceDays"
                    label="Remember Device (Days)"
                    formControl={formControl}
                    min={0}
                    max={365}
                    fullWidth
                  />
                </Grid>
              </Grid>
            </CardContent>
          </Card>
        </Stack>

        <CippWizardStepButtons
          currentStep={currentStep}
          lastStep={lastStep}
          onNextStep={onNextStep}
          onPreviousStep={onPreviousStep}
        />
      </CippWizardPage>
    );
  };

  const ConditionalAccessStep = ({ formControl, onNextStep, onPreviousStep, currentStep, lastStep }) => {
    const setupType = formControl.watch('setupType');
    
    // Only show this step for comprehensive or custom setup
    if (setupType === 'basic') {
      return null;
    }

    return (
      <CippWizardPage
        title="Conditional Access"
        subtitle="Configure location and device-based access controls"
      >
        <Stack spacing={3}>
          <Card variant="outlined">
            <CardHeader 
              title="Block Access from Untrusted Locations"
              action={
                <CippFormComponent
                  type="switch"
                  name="conditionalAccess.blockUntrustedLocations"
                  formControl={formControl}
                />
              }
            />
            <CardContent>
              <Typography variant="body2" color="text.secondary">
                Block sign-ins from locations outside your organization's trusted network.
              </Typography>
            </CardContent>
          </Card>

          <Card variant="outlined">
            <CardHeader 
              title="Require MFA for Admin Accounts"
              action={
                <CippFormComponent
                  type="switch"
                  name="conditionalAccess.requireMFAForAdmins"
                  formControl={formControl}
                />
              }
            />
            <CardContent>
              <Typography variant="body2" color="text.secondary">
                Require multi-factor authentication for all administrative accounts.
              </Typography>
            </CardContent>
          </Card>

          <Card variant="outlined">
            <CardHeader 
              title="Block Legacy Authentication"
              action={
                <CippFormComponent
                  type="switch"
                  name="conditionalAccess.blockLegacyAuth"
                  formControl={formControl}
                />
              }
            />
            <CardContent>
              <Typography variant="body2" color="text.secondary">
                Block authentication from legacy protocols that don't support modern security features.
              </Typography>
            </CardContent>
          </Card>

          <Card variant="outlined">
            <CardHeader 
              title="Require Compliant Devices"
              action={
                <CippFormComponent
                  type="switch"
                  name="conditionalAccess.requireCompliantDevices"
                  formControl={formControl}
                />
              }
            />
            <CardContent>
              <Typography variant="body2" color="text.secondary">
                Allow access only from devices that meet your compliance policies.
              </Typography>
            </CardContent>
          </Card>
        </Stack>

        <CippWizardStepButtons
          currentStep={currentStep}
          lastStep={lastStep}
          onNextStep={onNextStep}
          onPreviousStep={onPreviousStep}
        />
      </CippWizardPage>
    );
  };

  const ReviewAndDeployStep = ({ formControl, onPreviousStep, onComplete, isSubmitting }) => {
    const formData = formControl.getValues();
    const [deploymentProgress, setDeploymentProgress] = useState({});

    const handleDeploy = async () => {
      const tenants = formData.targetTenants;
      const totalTenants = tenants.length;
      
      for (let i = 0; i < tenants.length; i++) {
        const tenant = tenants[i];
        setDeploymentProgress(prev => ({
          ...prev,
          [tenant]: { status: 'deploying', progress: 0 }
        }));

        try {
          await deployToTenant(tenant, formData);
          setDeploymentProgress(prev => ({
            ...prev,
            [tenant]: { status: 'completed', progress: 100 }
          }));
        } catch (error) {
          setDeploymentProgress(prev => ({
            ...prev,
            [tenant]: { status: 'error', progress: 0, error: error.message }
          }));
        }
      }

      onComplete();
    };

    return (
      <CippWizardPage
        title="Review & Deploy"
        subtitle="Review configuration and deploy to selected tenants"
      >
        <Stack spacing={3}>
          {/* Configuration Summary */}
          <Card variant="outlined">
            <CardHeader title="Configuration Summary" />
            <CardContent>
              <Grid container spacing={2}>
                <Grid item xs={12} md={6}>
                  <Typography variant="subtitle2">Setup Type</Typography>
                  <Typography variant="body2" sx={{ mb: 2 }}>
                    {formData.setupType}
                  </Typography>
                </Grid>
                <Grid item xs={12} md={6}>
                  <Typography variant="subtitle2">Target Tenants</Typography>
                  <Typography variant="body2" sx={{ mb: 2 }}>
                    {formData.targetTenants?.length || 0} tenants selected
                  </Typography>
                </Grid>
                <Grid item xs={12}>
                  <Typography variant="subtitle2">Enabled Features</Typography>
                  <List dense>
                    {formData.passwordPolicy && (
                      <ListItem>
                        <ListItemIcon><CheckIcon color="success" /></ListItemIcon>
                        <ListItemText primary="Password Policy" />
                      </ListItem>
                    )}
                    {formData.mfaSettings?.enableMFA && (
                      <ListItem>
                        <ListItemIcon><CheckIcon color="success" /></ListItemIcon>
                        <ListItemText primary="Multi-Factor Authentication" />
                      </ListItem>
                    )}
                    {formData.conditionalAccess?.blockUntrustedLocations && (
                      <ListItem>
                        <ListItemIcon><CheckIcon color="success" /></ListItemIcon>
                        <ListItemText primary="Block Untrusted Locations" />
                      </ListItem>
                    )}
                  </List>
                </Grid>
              </Grid>
            </CardContent>
          </Card>

          {/* Deployment Progress */}
          {Object.keys(deploymentProgress).length > 0 && (
            <Card variant="outlined">
              <CardHeader title="Deployment Progress" />
              <CardContent>
                {Object.entries(deploymentProgress).map(([tenant, status]) => (
                  <Box key={tenant} sx={{ mb: 2 }}>
                    <Box sx={{ display: 'flex', alignItems: 'center', mb: 1 }}>
                      <Typography variant="body2" sx={{ flexGrow: 1 }}>
                        {tenant}
                      </Typography>
                      {status.status === 'completed' && <CheckIcon color="success" />}
                      {status.status === 'error' && <XMarkIcon color="error" />}
                      {status.status === 'deploying' && <CircularProgress size={20} />}
                    </Box>
                    <LinearProgress 
                      variant="determinate" 
                      value={status.progress} 
                      color={status.status === 'error' ? 'error' : 'primary'}
                    />
                    {status.error && (
                      <Typography variant="caption" color="error" display="block">
                        Error: {status.error}
                      </Typography>
                    )}
                  </Box>
                ))}
              </CardContent>
            </Card>
          )}

          <Alert severity="warning">
            <Typography variant="body2">
              This will apply the selected configuration to all target tenants. 
              Make sure you have reviewed all settings before proceeding.
            </Typography>
          </Alert>
        </Stack>

        <Box sx={{ display: 'flex', justifyContent: 'space-between', mt: 3 }}>
          <Button onClick={onPreviousStep} disabled={isSubmitting}>
            Previous
          </Button>
          <Button
            variant="contained"
            onClick={handleDeploy}
            disabled={isSubmitting}
            startIcon={isSubmitting ? <CircularProgress size={20} /> : <RocketLaunchIcon />}
          >
            {isSubmitting ? 'Deploying...' : 'Deploy Configuration'}
          </Button>
        </Box>
      </CippWizardPage>
    );
  };

  const wizardSteps = [
    {
      label: 'Select Tenants',
      description: 'Choose target tenants',
      component: TenantSelectionStep,
      required: true,
    },
    {
      label: 'Setup Type',
      description: 'Configuration approach',
      component: SetupTypeStep,
      required: true,
    },
    {
      label: 'Security Policies',
      description: 'Password and MFA settings',
      component: SecurityPolicyStep,
      required: true,
    },
    {
      label: 'Conditional Access',
      description: 'Advanced access controls',
      component: ConditionalAccessStep,
      hideStepWhen: (data) => data.setupType === 'basic',
    },
    {
      label: 'Review & Deploy',
      description: 'Deploy configuration',
      component: ReviewAndDeployStep,
      required: true,
    },
  ];

  return (
    <Box sx={{ maxWidth: 1400, mx: 'auto', p: 3 }}>
      <Typography variant="h4" gutterBottom>
        Tenant Security Setup
      </Typography>
      
      <CippWizard
        steps={wizardSteps}
        orientation="horizontal"
        initialState={{
          targetTenants: [],
          selectAll: false,
          setupType: '',
          passwordPolicy: {
            minimumLength: 12,
            passwordHistory: 12,
            requireUppercase: true,
            requireLowercase: true,
            requireNumbers: true,
            requireSymbols: true,
          },
          mfaSettings: {
            enableMFA: true,
            allowedMethods: ['authenticator', 'phone'],
            rememberDeviceDays: 30,
          },
          conditionalAccess: {
            blockUntrustedLocations: true,
            requireMFAForAdmins: true,
            blockLegacyAuth: true,
            requireCompliantDevices: false,
          },
        }}
      />
    </Box>
  );
};
```