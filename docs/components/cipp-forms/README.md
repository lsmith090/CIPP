# CippForms

Form components built with React Hook Form integration, providing validation, submission handling, and consistent styling across the application. All form components support tenant-aware operations and permission-based field visibility.

> **Architecture Reference**: See [Component Architecture - CippFormPages](../../architecture/components.md#3-cippformpages---form-management) for architectural patterns and design principles.

## Components

### CippFormPage
Complete form page layout with validation, submission, and error handling.

**Key Features:**
- React Hook Form integration
- Automatic form validation
- API submission with loading states
- Success/error message display
- Back navigation support
- Query parameter pre-population
- Form reset capabilities
- Custom data formatting

**Props:**
- `title` (string): Page title
- `formPageType` (string): Form type ("Add", "Edit", "Configure")
- `formControl` (object, required): React Hook Form control object
- `postUrl` (string, required): API endpoint for form submission
- `queryKey` (string/array): React Query cache key to invalidate
- `customDataformatter` (function): Transform form data before submission
- `backButtonTitle` (string): Back button text
- `titleButton` (ReactNode): Button in page header
- `resetForm` (boolean): Reset form after successful submission
- `hideBackButton` (boolean): Hide back navigation
- `hidePageType` (boolean): Hide form type prefix
- `hideTitle` (boolean): Hide page title
- `hideSubmit` (boolean): Hide submit button
- `allowResubmit` (boolean): Allow resubmission without changes
- `addedButtons` (ReactNode): Additional action buttons

**Usage:**
```jsx
import CippFormPage from '/src/components/CippFormPages/CippFormPage.jsx';
import { useForm } from 'react-hook-form';

const AddUserForm = () => {
  const formControl = useForm({
    mode: 'onChange',
    defaultValues: {
      displayName: '',
      mail: '',
      department: '',
      accountEnabled: true,
    }
  });

  const customDataFormatter = (data) => {
    return {
      ...data,
      userPrincipalName: `${data.mail}@${currentTenant}`,
      mailNickname: data.mail.split('@')[0],
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
      <Grid container spacing={3}>
        <Grid size={{ xs: 12, md: 6 }}>
          <CippFormComponent
            type="textField"
            label="Display Name"
            name="displayName"
            formControl={formControl}
            validators={{ required: "Display Name is required" }}
          />
        </Grid>
        <Grid size={{ xs: 12, md: 6 }}>
          <CippFormComponent
            type="textField"
            label="Email"
            name="mail"
            formControl={formControl}
            validators={{ 
              required: "Email is required",
              pattern: {
                value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
                message: "Invalid email format"
              }
            }}
          />
        </Grid>
        <Grid size={{ xs: 12, md: 6 }}>
          <CippFormComponent
            type="textField"
            label="Department"
            name="department"
            formControl={formControl}
          />
        </Grid>
        <Grid size={{ xs: 12, md: 6 }}>
          <CippFormComponent
            type="switch"
            label="Account Enabled"
            name="accountEnabled"
            formControl={formControl}
          />
        </Grid>
      </Grid>
    </CippFormPage>
  );
};
```

### CippFormSection
Collapsible form sections for organizing related fields.

**Props:**
- `title` (string, required): Section title
- `description` (string): Section description
- `defaultExpanded` (boolean): Initially expanded state
- `required` (boolean): Show required indicator
- `children` (ReactNode): Form fields content

**Usage:**
```jsx
import { CippFormSection } from '/src/components/CippFormPages/CippFormSection.jsx';

<CippFormSection
  title="Basic Information"
  description="Core user details and contact information"
  defaultExpanded={true}
  required={true}
>
  <Grid container spacing={2}>
    <Grid size={{ xs: 12, md: 6 }}>
      <CippFormComponent
        type="textField"
        label="First Name"
        name="givenName"
        formControl={formControl}
        required
      />
    </Grid>
    <Grid size={{ xs: 12, md: 6 }}>
      <CippFormComponent
        type="textField"
        label="Last Name"
        name="surname"
        formControl={formControl}
        required
      />
    </Grid>
  </Grid>
</CippFormSection>
```

### CippFormComponent
Universal form field component supporting multiple input types.

**Supported Types:**
- `textField`: Standard text input
- `password`: Password input field
- `number`: Numeric input field
- `select`: Dropdown selection
- `autoComplete`: Autocomplete with search
- `switch`: Boolean toggle switch
- `checkbox`: Checkbox input
- `radio`: Radio button group
- `datePicker`: Date selection
- `file`: File upload input
- `richText`: Rich text editor with formatting
- `CSVReader`: CSV file upload and preview
- `cippDataTable`: Embedded data table component
- `hidden`: Hidden form field

**Common Props:**
- `type` (string, required): Field type
- `name` (string, required): Field name for form state
- `label` (string): Field label
- `formControl` (object, required): React Hook Form control
- `validators` (object): Form validation rules
- `defaultValue` (any): Default field value
- `helperText` (string): Help text below field
- `labelLocation` (string): Label position for switches ('above', 'behind') - default: 'behind'

**Type-Specific Props:**

**Select/AutoComplete:**
- `options` (array): Option values
- `multiple` (boolean): Multi-selection
- `loading` (boolean): Loading state
- `apiUrl` (string): Dynamic options from API
- `optionKey` (string): Value property name
- `optionLabel` (string): Display property name

**TextField:**
- `multiline` (boolean): Multi-line input
- `rows` (number): Number of rows for multiline
- `maxLength` (number): Character limit

**DatePicker:**
- `disablePast` (boolean): Disable past dates
- `disableFuture` (boolean): Disable future dates
- `format` (string): Date format

**Usage Examples:**
```jsx
import { CippFormComponent } from '/src/components/CippComponents/CippFormComponent.jsx';

// Text field
<CippFormComponent
  type="textField"
  name="displayName"
  label="Display Name"
  formControl={formControl}
  validators={{ required: "Display Name is required" }}
  helperText="Enter the user's full name"
/>

// Select field
<CippFormComponent
  type="select"
  name="department"
  label="Department"
  formControl={formControl}
  options={[
    { value: 'IT', label: 'Information Technology' },
    { value: 'HR', label: 'Human Resources' },
    { value: 'Finance', label: 'Finance' },
  ]}
/>

// Autocomplete with API data
<CippFormComponent
  type="autoComplete"
  name="manager"
  label="Manager"
  formControl={formControl}
  apiUrl="/api/ListUsers"
  optionKey="id"
  optionLabel="displayName"
  loading={isLoadingUsers}
/>

// Switch toggle
<CippFormComponent
  type="switch"
  name="accountEnabled"
  label="Account Enabled"
  formControl={formControl}
/>

// Date picker
<CippFormComponent
  type="datePicker"
  name="startDate"
  label="Start Date"
  formControl={formControl}
  disablePast={true}
/>
```

### CippFormTenantSelector
Tenant selection component for multi-tenant forms.

**Props:**
- `name` (string): Form field name
- `label` (string): Field label
- `formControl` (object): React Hook Form control
- `required` (boolean): Required validation
- `multiple` (boolean): Multiple tenant selection
- `excludeCurrentTenant` (boolean): Exclude current tenant

**Usage:**
```jsx
import { CippFormTenantSelector } from '/src/components/CippComponents/CippFormTenantSelector.jsx';

<CippFormTenantSelector
  name="targetTenant"
  label="Target Tenant"
  formControl={formControl}
  required
/>
```

### CippFormUserSelector
User selection component with search and filtering.

**Props:**
- `name` (string): Form field name
- `label` (string): Field label
- `formControl` (object): React Hook Form control
- `multiple` (boolean): Multiple user selection
- `userType` (string): Filter by user type ('all', 'users', 'guests')
- `includeGroups` (boolean): Include security groups

**Usage:**
```jsx
import { CippFormUserSelector } from '/src/components/CippComponents/CippFormUserSelector.jsx';

<CippFormUserSelector
  name="assignedUsers"
  label="Assigned Users"
  formControl={formControl}
  multiple={true}
  userType="users"
/>
```

### CippFormGroupSelector
Group selection component with filtering options.

**Props:**
- `name` (string): Form field name
- `label` (string): Field label
- `formControl` (object): React Hook Form control
- `multiple` (boolean): Multiple group selection
- `groupTypes` (array): Filter by group types
- `includeDistributionGroups` (boolean): Include distribution groups

**Usage:**
```jsx
import { CippFormGroupSelector } from '/src/components/CippComponents/CippFormGroupSelector.jsx';

<CippFormGroupSelector
  name="memberOf"
  label="Group Membership"
  formControl={formControl}
  multiple={true}
  groupTypes={['security', 'office365']}
/>
```

## Form Patterns

### Basic Form Structure
```jsx
const MyForm = () => {
  const formControl = useForm({
    resolver: yupResolver(validationSchema),
    defaultValues: initialValues,
  });

  return (
    <CippFormPage
      title="My Form"
      formControl={formControl}
      postUrl="/api/endpoint"
    >
      <CippFormSection title="Section 1">
        {/* Form fields */}
      </CippFormSection>
    </CippFormPage>
  );
};
```

### Conditional Fields
```jsx
const ConditionalForm = () => {
  const formControl = useForm();
  const watchAccountType = formControl.watch('accountType');

  return (
    <CippFormPage formControl={formControl} {...props}>
      <CippFormComponent
        type="select"
        name="accountType"
        label="Account Type"
        formControl={formControl}
        options={accountTypes}
      />
      
      {watchAccountType === 'service' && (
        <CippFormComponent
          type="textField"
          name="applicationId"
          label="Application ID"
          formControl={formControl}
        />
      )}
    </CippFormPage>
  );
};
```

### Dynamic Field Arrays
```jsx
import { useFieldArray } from 'react-hook-form';

const DynamicFieldsForm = () => {
  const formControl = useForm();
  const { fields, append, remove } = useFieldArray({
    control: formControl.control,
    name: 'licenses',
  });

  return (
    <CippFormPage formControl={formControl} {...props}>
      {fields.map((field, index) => (
        <Grid container spacing={2} key={field.id}>
          <Grid size={{ xs: 10 }}>
            <CippFormComponent
              type="select"
              name={`licenses.${index}.skuId`}
              label="License"
              formControl={formControl}
              options={licenseOptions}
            />
          </Grid>
          <Grid size={{ xs: 2 }}>
            <IconButton onClick={() => remove(index)}>
              <DeleteIcon />
            </IconButton>
          </Grid>
        </Grid>
      ))}
      
      <Button onClick={() => append({ skuId: '' })}>
        Add License
      </Button>
    </CippFormPage>
  );
};
```

### Validation Patterns
```jsx
// Simple required validation
validators={{ required: "Display name is required" }}

// Complex validation with multiple rules
validators={{
  required: "Email is required",
  pattern: {
    value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    message: "Invalid email format"
  }
}}

// Multiple validation rules
validators={{
  required: { value: true, message: "Display name is required" },
  minLength: { value: 2, message: "Must be at least 2 characters" },
  maxLength: { value: 255, message: "Cannot exceed 255 characters" }
}}

// Custom validation function
validators={{
  required: "UPN is required",
  validate: {
    validUPN: (value) => 
      /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/.test(value) || "Invalid UPN format"
  }
}}
```

## Advanced Features

### Custom Validation
```jsx
const validateUniqueEmail = async (email) => {
  const response = await fetch(`/api/ValidateEmail?email=${email}`);
  const result = await response.json();
  return result.isUnique || 'Email already exists';
};

<CippFormComponent
  type="emailField"
  name="mail"
  label="Email"
  formControl={formControl}
  validators={{
    required: "Email is required",
    validate: {
      uniqueEmail: validateUniqueEmail
    }
  }}
/>
```

### File Upload with Progress
```jsx
const FileUploadForm = () => {
  const [uploadProgress, setUploadProgress] = useState(0);

  return (
    <CippFormComponent
      type="filePicker"
      name="csvFile"
      label="Upload CSV"
      formControl={formControl}
      accept=".csv"
      onUploadProgress={setUploadProgress}
      helperText="Upload a CSV file with user data"
    />
  );
};
```

### Multi-Step Form State
```jsx
const MultiStepForm = () => {
  const [step, setStep] = useState(1);
  const formControl = useForm({
    mode: 'onChange',
    defaultValues: {
      displayName: '',
      email: '',
      licenseType: ''
    }
  });

  const isStepValid = (stepNumber) => {
    const stepFields = getFieldsForStep(stepNumber);
    return stepFields.every(field => 
      formControl.formState.errors[field] === undefined
    );
  };

  return (
    <CippFormPage
      formControl={formControl}
      hideSubmit={step < 3}
      {...props}
    >
      {step === 1 && <BasicInfoStep />}
      {step === 2 && <LicenseStep />}
      {step === 3 && <ReviewStep />}
      
      <Stack direction="row" spacing={2} justifyContent="flex-end">
        {step > 1 && (
          <Button onClick={() => setStep(step - 1)}>
            Previous
          </Button>
        )}
        {step < 3 && (
          <Button
            variant="contained"
            disabled={!isStepValid(step)}
            onClick={() => setStep(step + 1)}
          >
            Next
          </Button>
        )}
      </Stack>
    </CippFormPage>
  );
};
```

## Best Practices

1. **Validation**: Use React Hook Form's built-in validators for consistent validation
2. **Default Values**: Provide sensible defaults to improve UX
3. **Error Handling**: Show clear, actionable error messages
4. **Loading States**: Indicate when forms are submitting
5. **Field Organization**: Group related fields in sections
6. **Responsive Design**: Ensure forms work on all screen sizes
7. **Accessibility**: Include proper labels and ARIA attributes
8. **Data Transformation**: Use customDataformatter for API compatibility
9. **Query Invalidation**: Invalidate relevant cache keys after submission
10. **Progressive Enhancement**: Start with basic functionality, add advanced features