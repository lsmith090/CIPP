# Wizard Pages - CippWizard Pattern

Wizard pages provide guided, multi-step processes for complex operations in CIPP. They break down complicated workflows into manageable steps with clear navigation, progress tracking, and state management across the entire process.

## Core Components

The wizard system consists of two main components:

**Primary Components**:
- `/src/components/CippWizard/CippWizardPage.jsx` - Page wrapper for wizards
- `/src/components/CippWizard/CippWizard.jsx` - Core wizard component

## Basic Usage

```javascript
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import CippWizardPage from "../../../components/CippWizard/CippWizardPage.jsx";
import { CippTenantStep } from "../../../components/CippWizard/CippTenantStep.jsx";
import { CippWizardConfirmation } from "/src/components/CippWizard/CippWizardConfirmation";

const Page = () => {
  const steps = [
    {
      title: "Step 1",
      description: "Select Tenant",
      component: CippTenantStep,
      componentProps: {
        allTenants: false,
        type: "single"
      }
    },
    {
      title: "Step 2", 
      description: "Confirmation",
      component: CippWizardConfirmation
    }
  ];

  return (
    <CippWizardPage
      initialState={{}}
      steps={steps}
      postUrl="/api/ExecuteAction"
      wizardTitle="My Wizard"
    />
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default Page;
```

## CippWizardPage Props

### Required Props

| Prop | Type | Description |
|------|------|-------------|
| `steps` | `array` | Array of step definitions |
| `postUrl` | `string` | API endpoint for final submission |
| `wizardTitle` | `string` | Title displayed in header |

### Optional Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `initialState` | `object` | `{}` | Initial form state |
| `backButton` | `boolean` | `true` | Show back button in header |
| `wizardOrientation` | `string` | `"horizontal"` | Step orientation |

## Step Definition Structure

Each step in the `steps` array follows this structure:

```javascript
{
  title: "Step 1",              // Short step title
  description: "Description",    // Longer description
  component: ComponentName,      // React component to render
  componentProps: {              // Props passed to component
    // Component-specific props
  }
}
```

## Real-World Example: Offboarding Wizard

Here's the actual implementation from `/src/pages/identity/administration/offboarding-wizard/index.js`:

```javascript
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { CippWizardConfirmation } from "/src/components/CippWizard/CippWizardConfirmation";
import CippWizardPage from "../../../components/CippWizard/CippWizardPage.jsx";
import { CippTenantStep } from "../../../components/CippWizard/CippTenantStep.jsx";
import { CippWizardAutoComplete } from "../../../../components/CippWizard/CippWizardAutoComplete";
import { CippWizardOffboarding } from "../../../../components/CippWizard/CippWizardOffboarding";
import { useSettings } from "../../../../hooks/use-settings";

const Page = () => {
  const initialState = useSettings();
  
  const steps = [
    {
      title: "Step 1",
      description: "Tenant Selection",
      component: CippTenantStep,
      componentProps: {
        allTenants: false,
        type: "single",
      },
    },
    {
      title: "Step 2",
      description: "User Selection",
      component: CippWizardAutoComplete,
      componentProps: {
        title: "Select the users to offboard",
        name: "user",
        placeholder: "Select User",
        type: "multiple",
        api: {
          url: "/api/ListGraphRequest",
          dataKey: "Results",
          queryKey: "Users - {tenant}",
          data: {
            Endpoint: "users",
            manualPagination: true,
            $select: "id,userPrincipalName,displayName",
            $count: true,
            $orderby: "displayName",
            $top: 999,
          },
          labelField: (option) => `${option.displayName} (${option.userPrincipalName})`,
          valueField: "userPrincipalName",
        },
      },
    },
    {
      title: "Step 3",
      description: "Offboarding Options",
      component: CippWizardOffboarding,
    },
    {
      title: "Step 4",
      description: "Confirmation",
      component: CippWizardConfirmation,
    },
  ];

  return (
    <>
      <CippWizardPage
        initialState={{ 
          ...initialState.offboardingDefaults, 
          ...{ Scheduled: { enabled: false } } 
        }}
        steps={steps}
        postUrl="/api/ExecOffboardUser"
        wizardTitle="User Offboarding Wizard"
      />
    </>
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default Page;
```

## Built-in Step Components

### CippTenantStep
Tenant selection step for multi-tenant operations.

```javascript
{
  title: "Tenant Selection",
  description: "Choose target tenant",
  component: CippTenantStep,
  componentProps: {
    allTenants: false,      // Allow "All Tenants" option
    type: "single"          // "single" or "multiple"
  }
}
```

### CippWizardAutoComplete
Dynamic autocomplete selection with API integration.

```javascript
{
  title: "User Selection",
  description: "Select users",
  component: CippWizardAutoComplete,
  componentProps: {
    title: "Select users to process",
    name: "selectedUsers",
    placeholder: "Search users...",
    type: "multiple",       // "single" or "multiple"
    api: {
      url: "/api/ListGraphRequest",
      dataKey: "Results",
      queryKey: "Users - {tenant}",
      data: {
        Endpoint: "users",
        $select: "id,displayName,userPrincipalName"
      },
      labelField: (option) => option.displayName,
      valueField: "id"
    }
  }
}
```

### CippWizardConfirmation
Final confirmation step with summary display.

```javascript
{
  title: "Confirmation",
  description: "Review and confirm",
  component: CippWizardConfirmation
}
```

### CippWizardBulkOptions
Bulk operation configuration step.

```javascript
{
  title: "Bulk Options",
  description: "Configure bulk operation",
  component: CippWizardBulkOptions,
  componentProps: {
    options: [
      { name: "removeFromGroups", label: "Remove from all groups" },
      { name: "revokeSignIn", label: "Revoke all sign-in sessions" },
      { name: "resetPassword", label: "Reset password" }
    ]
  }
}
```

## Custom Step Components

### Creating Custom Steps

Custom step components receive these props:

```javascript
const CustomStep = ({ 
  formState,        // Current wizard form state
  updateFormState,  // Function to update state
  onNext,          // Function to proceed to next step
  onPrevious,      // Function to go to previous step
  isLast,          // Boolean if this is the last step
  isFirst,         // Boolean if this is the first step
  ...componentProps // Props from step definition
}) => {
  const handleChange = (field, value) => {
    updateFormState({
      [field]: value
    });
  };

  return (
    <Box>
      <Typography variant="h6">Custom Step</Typography>
      <TextField
        label="Custom Field"
        value={formState.customField || ""}
        onChange={(e) => handleChange("customField", e.target.value)}
      />
    </Box>
  );
};
```

### Validation in Custom Steps

```javascript
const ValidatedStep = ({ formState, updateFormState, onNext }) => {
  const [errors, setErrors] = useState({});

  const handleNext = () => {
    const newErrors = {};
    
    if (!formState.requiredField) {
      newErrors.requiredField = "This field is required";
    }
    
    if (Object.keys(newErrors).length === 0) {
      onNext();
    } else {
      setErrors(newErrors);
    }
  };

  return (
    <Box>
      <TextField
        label="Required Field"
        value={formState.requiredField || ""}
        onChange={(e) => updateFormState({ requiredField: e.target.value })}
        error={!!errors.requiredField}
        helperText={errors.requiredField}
      />
      <Button onClick={handleNext}>Next</Button>
    </Box>
  );
};
```

### Conditional Steps

```javascript
const ConditionalStep = ({ formState, updateFormState }) => {
  const showAdvanced = formState.enableAdvanced;

  return (
    <Box>
      <FormControlLabel
        control={
          <Checkbox
            checked={showAdvanced || false}
            onChange={(e) => updateFormState({ enableAdvanced: e.target.checked })}
          />
        }
        label="Enable Advanced Options"
      />
      
      {showAdvanced && (
        <Box sx={{ mt: 2 }}>
          <TextField
            label="Advanced Setting"
            value={formState.advancedSetting || ""}
            onChange={(e) => updateFormState({ advancedSetting: e.target.value })}
          />
        </Box>
      )}
    </Box>
  );
};
```

## Form State Management

### Initial State

```javascript
const initialState = {
  tenant: "AllTenants",
  users: [],
  options: {
    removeFromGroups: true,
    revokeSignIn: true
  }
};

<CippWizardPage
  initialState={initialState}
  // ... other props
/>
```

### State Updates

```javascript
// In a custom step component
const handleUserSelection = (selectedUsers) => {
  updateFormState({
    users: selectedUsers,
    userCount: selectedUsers.length
  });
};
```

### Persistent State

The wizard automatically persists state across steps. State is maintained until:
- Wizard is completed
- User navigates away
- Browser session ends

## Navigation Patterns

### Step Validation

```javascript
const ValidatedStep = ({ onNext, formState }) => {
  const canProceed = formState.requiredField && formState.requiredField.length > 0;

  return (
    <Box>
      {/* Step content */}
      <Button 
        onClick={onNext} 
        disabled={!canProceed}
      >
        Next
      </Button>
    </Box>
  );
};
```

### Skip Steps

```javascript
const ConditionalNavigationStep = ({ onNext, formState, skipToStep }) => {
  const handleSkipToEnd = () => {
    skipToStep(3); // Skip to step index 3
  };

  return (
    <Box>
      <Button onClick={onNext}>Next Step</Button>
      <Button onClick={handleSkipToEnd}>Skip to Confirmation</Button>
    </Box>
  );
};
```

### Back Button Control

```javascript
const NoBackStep = ({ onNext, disableBack }) => {
  useEffect(() => {
    disableBack(true);
    return () => disableBack(false);
  }, []);

  return (
    <Box>
      {/* Step content - back button disabled */}
    </Box>
  );
};
```

## API Integration

### Dynamic Step Data

```javascript
const ApiDrivenStep = ({ formState, updateFormState }) => {
  const { data: options } = ApiGetCall({
    url: "/api/GetStepOptions",
    data: { tenant: formState.tenant },
    queryKey: `StepOptions-${formState.tenant}`
  });

  return (
    <Box>
      {options?.map(option => (
        <FormControlLabel
          key={option.id}
          control={
            <Checkbox
              checked={formState.selectedOptions?.includes(option.id)}
              onChange={(e) => {
                const current = formState.selectedOptions || [];
                const updated = e.target.checked
                  ? [...current, option.id]
                  : current.filter(id => id !== option.id);
                updateFormState({ selectedOptions: updated });
              }}
            />
          }
          label={option.label}
        />
      ))}
    </Box>
  );
};
```

### Tenant-Dependent Steps

```javascript
const TenantAwareStep = ({ formState }) => {
  const { data: tenantInfo } = ApiGetCall({
    url: "/api/GetTenantInfo",
    data: { tenantFilter: formState.tenant },
    queryKey: `TenantInfo-${formState.tenant}`,
    waiting: !formState.tenant
  });

  if (!tenantInfo) {
    return <CircularProgress />;
  }

  return (
    <Box>
      <Typography>
        Configuring for: {tenantInfo.displayName}
      </Typography>
      {/* Tenant-specific options */}
    </Box>
  );
};
```

## Data Flow

1. **Initialization**: Wizard starts with `initialState`
2. **Step Navigation**: User proceeds through steps sequentially
3. **State Updates**: Each step can modify the global form state
4. **Validation**: Steps can validate before allowing progression
5. **Confirmation**: Final step shows summary of all selections
6. **Submission**: Complete state sent to `postUrl` endpoint
7. **Result Handling**: Success/error feedback displayed

## Advanced Patterns

### Branching Workflows

```javascript
const BranchingStep = ({ formState, skipToStep }) => {
  const handleChoice = (choice) => {
    updateFormState({ workflowType: choice });
    
    if (choice === "simple") {
      skipToStep(4); // Skip advanced configuration
    } else {
      onNext(); // Continue to advanced steps
    }
  };

  return (
    <Box>
      <Button onClick={() => handleChoice("simple")}>
        Simple Setup
      </Button>
      <Button onClick={() => handleChoice("advanced")}>
        Advanced Setup
      </Button>
    </Box>
  );
};
```

### Progress Tracking

```javascript
const ProgressStep = ({ currentStep, totalSteps, formState }) => {
  const progress = (currentStep / totalSteps) * 100;

  return (
    <Box>
      <LinearProgress variant="determinate" value={progress} />
      <Typography>
        Step {currentStep} of {totalSteps}
      </Typography>
      {/* Step content */}
    </Box>
  );
};
```

### Summary Generation

```javascript
const SummaryStep = ({ formState }) => {
  const summary = useMemo(() => {
    return {
      "Selected Tenant": formState.tenant,
      "Users Count": formState.users?.length || 0,
      "Actions": Object.keys(formState.options || {})
        .filter(key => formState.options[key])
        .join(", ")
    };
  }, [formState]);

  return (
    <Box>
      <Typography variant="h6">Summary</Typography>
      {Object.entries(summary).map(([key, value]) => (
        <Box key={key} sx={{ display: "flex", justifyContent: "space-between" }}>
          <Typography>{key}:</Typography>
          <Typography fontWeight="bold">{value}</Typography>
        </Box>
      ))}
    </Box>
  );
};
```

## Performance Considerations

### Lazy Loading Steps

```javascript
import dynamic from "next/dynamic";

const HeavyStep = dynamic(
  () => import("./HeavyStepComponent"),
  { ssr: false }
);

const steps = [
  {
    title: "Heavy Step",
    description: "Resource intensive step",
    component: HeavyStep
  }
];
```

### Memoized Calculations

```javascript
const CalculationStep = ({ formState }) => {
  const expensiveCalculation = useMemo(() => {
    return performExpensiveOperation(formState.data);
  }, [formState.data]);

  return (
    <Box>
      <Typography>Result: {expensiveCalculation}</Typography>
    </Box>
  );
};
```

### Debounced Updates

```javascript
import { useDebounce } from "/src/hooks/use-debounce";

const SearchStep = ({ formState, updateFormState }) => {
  const [searchTerm, setSearchTerm] = useState("");
  const debouncedSearch = useDebounce(searchTerm, 300);

  useEffect(() => {
    if (debouncedSearch) {
      updateFormState({ searchResults: performSearch(debouncedSearch) });
    }
  }, [debouncedSearch]);

  return (
    <TextField
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
      placeholder="Search..."
    />
  );
};
```

## Common Use Cases

### User Onboarding

```javascript
const onboardingSteps = [
  {
    title: "Welcome",
    description: "Welcome to the onboarding process",
    component: WelcomeStep
  },
  {
    title: "Tenant Selection",
    description: "Select target tenant",
    component: CippTenantStep,
    componentProps: { allTenants: false, type: "single" }
  },
  {
    title: "User Information",
    description: "Enter user details",
    component: UserInfoStep
  },
  {
    title: "License Assignment",
    description: "Assign licenses",
    component: LicenseStep
  },
  {
    title: "Group Membership",
    description: "Add to groups",
    component: GroupStep
  },
  {
    title: "Confirmation",
    description: "Review and confirm",
    component: CippWizardConfirmation
  }
];
```

### Bulk Operations

```javascript
const bulkOperationSteps = [
  {
    title: "Tenant Selection",
    description: "Choose tenant",
    component: CippTenantStep
  },
  {
    title: "User Selection",
    description: "Select users for bulk operation",
    component: CippWizardAutoComplete,
    componentProps: {
      type: "multiple",
      api: { /* user API config */ }
    }
  },
  {
    title: "Operation Options",
    description: "Configure bulk operation",
    component: CippWizardBulkOptions
  },
  {
    title: "Confirmation",
    description: "Review selections",
    component: CippWizardConfirmation
  }
];
```

### Configuration Wizards

```javascript
const configurationSteps = [
  {
    title: "Basic Settings",
    description: "Configure basic options",
    component: BasicConfigStep
  },
  {
    title: "Advanced Settings",
    description: "Advanced configuration",
    component: AdvancedConfigStep
  },
  {
    title: "Integration Setup",
    description: "Configure integrations",
    component: IntegrationStep
  },
  {
    title: "Test Configuration",
    description: "Test the setup",
    component: TestStep
  },
  {
    title: "Deploy",
    description: "Deploy configuration",
    component: DeployStep
  }
];
```

## Best Practices

1. **Clear Steps**: Each step should have a single, clear purpose
2. **Progressive Disclosure**: Show only relevant information at each step
3. **Validation**: Validate inputs before allowing progression
4. **Error Handling**: Provide clear error messages and recovery options
5. **Progress Indication**: Show users where they are in the process
6. **State Persistence**: Maintain state across browser refreshes when possible
7. **Mobile Optimization**: Ensure wizards work well on small screens
8. **Accessibility**: Provide proper navigation for screen readers
9. **Performance**: Lazy load heavy components and use proper memoization
10. **User Experience**: Allow users to go back and modify previous selections

## Troubleshooting

### Common Issues

**State Not Persisting**
- Check that `updateFormState` is being called correctly
- Verify state structure matches expected format
- Ensure no accidental state overwrites

**Navigation Not Working**
- Verify step components are using `onNext`/`onPrevious` correctly
- Check for validation errors preventing progression
- Ensure step array is properly structured

**API Calls Failing**
- Check network tab for request details
- Verify tenant selection is properly set
- Ensure API endpoints are accessible

**Performance Issues**
- Implement lazy loading for heavy components
- Use memoization for expensive calculations
- Check for unnecessary re-renders in step components

## Related Components

- [`CippWizard`](../components/cipp-wizard/README.md) - Core wizard component
- [`CippTenantStep`](../components/cipp-wizard/README.md) - Tenant selection step
- [`CippWizardAutoComplete`](../components/cipp-wizard/README.md) - Autocomplete selection step
- [`CippWizardConfirmation`](../components/cipp-wizard/README.md) - Final confirmation step