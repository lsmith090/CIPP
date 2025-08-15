# CippWizard

Multi-step wizard system for complex workflows and guided user experiences. The wizard system provides step management, validation, conditional steps, and form state persistence across steps.

## Components

### CippWizard
Main wizard container that manages step navigation and form state.

**Key Features:**
- Step-by-step navigation
- Form state persistence across steps
- Conditional step visibility
- Validation per step
- Horizontal and vertical layouts
- React Hook Form integration
- Automatic step completion tracking

**Props:**
- `steps` (array, required): Array of step configurations
- `postUrl` (string): API endpoint for final submission
- `orientation` (string): 'horizontal' or 'vertical' layout
- `initialState` (object): Initial form values
- `onComplete` (function): Callback when wizard completes
- `allowSkipSteps` (boolean): Allow users to skip steps
- `showStepNumbers` (boolean): Display step numbers

**Step Configuration:**
```jsx
{
  label: 'Step Name',
  description: 'Step description',
  component: StepComponent,
  componentProps: {
    // Props passed to step component
    title: 'Step Title',
    subtext: 'Additional information',
    options: [], // Step-specific options
  },
  validation: validationRules, // Step validation rules
  hideStepWhen: (formData) => boolean, // Conditional hiding
  showStepWhen: (formData) => boolean, // Conditional showing
  required: true, // Step is required
}
```

**Usage:**
```jsx
import { CippWizard } from '../CippWizard/CippWizard';

const SetupWizard = () => {
  const wizardSteps = [
    {
      label: 'Basic Information',
      description: 'Configure basic settings',
      component: BasicInfoStep,
      componentProps: {
        title: 'Organization Setup',
        subtext: 'Enter your organization details',
      },
      validation: basicInfoSchema,
      required: true,
    },
    {
      label: 'User Configuration',
      description: 'Set up user defaults',
      component: UserConfigStep,
      componentProps: {
        title: 'User Defaults',
      },
      validation: userConfigSchema,
    },
    {
      label: 'Security Settings',
      description: 'Configure security policies',
      component: SecurityStep,
      hideStepWhen: (data) => data.basicSetup === false,
      validation: securitySchema,
    },
    {
      label: 'Review & Complete',
      description: 'Review your configuration',
      component: ReviewStep,
      componentProps: {
        title: 'Review Configuration',
        subtext: 'Please review your settings before proceeding',
      },
    },
  ];

  return (
    <CippWizard
      steps={wizardSteps}
      postUrl="/api/CompleteSetup"
      orientation="horizontal"
      initialState={{
        organizationName: '',
        adminEmail: '',
        basicSetup: true,
        userDefaults: {},
        securitySettings: {},
      }}
      onComplete={(result) => {
        router.push('/dashboard');
        showNotification('Setup completed successfully!');
      }}
    />
  );
};
```

### CippWizardPage
Wrapper component for individual wizard pages with consistent layout.

**Props:**
- `title` (string): Page title
- `subtitle` (string): Page subtitle
- `children` (ReactNode): Page content
- `loading` (boolean): Loading state
- `error` (string): Error message
- `helpText` (string): Help information

**Usage:**
```jsx
import { CippWizardPage } from '../CippWizard/CippWizardPage';

const BasicInfoStep = ({ formControl, title, subtext }) => {
  return (
    <CippWizardPage
      title={title}
      subtitle={subtext}
      helpText="This information will be used to configure your organization settings"
    >
      <Grid container spacing={3}>
        <Grid item xs={12} md={6}>
          <CippFormComponent
            type="textField"
            name="organizationName"
            label="Organization Name"
            formControl={formControl}
            required
            fullWidth
          />
        </Grid>
        <Grid item xs={12} md={6}>
          <CippFormComponent
            type="emailField"
            name="adminEmail"
            label="Administrator Email"
            formControl={formControl}
            required
            fullWidth
          />
        </Grid>
      </Grid>
    </CippWizardPage>
  );
};
```

### CippWizardStepButtons
Navigation button component for wizard steps.

**Props:**
- `currentStep` (number): Current step index
- `lastStep` (number): Last step index
- `onPreviousStep` (function): Previous button handler
- `onNextStep` (function): Next button handler
- `onComplete` (function): Complete button handler
- `isValid` (boolean): Current step validation state
- `isSubmitting` (boolean): Submission state
- `nextLabel` (string): Next button text
- `previousLabel` (string): Previous button text
- `completeLabel` (string): Complete button text
- `hideNext` (boolean): Hide next button
- `hidePrevious` (boolean): Hide previous button
- `customButtons` (ReactNode): Additional buttons

**Usage:**
```jsx
import { CippWizardStepButtons } from '../CippWizard/CippWizardStepButtons';

const CustomStepComponent = ({ 
  currentStep, 
  lastStep, 
  onNextStep, 
  onPreviousStep,
  formControl 
}) => {
  const { isValid } = useFormState({ control: formControl.control });

  return (
    <CippWizardPage title="Custom Step">
      {/* Step content */}
      
      <CippWizardStepButtons
        currentStep={currentStep}
        lastStep={lastStep}
        onPreviousStep={onPreviousStep}
        onNextStep={onNextStep}
        isValid={isValid}
        nextLabel="Continue"
        customButtons={
          <Button variant="outlined" onClick={handleSaveDraft}>
            Save Draft
          </Button>
        }
      />
    </CippWizardPage>
  );
};
```

### CippWizardConfirmation
Final confirmation step component with review functionality.

**Props:**
- `title` (string): Confirmation title
- `subtitle` (string): Confirmation subtitle
- `data` (object): Form data to review
- `sections` (array): Review sections configuration
- `onEdit` (function): Edit handler for sections
- `submitLabel` (string): Submit button text
- `isSubmitting` (boolean): Submission state

**Section Configuration:**
```jsx
{
  title: 'Basic Information',
  fields: [
    { key: 'organizationName', label: 'Organization Name' },
    { key: 'adminEmail', label: 'Administrator Email' },
  ],
  editStep: 0, // Step index to edit
}
```

**Usage:**
```jsx
import { CippWizardConfirmation } from '../CippWizard/CippWizardConfirmation';

const ReviewStep = ({ formControl, onPreviousStep, currentStep }) => {
  const formData = formControl.getValues();

  const reviewSections = [
    {
      title: 'Organization Details',
      fields: [
        { key: 'organizationName', label: 'Organization Name' },
        { key: 'adminEmail', label: 'Administrator Email' },
        { key: 'domain', label: 'Primary Domain' },
      ],
      editStep: 0,
    },
    {
      title: 'User Configuration',
      fields: [
        { key: 'defaultLicense', label: 'Default License' },
        { key: 'defaultGroups', label: 'Default Groups' },
      ],
      editStep: 1,
    },
    {
      title: 'Security Settings',
      fields: [
        { key: 'mfaRequired', label: 'MFA Required' },
        { key: 'passwordPolicy', label: 'Password Policy' },
      ],
      editStep: 2,
    },
  ];

  const handleEdit = (stepIndex) => {
    // Navigate to specific step for editing
    setActiveStep(stepIndex);
  };

  return (
    <CippWizardConfirmation
      title="Review Configuration"
      subtitle="Please review your settings before completing setup"
      data={formData}
      sections={reviewSections}
      onEdit={handleEdit}
      submitLabel="Complete Setup"
      isSubmitting={isSubmitting}
    />
  );
};
```

## Wizard Patterns

### Linear Wizard
```jsx
const LinearWizard = () => {
  const steps = [
    { label: 'Step 1', component: Step1Component },
    { label: 'Step 2', component: Step2Component },
    { label: 'Step 3', component: Step3Component },
  ];

  return (
    <CippWizard
      steps={steps}
      orientation="horizontal"
      postUrl="/api/complete"
    />
  );
};
```

### Conditional Wizard
```jsx
const ConditionalWizard = () => {
  const steps = [
    {
      label: 'Setup Type',
      component: SetupTypeStep,
      required: true,
    },
    {
      label: 'Basic Setup',
      component: BasicSetupStep,
      showStepWhen: (data) => data.setupType === 'basic',
    },
    {
      label: 'Advanced Setup',
      component: AdvancedSetupStep,
      showStepWhen: (data) => data.setupType === 'advanced',
    },
    {
      label: 'Enterprise Features',
      component: EnterpriseStep,
      showStepWhen: (data) => data.setupType === 'advanced' && data.licenseType === 'enterprise',
    },
    {
      label: 'Review',
      component: ReviewStep,
      required: true,
    },
  ];

  return <CippWizard steps={steps} />;
};
```

### Validation per Step
```jsx
// Validation rules for each step
const step1ValidationRules = {
  organizationName: { required: 'Organization name is required' },
  adminEmail: { 
    required: 'Email is required',
    pattern: {
      value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      message: 'Invalid email format'
    }
  }
};

const step2ValidationRules = {
  userCount: { 
    required: 'User count is required',
    min: { value: 1, message: 'At least 1 user required' }
  },
  licenseType: { required: 'License type is required' }
};

const ValidatedWizard = () => {
  const steps = [
    {
      label: 'Organization',
      component: OrganizationStep,
      validation: step1ValidationRules,
    },
    {
      label: 'Licensing',
      component: LicensingStep,
      validation: step2ValidationRules,
    },
  ];

  return <CippWizard steps={steps} />;
};
```

### Custom Step Component
```jsx
const CustomStep = ({
  formControl,
  onNextStep,
  onPreviousStep,
  currentStep,
  lastStep,
  title,
  options,
}) => {
  const [loading, setLoading] = useState(false);
  const { isValid } = useFormState({ control: formControl.control });

  const handleNext = async () => {
    if (!isValid) return;
    
    setLoading(true);
    try {
      // Custom validation or processing
      await validateStepData(formControl.getValues());
      onNextStep();
    } catch (error) {
      showError(error.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <CippWizardPage
      title={title}
      loading={loading}
    >
      <Grid container spacing={3}>
        <Grid item xs={12}>
          <CippFormComponent
            type="select"
            name="selectedOption"
            label="Choose Option"
            formControl={formControl}
            options={options}
            required
          />
        </Grid>
        
        {/* Dynamic content based on selection */}
        <Grid item xs={12}>
          <ConditionalContent formControl={formControl} />
        </Grid>
      </Grid>

      <CippWizardStepButtons
        currentStep={currentStep}
        lastStep={lastStep}
        onPreviousStep={onPreviousStep}
        onNextStep={handleNext}
        isValid={isValid && !loading}
        isSubmitting={loading}
      />
    </CippWizardPage>
  );
};
```

## Advanced Features

### Save and Resume
```jsx
const ResumableWizard = () => {
  const [draftId, setDraftId] = useState(null);

  const saveDraft = async (formData) => {
    try {
      const response = await fetch('/api/SaveWizardDraft', {
        method: 'POST',
        body: JSON.stringify({ data: formData, draftId }),
      });
      const result = await response.json();
      setDraftId(result.draftId);
      showNotification('Draft saved successfully');
    } catch (error) {
      showError('Failed to save draft');
    }
  };

  const loadDraft = async (id) => {
    try {
      const response = await fetch(`/api/GetWizardDraft/${id}`);
      const draft = await response.json();
      return draft.data;
    } catch (error) {
      showError('Failed to load draft');
      return null;
    }
  };

  return (
    <CippWizard
      steps={steps}
      onStepChange={(stepIndex, formData) => {
        // Auto-save on step change
        saveDraft(formData);
      }}
      initialState={draftData}
    />
  );
};
```

### Progress Tracking
```jsx
const ProgressTrackedWizard = () => {
  const [completedSteps, setCompletedSteps] = useState(new Set());
  const [currentStep, setCurrentStep] = useState(0);

  const calculateProgress = () => {
    return (completedSteps.size / steps.length) * 100;
  };

  return (
    <Card>
      <CardHeader
        title="Setup Wizard"
        subheader={
          <Box sx={{ mt: 2 }}>
            <LinearProgress 
              variant="determinate" 
              value={calculateProgress()} 
            />
            <Typography variant="caption" sx={{ mt: 1, display: 'block' }}>
              {completedSteps.size} of {steps.length} steps completed
            </Typography>
          </Box>
        }
      />
      <CardContent>
        <CippWizard
          steps={steps}
          onStepComplete={(stepIndex) => {
            setCompletedSteps(prev => new Set([...prev, stepIndex]));
          }}
          onStepChange={setCurrentStep}
        />
      </CardContent>
    </Card>
  );
};
```

### Multi-Tenant Wizard
```jsx
const MultiTenantWizard = () => {
  const [selectedTenants, setSelectedTenants] = useState([]);

  const steps = [
    {
      label: 'Select Tenants',
      component: TenantSelectionStep,
      componentProps: {
        onSelectionChange: setSelectedTenants,
      },
    },
    {
      label: 'Configuration',
      component: ConfigurationStep,
      componentProps: {
        tenants: selectedTenants,
      },
      showStepWhen: (data) => selectedTenants.length > 0,
    },
    {
      label: 'Apply Settings',
      component: ApplyStep,
      componentProps: {
        tenants: selectedTenants,
      },
    },
  ];

  const handleComplete = async (formData) => {
    // Apply configuration to selected tenants
    const promises = selectedTenants.map(tenant =>
      fetch('/api/ApplyConfiguration', {
        method: 'POST',
        body: JSON.stringify({
          tenant: tenant.id,
          configuration: formData,
        }),
      })
    );

    await Promise.all(promises);
    showNotification('Configuration applied to all tenants');
  };

  return (
    <CippWizard
      steps={steps}
      onComplete={handleComplete}
    />
  );
};
```

## Best Practices

1. **Step Organization**: Keep steps focused and logically grouped
2. **Validation**: Validate each step before allowing progression
3. **Error Handling**: Provide clear error messages and recovery options
4. **Progress Indication**: Show users where they are in the process
5. **Data Persistence**: Save progress to prevent data loss
6. **Conditional Logic**: Hide irrelevant steps based on user choices
7. **Mobile Responsiveness**: Ensure wizards work on all screen sizes
8. **Accessibility**: Include proper ARIA labels and keyboard navigation
9. **Performance**: Load step data lazily when possible
10. **User Experience**: Allow users to go back and modify previous steps