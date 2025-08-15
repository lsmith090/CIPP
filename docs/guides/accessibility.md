# Accessibility Guidelines

This document outlines the accessibility (a11y) standards and implementation patterns used in CIPP to ensure the application is usable by all users, including those with disabilities.

## Accessibility Principles

CIPP follows the Web Content Accessibility Guidelines (WCAG) 2.1 Level AA standards, focusing on the four main principles:

### 1. **Perceivable**
Information and UI components must be presentable in ways users can perceive.

### 2. **Operable** 
UI components and navigation must be operable by all users.

### 3. **Understandable**
Information and UI operation must be understandable.

### 4. **Robust**
Content must be robust enough for interpretation by assistive technologies.

## Material-UI Accessibility Features

CIPP leverages Material-UI's built-in accessibility features:

### 1. Semantic HTML and ARIA
Material-UI components use proper semantic HTML and ARIA attributes:

```jsx
// Material-UI components include proper ARIA attributes
<Button
  aria-label="Add new user"
  aria-describedby="add-user-help-text"
>
  Add User
</Button>

<Typography id="add-user-help-text" variant="caption">
  Create a new user account
</Typography>
```

### 2. Focus Management
Proper focus management is built into MUI components:

```jsx
// Dialog focus management
<Dialog
  open={open}
  onClose={onClose}
  aria-labelledby="dialog-title"
  aria-describedby="dialog-description"
>
  <DialogTitle id="dialog-title">
    User Management
  </DialogTitle>
  <DialogContent>
    <Typography id="dialog-description">
      Manage user accounts and permissions
    </Typography>
  </DialogContent>
</Dialog>
```

## Form Accessibility

### 1. Form Labels and Descriptions
All form fields include proper labels and error messaging:

```jsx
const CippAccessibleForm = () => {
  const { errors } = useFormState({ control: formControl.control });

  return (
    <form>
      <TextField
        label="Email Address"
        type="email"
        required
        error={!!errors.email}
        helperText={errors.email?.message || "Enter your email address"}
        aria-describedby="email-help"
        inputProps={{
          'aria-required': true,
          'aria-invalid': !!errors.email,
        }}
      />
      
      <Typography id="email-help" variant="caption" sx={{ mt: 1, display: 'block' }}>
        We'll use this email to send you notifications
      </Typography>
    </form>
  );
};
```

### 2. Form Validation Accessibility
Error messages are properly associated with form fields:

```jsx
const CippFormComponent = ({ 
  name, 
  label, 
  formControl, 
  validators,
  required = false,
  ...other 
}) => {
  const { errors } = useFormState({ control: formControl.control });
  const fieldError = get(errors, name);
  const fieldId = `field-${name}`;
  const errorId = `${fieldId}-error`;
  const helpId = `${fieldId}-help`;

  return (
    <TextField
      id={fieldId}
      label={label}
      error={!!fieldError}
      helperText={fieldError?.message}
      required={required}
      {...formControl.register(name, validators)}
      inputProps={{
        'aria-required': required,
        'aria-invalid': !!fieldError,
        'aria-describedby': fieldError ? errorId : helpId,
      }}
      {...other}
    />
  );
};
```

## Navigation Accessibility

### 1. Skip Links
Skip links allow keyboard users to bypass repetitive navigation:

```jsx
const CippSkipLinks = () => (
  <Box
    sx={{
      position: 'absolute',
      top: -40,
      left: 6,
      zIndex: 1300,
      '&:focus-within': {
        top: 6,
      },
    }}
  >
    <Button
      variant="contained"
      href="#main-content"
      sx={{
        '&:focus': {
          outline: '2px solid',
          outlineColor: 'primary.main',
          outlineOffset: 2,
        },
      }}
    >
      Skip to main content
    </Button>
  </Box>
);

// Usage in layout
const MainLayout = ({ children }) => (
  <Box>
    <CippSkipLinks />
    <SideNav />
    <Box component="main" id="main-content" tabIndex={-1}>
      {children}
    </Box>
  </Box>
);
```

### 2. Accessible Navigation Menus
Navigation menus include proper ARIA attributes and keyboard support:

```jsx
const CippSideNavItem = ({ 
  title, 
  path, 
  icon, 
  active, 
  children, 
  hasChildren 
}) => {
  const [isOpen, setIsOpen] = useState(false);
  const itemId = `nav-${title.toLowerCase().replace(/\s+/g, '-')}`;

  return (
    <li role="none">
      <Box
        component={path ? Link : 'button'}
        href={path}
        role={hasChildren ? 'button' : 'menuitem'}
        aria-expanded={hasChildren ? isOpen : undefined}
        aria-controls={hasChildren ? `${itemId}-submenu` : undefined}
        aria-current={active ? 'page' : undefined}
        onClick={hasChildren ? () => setIsOpen(!isOpen) : undefined}
        sx={{
          display: 'flex',
          alignItems: 'center',
          width: '100%',
          p: 1,
          textDecoration: 'none',
          color: 'inherit',
          backgroundColor: active ? 'action.selected' : 'transparent',
          '&:hover': {
            backgroundColor: 'action.hover',
          },
          '&:focus': {
            outline: '2px solid',
            outlineColor: 'primary.main',
            outlineOffset: -2,
          },
        }}
      >
        {icon && (
          <Box sx={{ mr: 2 }} aria-hidden="true">
            <SvgIcon fontSize="small">{icon}</SvgIcon>
          </Box>
        )}
        <Typography variant="body2">{title}</Typography>
        {hasChildren && (
          <Box sx={{ ml: 'auto' }} aria-hidden="true">
            <SvgIcon fontSize="small">
              {isOpen ? <ChevronUpIcon /> : <ChevronDownIcon />}
            </SvgIcon>
          </Box>
        )}
      </Box>
      
      {hasChildren && (
        <Collapse in={isOpen}>
          <Box
            id={`${itemId}-submenu`}
            role="menu"
            sx={{ ml: 2 }}
          >
            {children}
          </Box>
        </Collapse>
      )}
    </li>
  );
};
```

## Table Accessibility

### 1. Data Table with Proper Headers
Tables include proper headers and navigation support:

```jsx
const CippAccessibleDataTable = ({ data, columns, title }) => {
  const tableId = `table-${title.toLowerCase().replace(/\s+/g, '-')}`;

  return (
    <Box>
      <Typography 
        variant="h6" 
        component="h2" 
        id={`${tableId}-title`}
        sx={{ mb: 2 }}
      >
        {title}
      </Typography>
      
      <Table
        aria-labelledby={`${tableId}-title`}
        aria-describedby={`${tableId}-summary`}
      >
        <caption id={`${tableId}-summary`} style={{ captionSide: 'bottom' }}>
          Table showing {data.length} items. Use arrow keys to navigate.
        </caption>
        
        <TableHead>
          <TableRow>
            {columns.map((column) => (
              <TableCell
                key={column.accessorKey}
                scope="col"
                aria-sort={getSortDirection(column)}
              >
                {column.sortable ? (
                  <TableSortLabel
                    active={sortColumn === column.accessorKey}
                    direction={sortDirection}
                    onClick={() => handleSort(column.accessorKey)}
                  >
                    {column.header}
                  </TableSortLabel>
                ) : (
                  column.header
                )}
              </TableCell>
            ))}
          </TableRow>
        </TableHead>
        
        <TableBody>
          {data.map((row, rowIndex) => (
            <TableRow
              key={row.id}
              hover
              tabIndex={0}
              role="row"
              aria-rowindex={rowIndex + 2} // +2 for header row
            >
              {columns.map((column) => (
                <TableCell
                  key={column.accessorKey}
                  scope={column.accessorKey === 'name' ? 'row' : undefined}
                >
                  {column.cell ? column.cell({ row }) : row[column.accessorKey]}
                </TableCell>
              ))}
            </TableRow>
          ))}
        </TableBody>
      </Table>
    </Box>
  );
};
```

## Loading States and Feedback

### 1. Accessible Loading States
Loading states provide appropriate feedback to assistive technologies:

```jsx
const CippLoadingState = ({ isLoading, children, loadingText = "Loading..." }) => {
  if (isLoading) {
    return (
      <Box
        role="status"
        aria-live="polite"
        aria-label={loadingText}
        sx={{
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          p: 3,
        }}
      >
        <CircularProgress size={40} />
        <Typography variant="body2" sx={{ ml: 2 }}>
          {loadingText}
        </Typography>
      </Box>
    );
  }

  return children;
};
```

### 2. Error States with Clear Messaging
Error states provide clear, actionable information:

```jsx
const CippErrorState = ({ error, onRetry, title = "An error occurred" }) => {
  return (
    <Box
      role="alert"
      aria-live="assertive"
      sx={{
        display: 'flex',
        flexDirection: 'column',
        alignItems: 'center',
        p: 3,
        textAlign: 'center',
      }}
    >
      <Avatar
        sx={{
          bgcolor: 'error.alpha12',
          color: 'error.main',
          width: 64,
          height: 64,
          mb: 2,
        }}
      >
        <ErrorIcon />
      </Avatar>
      
      <Typography variant="h6" component="h3" gutterBottom>
        {title}
      </Typography>
      
      <Typography variant="body2" color="text.secondary" sx={{ mb: 3 }}>
        {error?.message || "Please try again or contact support if the problem persists."}
      </Typography>
      
      {onRetry && (
        <Button
          variant="contained"
          onClick={onRetry}
          aria-label="Retry the failed operation"
        >
          Try Again
        </Button>
      )}
    </Box>
  );
};
```

## Best Practices Summary

### ✅ Accessibility Checklist
- [ ] All interactive elements are keyboard accessible
- [ ] Proper ARIA labels and descriptions are provided
- [ ] Color contrast meets WCAG AA standards (4.5:1 for normal text)
- [ ] Focus indicators are visible and clear
- [ ] Form labels are properly associated with inputs
- [ ] Error messages are descriptive and actionable
- [ ] Loading states provide appropriate feedback
- [ ] Tables have proper headers and structure
- [ ] Navigation includes skip links
- [ ] Images have descriptive alt text
- [ ] Dynamic content changes are announced
- [ ] Page titles are descriptive and unique

### ✅ Testing Tools
1. **Automated Testing**: Use tools like axe-core for automated accessibility testing
2. **Keyboard Navigation**: Test all functionality using only keyboard
3. **Screen Reader Testing**: Test with NVDA, JAWS, or VoiceOver
4. **Color Contrast**: Use tools like WebAIM's contrast checker
5. **Focus Management**: Verify focus moves logically through the interface

### ✅ Common Patterns
```jsx
// Button with proper labeling
<Button aria-label="Delete user John Doe">
  <DeleteIcon />
</Button>

// Form field with error
<TextField
  label="Email"
  error={!!error}
  helperText={error?.message}
  aria-describedby="email-help"
  aria-invalid={!!error}
/>

// Loading state
<Box role="status" aria-live="polite">
  Loading user data...
</Box>

// Error state
<Box role="alert" aria-live="assertive">
  Failed to load data. Please try again.
</Box>
```

By following these accessibility guidelines, CIPP ensures that all users can effectively interact with the application, regardless of their abilities or assistive technologies used.