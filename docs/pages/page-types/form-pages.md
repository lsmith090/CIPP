# Form Pages - CippFormPage Pattern

Form pages are used throughout CIPP for creating and editing entities. They provide a consistent interface for data input with validation, submission handling, and error management using React Hook Form integration.

## Core Component

The `CippFormPage` component is the foundation for all form-based pages in CIPP.

**Location**: `/src/components/CippFormPages/CippFormPage.jsx`

## Basic Usage

```javascript
import CippFormPage from "/src/components/CippFormPages/CippFormPage";
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { useForm } from "react-hook-form";

const Page = () => {
  const formControl = useForm({
    mode: "onBlur",
    defaultValues: {
      displayName: "",
      email: ""
    }
  });

  return (
    <CippFormPage
      title="User"
      formControl={formControl}
      postUrl="/api/AddUser"
    >
      {/* Form fields go here */}
    </CippFormPage>
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default Page;
```

## Component Props

### Required Props

| Prop | Type | Description |
|------|------|-------------|
| `formControl` | `UseFormReturn` | React Hook Form control object |
| `postUrl` | `string` | API endpoint for form submission |
| `children` | `ReactNode` | Form fields and content |

### Core Configuration Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `title` | `string` | `""` | Form title displayed in header |
| `formPageType` | `string` | `"Add"` | Type prefix for title (Add/Edit) |
| `backButtonTitle` | `string` | `"Back"` | Text for back button |

### UI Control Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `hideBackButton` | `boolean` | `false` | Hide the back navigation button |
| `hidePageType` | `boolean` | `false` | Hide the page type prefix in title |
| `hideTitle` | `boolean` | `false` | Hide the entire title section |
| `hideSubmit` | `boolean` | `false` | Hide the submit button |

### Advanced Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `customDataformatter` | `function` | `null` | Custom data transformation before submit |
| `resetForm` | `boolean` | `false` | Reset form after successful submission |
| `allowResubmit` | `boolean` | `false` | Allow resubmission without form changes |
| `queryKey` | `string/array` | `null` | React Query keys to invalidate on success |
| `titleButton` | `ReactNode` | `null` | Additional button next to title |
| `addedButtons` | `ReactNode` | `null` | Additional buttons in form actions |

## Real-World Example: Add User Page

Here's the actual implementation from `/src/pages/identity/administration/users/add.jsx`:

```javascript
import { Box } from "@mui/material";
import CippFormPage from "../../../../components/CippFormPages/CippFormPage";
import { Layout as DashboardLayout } from "/src/layouts/index.js";
import { useForm, useWatch } from "react-hook-form";
import { CippFormUserSelector } from "/src/components/CippComponents/CippFormUserSelector";
import { useSettings } from "../../../../hooks/use-settings";
import { useEffect } from "react";
import CippAddEditUser from "../../../../components/CippFormPages/CippAddEditUser";

const Page = () => {
  const userSettingsDefaults = useSettings();

  const formControl = useForm({
    mode: "onBlur",
    defaultValues: {
      tenantFilter: userSettingsDefaults.currentTenant,
      usageLocation: userSettingsDefaults.usageLocation,
    },
  });

  const formValues = useWatch({ control: formControl.control, name: "userProperties" });
  
  useEffect(() => {
    if (formValues) {
      const { userPrincipalName, usageLocation, ...restFields } = formValues.addedFields || {};
      let newFields = { ...restFields };
      
      if (userPrincipalName) {
        const [mailNickname, domainNamePart] = userPrincipalName.split("@");
        if (mailNickname) {
          newFields.mailNickname = mailNickname;
        }
        if (domainNamePart) {
          newFields.primDomain = { label: domainNamePart, value: domainNamePart };
        }
      }
      
      if (usageLocation) {
        newFields.usageLocation = { label: usageLocation, value: usageLocation };
      }
      
      newFields.tenantFilter = userSettingsDefaults.currentTenant;
      formControl.reset(newFields);
    }
  }, [formValues]);

  return (
    <>
      <CippFormPage
        queryKey={`Users-${userSettingsDefaults.currentTenant}`}
        formControl={formControl}
        title="User"
        backButtonTitle="User Overview"
        postUrl="/api/AddUser"
      >
        <Box sx={{ my: 2 }}>
          <CippFormUserSelector
            formControl={formControl}
            name="userProperties"
            label="Copy properties from another user"
            multiple={false}
            select="id,userPrincipalName,displayName,givenName,surname,mailNickname,jobTitle,department,streetAddress,city,state,postalCode,companyName,mobilePhone,businessPhones,usageLocation,office"
            addedField={{
              groupType: "calculatedGroupType",
              displayName: "displayName",
              userPrincipalName: "userPrincipalName",
              id: "id",
              givenName: "givenName",
              surname: "surname",
              mailNickname: "mailNickname",
              jobTitle: "jobTitle",
              department: "department",
              streetAddress: "streetAddress",
              city: "city",
              state: "state",
              postalCode: "postalCode",
              companyName: "companyName",
              mobilePhone: "mobilePhone",
              businessPhones: "businessPhones",
              usageLocation: "usageLocation",
              office: "office",
            }}
          />
        </Box>
        <Box sx={{ my: 2 }}>
          <CippAddEditUser 
            formControl={formControl} 
            userSettingsDefaults={userSettingsDefaults} 
          />
        </Box>
      </CippFormPage>
    </>
  );
};

Page.getLayout = (page) => <DashboardLayout>{page}</DashboardLayout>;
export default Page;
```

## Key Features

### 1. React Hook Form Integration

The component is built around React Hook Form for:
- Form state management
- Validation handling
- Performance optimization
- Controlled/uncontrolled input support

```javascript
const formControl = useForm({
  mode: "onBlur",           // Validate on blur
  defaultValues: {          // Initial form values
    name: "",
    email: ""
  },
  resolver: yupResolver(schema) // Optional validation schema
});
```

### 2. Automatic Form Submission

The component handles:
- Form validation before submission
- API call with form data
- Loading states during submission
- Success/error message display
- Optional form reset after success

### 3. URL Parameter Integration

Form automatically populates from URL query parameters:

```javascript
// URL: /users/add?name=John&email=john@example.com
// Form will auto-populate with these values
```

### 4. Custom Data Formatting

Transform form data before submission:

```javascript
const customDataFormatter = (formData) => {
  return {
    ...formData,
    fullName: `${formData.firstName} ${formData.lastName}`,
    timestamp: new Date().toISOString()
  };
};

<CippFormPage
  customDataformatter={customDataFormatter}
  // ... other props
/>
```

## Form Field Patterns

### Basic Input Fields

```javascript
import { CippFormComponent } from "/src/components/CippComponents/CippFormComponent";

<CippFormComponent
  type="text"
  label="Display Name"
  name="displayName"
  formControl={formControl}
  validators={{
    required: "Display name is required"
  }}
/>
```

### Dropdown Selectors

```javascript
import { CippFormTenantSelector } from "/src/components/CippComponents/CippFormTenantSelector";

<CippFormTenantSelector
  formControl={formControl}
  name="tenantFilter"
  label="Select Tenant"
  allTenants={false}
/>
```

### User/Group Selectors

```javascript
import { CippFormUserSelector } from "/src/components/CippComponents/CippFormUserSelector";

<CippFormUserSelector
  formControl={formControl}
  name="selectedUsers"
  label="Select Users"
  multiple={true}
  select="id,displayName,userPrincipalName"
/>
```

### License Selectors

```javascript
import { CippFormLicenseSelector } from "/src/components/CippComponents/CippFormLicenseSelector";

<CippFormLicenseSelector
  formControl={formControl}
  name="assignedLicenses"
  label="Assign Licenses"
  multiple={true}
/>
```

## Validation Patterns

### Built-in Validation

```javascript
<CippFormComponent
  type="email"
  name="email"
  label="Email Address"
  formControl={formControl}
  validators={{
    required: "Email is required",
    pattern: {
      value: /^\S+@\S+\.\S+$/,
      message: "Invalid email format"
    }
  }}
/>
```

### Cross-field Validation

```javascript
const validatePasswordMatch = (value) => {
  const password = formControl.getValues("password");
  return value === password || "Passwords do not match";
};

<CippFormComponent
  type="password"
  name="confirmPassword"
  label="Confirm Password"
  formControl={formControl}
  validators={{
    validate: validatePasswordMatch
  }}
/>
```

### Conditional Validation

```javascript
const watchEnableFeature = useWatch({
  control: formControl.control,
  name: "enableFeature"
});

<CippFormComponent
  type="text"
  name="featureValue"
  label="Feature Value"
  formControl={formControl}
  validators={watchEnableFeature ? {
    required: "Feature value is required when feature is enabled"
  } : {}}
/>
```

## Form Sections

Complex forms can be organized into sections:

```javascript
import { CippFormSection } from "/src/components/CippFormPages/CippFormSection";

<CippFormPage>
  <CippFormSection title="Basic Information">
    <CippFormComponent name="displayName" label="Display Name" />
    <CippFormComponent name="email" label="Email" />
  </CippFormSection>
  
  <CippFormSection title="Contact Information">
    <CippFormComponent name="phone" label="Phone" />
    <CippFormComponent name="address" label="Address" />
  </CippFormSection>
</CippFormPage>
```

## Advanced Patterns

### Dynamic Form Fields

```javascript
const watchUserType = useWatch({
  control: formControl.control,
  name: "userType"
});

return (
  <CippFormPage>
    <CippFormComponent name="userType" label="User Type" type="select" />
    
    {watchUserType === "Employee" && (
      <CippFormComponent name="employeeId" label="Employee ID" />
    )}
    
    {watchUserType === "Guest" && (
      <CippFormComponent name="sponsorEmail" label="Sponsor Email" />
    )}
  </CippFormPage>
);
```

### Form Field Arrays

```javascript
import { useFieldArray } from "react-hook-form";

const { fields, append, remove } = useFieldArray({
  control: formControl.control,
  name: "phoneNumbers"
});

return (
  <Box>
    {fields.map((field, index) => (
      <Box key={field.id} sx={{ display: "flex", gap: 2 }}>
        <CippFormComponent
          name={`phoneNumbers.${index}.number`}
          label="Phone Number"
        />
        <Button onClick={() => remove(index)}>Remove</Button>
      </Box>
    ))}
    <Button onClick={() => append({ number: "" })}>
      Add Phone Number
    </Button>
  </Box>
);
```

### File Upload Fields

```javascript
import { CippDropzone } from "/src/components/CippComponents/CippDropzone";

<CippDropzone
  formControl={formControl}
  name="uploadedFile"
  label="Upload File"
  acceptedFiles={[".csv", ".xlsx"]}
  maxSize={5242880} // 5MB
/>
```

## Data Flow

1. **Form Initialization**: `useForm` creates form state with default values
2. **URL Integration**: Query parameters populate form fields
3. **User Input**: Form fields update via React Hook Form
4. **Validation**: Real-time validation on blur/change
5. **Submission**: Form data processed and sent to API
6. **Response Handling**: Success/error messages displayed
7. **Cache Invalidation**: Related queries refreshed on success

## Form State Management

### Watching Form Values

```javascript
const watchedValues = useWatch({
  control: formControl.control,
  name: ["displayName", "email"]
});

// React to changes
useEffect(() => {
  if (watchedValues[0] && watchedValues[1]) {
    // Do something when both fields have values
  }
}, [watchedValues]);
```

### Form Reset Patterns

```javascript
// Reset to initial values
formControl.reset();

// Reset with new values
formControl.reset({
  displayName: "New Name",
  email: "new@email.com"
});

// Reset specific fields
formControl.resetField("email");
```

### Error Handling

```javascript
// Set field errors
formControl.setError("email", {
  type: "manual",
  message: "Email already exists"
});

// Clear errors
formControl.clearErrors("email");

// Get field errors
const emailError = formControl.formState.errors.email;
```

## API Integration

### Standard Submission

```javascript
<CippFormPage
  postUrl="/api/AddUser"
  // Form data automatically sent as JSON
/>
```

### Custom Data Transformation

```javascript
const customFormatter = (formData) => {
  // Transform data before sending to API
  return {
    user: {
      displayName: formData.displayName,
      mail: formData.email
    },
    settings: {
      department: formData.department
    }
  };
};

<CippFormPage
  postUrl="/api/AddUser"
  customDataformatter={customFormatter}
/>
```

### Multi-step Submission

```javascript
const handleMultiStep = async (formData) => {
  // Step 1: Create user
  const userResponse = await apiCall("/api/AddUser", formData.user);
  
  // Step 2: Assign licenses
  if (formData.licenses.length > 0) {
    await apiCall("/api/AssignLicenses", {
      userId: userResponse.id,
      licenses: formData.licenses
    });
  }
};
```

## Performance Optimization

### Memoization

```javascript
import { useMemo } from "react";

const expensiveOptions = useMemo(() => {
  return processLargeDataset(rawData);
}, [rawData]);
```

### Debounced Validation

```javascript
import { useDebounce } from "/src/hooks/use-debounce";

const debouncedEmail = useDebounce(watchEmail, 500);

useEffect(() => {
  if (debouncedEmail) {
    // Validate email availability
    validateEmailAvailability(debouncedEmail);
  }
}, [debouncedEmail]);
```

### Lazy Loading

```javascript
import dynamic from "next/dynamic";

const LazyComplexComponent = dynamic(
  () => import("./ComplexComponent"),
  { ssr: false }
);
```

## Accessibility Considerations

### Form Labels

```javascript
<CippFormComponent
  name="email"
  label="Email Address"
  aria-describedby="email-help"
  helperText="We'll never share your email"
/>
```

### Error Announcements

```javascript
// Errors are automatically announced by React Hook Form
// Additional ARIA attributes can be added:
<CippFormComponent
  name="password"
  label="Password"
  aria-invalid={!!formControl.formState.errors.password}
  aria-describedby="password-error"
/>
```

### Keyboard Navigation

```javascript
// Focus management
const handleSubmit = async (data) => {
  try {
    await submitForm(data);
    // Focus success message or next form
    successMessageRef.current?.focus();
  } catch (error) {
    // Focus first error field
    const firstErrorField = Object.keys(formControl.formState.errors)[0];
    formControl.setFocus(firstErrorField);
  }
};
```

## Common Use Cases

### Add/Create Forms

```javascript
const AddUserPage = () => {
  const formControl = useForm({
    defaultValues: {
      accountEnabled: true,
      usageLocation: "US"
    }
  });

  return (
    <CippFormPage
      title="User"
      formPageType="Add"
      formControl={formControl}
      postUrl="/api/AddUser"
    >
      <UserFormFields />
    </CippFormPage>
  );
};
```

### Edit Forms

```javascript
const EditUserPage = () => {
  const { userId } = useRouter().query;
  const formControl = useForm();

  // Load existing data
  const { data: userData } = ApiGetCall({
    url: `/api/GetUser?id=${userId}`,
    queryKey: `User-${userId}`
  });

  useEffect(() => {
    if (userData) {
      formControl.reset(userData);
    }
  }, [userData]);

  return (
    <CippFormPage
      title="User"
      formPageType="Edit"
      formControl={formControl}
      postUrl="/api/UpdateUser"
    >
      <UserFormFields />
    </CippFormPage>
  );
};
```

### Settings Forms

```javascript
const SettingsPage = () => {
  const formControl = useForm();

  return (
    <CippFormPage
      title="Settings"
      formControl={formControl}
      postUrl="/api/UpdateSettings"
      resetForm={false}
      allowResubmit={true}
      hidePageType={true}
    >
      <SettingsFormFields />
    </CippFormPage>
  );
};
```

## Best Practices

1. **Form Structure**: Organize forms logically with clear sections
2. **Validation**: Provide real-time feedback with helpful error messages
3. **Performance**: Use debouncing for expensive operations
4. **Accessibility**: Ensure proper labeling and keyboard navigation
5. **User Experience**: Show loading states and clear success/error feedback
6. **Data Integrity**: Validate on both client and server sides
7. **Progressive Enhancement**: Work without JavaScript when possible
8. **Mobile Optimization**: Ensure forms work well on touch devices

## Troubleshooting

### Common Issues

**Form Not Submitting**
- Check form validation state with `formControl.formState.isValid`
- Verify all required fields have values
- Check network tab for API errors

**Default Values Not Loading**
- Ensure `defaultValues` is set in `useForm()`
- Check if async data is loading before form initialization
- Use `formControl.reset()` for dynamic default values

**Validation Not Working**
- Verify validator configuration syntax
- Check if validation mode is set correctly
- Ensure error messages are being displayed

**Performance Issues**
- Use `mode: "onBlur"` instead of `onChange` for better performance
- Implement debouncing for expensive operations
- Avoid unnecessary re-renders with proper dependencies

## Related Components

- [`CippFormComponent`](../components/cipp-forms/README.md) - Base form input component
- [`CippFormSection`](../components/cipp-forms/README.md) - Form section grouping
- [`CippFormUserSelector`](../components/cipp-components/README.md) - User selection component
- [`CippFormTenantSelector`](../components/cipp-components/README.md) - Tenant selection component
- [`CippDropzone`](../components/cipp-components/README.md) - File upload component