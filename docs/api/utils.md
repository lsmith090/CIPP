# CIPP Utility Functions Documentation

This document provides comprehensive documentation for all utility functions used in the CIPP application. These utilities provide common functionality for data processing, formatting, validation, and various helper operations.

## Table of Contents

1. [Data Processing](#data-processing)
2. [Formatting & Translation](#formatting--translation)
3. [Validation & Conversion](#validation--conversion)
4. [Array & Object Utilities](#array--object-utilities)
5. [String Utilities](#string-utilities)
6. [System Utilities](#system-utilities)

## Data Processing

### applyFilters

Client-side filtering utility with support for multiple filter operators.

```javascript
import { applyFilters } from '/src/utils/apply-filters';

const users = [
  { name: "John Doe", department: "IT", age: 30 },
  { name: "Jane Smith", department: "HR", age: 25 },
  { name: "Bob Johnson", department: "IT", age: 35 }
];

const filters = [
  { property: "department", operator: "equals", value: "IT" },
  { property: "age", operator: "greaterThan", value: 28 }
];

const filteredUsers = applyFilters(users, filters);
// Result: [{ name: "John Doe", department: "IT", age: 30 }, { name: "Bob Johnson", department: "IT", age: 35 }]
```

**Parameters:**
- `rows` - Array of objects to filter
- `filters` - Array of filter objects

**Filter Object Structure:**
```javascript
{
  property: string,    // Object property to filter on
  operator: string,    // Filter operator (see below)
  value: any          // Filter value (not needed for isBlank/isPresent)
}
```

**Supported Operators:**

| Operator | Field Types | Description | Example |
|----------|-------------|-------------|---------|
| `contains` | string | Contains substring (case-insensitive) | `{ operator: "contains", value: "john" }` |
| `equals` | string, number | Exact match (== comparison) | `{ operator: "equals", value: "IT" }` |
| `notEqual` | string, number | Not equal (!== comparison) | `{ operator: "notEqual", value: "HR" }` |
| `startsWith` | string | Starts with substring | `{ operator: "startsWith", value: "John" }` |
| `endsWith` | string | Ends with substring | `{ operator: "endsWith", value: ".com" }` |
| `notContains` | string | Does not contain substring | `{ operator: "notContains", value: "test" }` |
| `greaterThan` | number | Greater than value | `{ operator: "greaterThan", value: 25 }` |
| `lessThan` | number | Less than value | `{ operator: "lessThan", value: 50 }` |
| `isAfter` | date | Date is after value | `{ operator: "isAfter", value: "2023-01-01" }` |
| `isBefore` | date | Date is before value | `{ operator: "isBefore", value: "2023-12-31" }` |
| `isBlank` | any | Value is null or undefined | `{ operator: "isBlank" }` |
| `isPresent` | any | Value is not null/undefined | `{ operator: "isPresent" }` |

### applySort

Stable sorting utility for arrays of objects.

```javascript
import { applySort } from '/src/utils/apply-sort';

const users = [
  { name: "John", age: 30 },
  { name: "Jane", age: 25 },
  { name: "Bob", age: 35 }
];

// Sort by age, descending
const sortedUsers = applySort(users, "age", "desc");
// Result: [{ name: "Bob", age: 35 }, { name: "John", age: 30 }, { name: "Jane", age: 25 }]

// Sort by name, ascending
const sortedByName = applySort(users, "name", "asc");
// Result: [{ name: "Bob", age: 35 }, { name: "Jane", age: 25 }, { name: "John", age: 30 }]
```

**Parameters:**
- `documents` - Array of objects to sort
- `sortBy` - Property name to sort by
- `sortDir` - Sort direction: `"asc"` or `"desc"`

**Features:**
- Stable sort (maintains relative order of equal elements)
- Handles undefined values gracefully
- Preserves original array order for equal values

### applyPagination

Simple pagination utility for client-side data.

```javascript
import { applyPagination } from '/src/utils/apply-pagination';

const items = Array.from({ length: 100 }, (_, i) => ({ id: i, name: `Item ${i}` }));

// Get page 2 with 10 items per page (items 10-19)
const page2 = applyPagination(items, 1, 10);
// Result: [{ id: 10, name: "Item 10" }, ..., { id: 19, name: "Item 19" }]

// Get first page with 5 items per page (items 0-4)
const page1 = applyPagination(items, 0, 5);
// Result: [{ id: 0, name: "Item 0" }, ..., { id: 4, name: "Item 4" }]
```

**Parameters:**
- `documents` - Array to paginate
- `page` - Zero-based page number
- `rowsPerPage` - Number of items per page

## Formatting & Translation

### getCippFormatting

Comprehensive data formatting utility for consistent display across CIPP tables and components.

```javascript
import { getCippFormatting } from '/src/utils/get-cipp-formatting';

// Format different data types
const formattedDate = getCippFormatting("2023-12-01T10:30:00Z", "createdDateTime", "component");
// Returns: <CippTimeAgo data="2023-12-01T10:30:00Z" />

const formattedText = getCippFormatting("2023-12-01T10:30:00Z", "createdDateTime", "text");
// Returns: "12/1/2023, 10:30:00 AM"

const formattedBoolean = getCippFormatting(true, "accountEnabled", "component");
// Returns: <Check fontSize="10" />

const formattedArray = getCippFormatting(["tenant1", "tenant2"], "tenantFilter", "component");
// Returns: CollapsibleChipList with tenant chips
```

**Parameters:**
- `data` - Data to format
- `cellName` - Field name (determines formatting rules)
- `type` - Output type: `"text"` or `"component"`
- `canReceive` - Additional formatting options
- `flatten` - Whether to flatten nested objects (default: true)

**Supported Field Types:**

| Field Pattern | Description | Example Output |
|---------------|-------------|----------------|
| **Date/Time Fields** | Fields ending in DateTime, Date, Time, etc. | Time ago component or formatted date |
| **Boolean Fields** | `true`/`false` values | ✓ / ✗ icons or "Yes"/"No" text |
| **Array Fields** | Arrays of strings or objects | Collapsible chip list |
| **GUID Fields** | UUID format strings | Automatically resolved to display names |
| **License Fields** | `assignedLicenses` | Translated license names |
| **Role Fields** | `unifiedRoles`, `roleDefinitionId` | Translated role names |
| **Tenant Fields** | `tenantFilter`, `Tenant`, etc. | Tenant chips with icons |
| **URL Fields** | Strings starting with http | Clickable links |
| **JSON Fields** | JSON strings | Parsed and displayed as tables |
| **Password Fields** | Password-related fields | Hidden with copy-to-clipboard |

**Special Field Handling:**

```javascript
// Bytes to GB conversion
getCippFormatting(1073741824, "storageUsedInBytes", "text");
// Returns: "1.00 GB"

// Portal links with icons
getCippFormatting("https://portal.azure.com", "portal_azure", "component");
// Returns: <Link with Azure icon>

// Progress bars
getCippFormatting(75, "ScorePercentage", "component");
// Returns: <LinearProgressWithLabel value={75} />

// License translation
getCippFormatting([{skuId: "..."}], "assignedLicenses", "component");
// Returns: Chips with translated license names

// Complex object display
getCippFormatting([{name: "John", role: "Admin"}], "users", "component");
// Returns: <CippDataTableButton> for expandable table view
```

### getCippTranslation

Translates field names and technical terms into human-readable labels.

```javascript
import { getCippTranslation } from '/src/utils/get-cipp-translation';

// Basic translation
getCippTranslation("userPrincipalName");
// Returns: "User Principal Name"

// CamelCase conversion
getCippTranslation("lastModifiedDateTime");
// Returns: "Last Modified Date Time"

// Extension field handling
getCippTranslation("extension_customAttribute1");
// Returns: "Custom Attribute1"

// Schema extension handling
getCippTranslation("ext123_department");
// Returns: "Department"

// Dot notation conversion
getCippTranslation("user.profile.department");
// Returns: "User - Profile - Department"
```

**Translation Rules:**
1. Check predefined translations in `CippTranslations`
2. Handle extension field prefixes
3. Convert CamelCase to spaced words
4. Replace underscores with spaces
5. Replace dots with " - "
6. Capitalize first letter of each word

### License and Role Translation

Specialized translation utilities for Microsoft 365 licenses and roles.

```javascript
import { getCippLicenseTranslation } from '/src/utils/get-cipp-license-translation';
import { getCippRoleTranslation } from '/src/utils/get-cipp-role-translation';

// Translate license SKUs
const licenses = getCippLicenseTranslation([
  { skuId: "6fd2c87f-b296-42f0-b197-1e91e994b900" }
]);
// Returns: ["Office 365 E3"]

// Translate role definition IDs
const roleName = getCippRoleTranslation("62e90394-69f5-4237-9190-012177145e10");
// Returns: "Global Administrator"
```

## Validation & Conversion

### getCippError

Extracts meaningful error messages from various API response formats.

```javascript
import { getCippError } from '/src/utils/get-cipp-error';

// Handle different error response formats
const error1 = { response: { data: { result: "User not found" } } };
const error2 = { response: { data: { error: "Access denied" } } };
const error3 = { message: "Network error" };

console.log(getCippError(error1)); // "User not found"
console.log(getCippError(error2)); // "Access denied"
console.log(getCippError(error3)); // "Network error"

// Handle HTML error pages
const htmlError = { 
  response: { 
    data: "<!DOCTYPE html><html>Error page</html>" 
  },
  message: "Server error"
};
console.log(getCippError(htmlError)); // "Server error"
```

**Error Extraction Priority:**
1. `response.data.result`
2. `response.data.error`
3. `response.data.message`
4. For HTML responses: `message`
5. `response.data.Results`
6. `response.data`
7. `message`

### getCippValidator

Validation utility for form inputs and data validation.

```javascript
import { getCippValidator } from '/src/utils/get-cipp-validator';

// Email validation
const emailValidator = getCippValidator("email");
const isValidEmail = emailValidator("user@example.com"); // true

// Required field validation
const requiredValidator = getCippValidator("required");
const isRequired = requiredValidator(""); // false

// Custom pattern validation
const phoneValidator = getCippValidator("phone");
const isValidPhone = phoneValidator("+1-555-123-4567"); // true
```

**Available Validators:**
- `required` - Non-empty values
- `email` - Valid email format
- `phone` - Valid phone number format
- `url` - Valid URL format
- `number` - Numeric values
- `alphanumeric` - Letters and numbers only
- `password` - Strong password requirements

## Array & Object Utilities

### objFromArray

Converts an array of objects into a keyed object for fast lookups.

```javascript
import { objFromArray } from '/src/utils/obj-from-array';

const users = [
  { id: 1, name: "John", email: "john@example.com" },
  { id: 2, name: "Jane", email: "jane@example.com" },
  { id: 3, name: "Bob", email: "bob@example.com" }
];

// Create object keyed by 'id' (default)
const usersById = objFromArray(users);
// Result: {
//   1: { id: 1, name: "John", email: "john@example.com" },
//   2: { id: 2, name: "Jane", email: "jane@example.com" },
//   3: { id: 3, name: "Bob", email: "bob@example.com" }
// }

// Create object keyed by 'email'
const usersByEmail = objFromArray(users, 'email');
// Result: {
//   "john@example.com": { id: 1, name: "John", email: "john@example.com" },
//   "jane@example.com": { id: 2, name: "Jane", email: "jane@example.com" },
//   "bob@example.com": { id: 3, name: "Bob", email: "bob@example.com" }
// }

// Fast lookups
const user = usersById[2]; // Much faster than users.find(u => u.id === 2)
```

**Parameters:**
- `arr` - Array of objects to convert
- `key` - Property name to use as object key (default: "id")

**Use Cases:**
- Convert arrays to objects for O(1) lookups
- Create lookup tables for data normalization
- Optimize repeated array searches

### deepCopy

Creates deep copies of objects, arrays, and primitive values.

```javascript
import { deepCopy } from '/src/utils/deep-copy';

const original = {
  name: "John",
  settings: {
    theme: "dark",
    notifications: true
  },
  tags: ["admin", "user"],
  createdAt: new Date()
};

const copy = deepCopy(original);

// Modify copy without affecting original
copy.settings.theme = "light";
copy.tags.push("editor");

console.log(original.settings.theme); // "dark" (unchanged)
console.log(copy.settings.theme);     // "light"
```

**Supported Types:**
- Primitive values (string, number, boolean, null, undefined)
- Objects (plain objects)
- Arrays
- Date objects
- Nested structures

**Features:**
- Handles circular references safely
- Preserves Date objects
- Works with nested structures
- Does not copy functions or complex objects

## String Utilities

### getInitials

Extracts initials from names for avatar displays.

```javascript
import { getInitials } from '/src/utils/get-initials';

// Basic usage
getInitials("John Doe");           // "JD"
getInitials("Jane Smith Wilson");  // "JS" (first two words only)
getInitials("Madonna");            // "M"
getInitials("");                   // ""

// Handle extra spaces
getInitials("  John   Doe  ");     // "JD"

// Handle single name
getInitials("Cher");               // "C"
```

**Features:**
- Extracts first letter of first two words
- Normalizes whitespace
- Handles empty/undefined input
- Returns uppercase initials

**Use Cases:**
- Avatar placeholders when no image available
- User identification in lists
- Compact display of names

### Filter Operators

Predefined filter operator configurations for building filter UIs.

```javascript
import { 
  containsOperator,
  equalsOperator,
  isBlankOperator,
  greaterThanOperator 
} from '/src/utils/filter-operators';

const stringOperators = [containsOperator, equalsOperator, isBlankOperator];
const numberOperators = [equalsOperator, greaterThanOperator, isBlankOperator];

// Build filter UI
const FilterBuilder = ({ fieldType }) => {
  const operators = fieldType === 'string' ? stringOperators : numberOperators;
  
  return (
    <select>
      {operators.map(op => (
        <option key={op.name} value={op.name}>
          {op.label}
        </option>
      ))}
    </select>
  );
};
```

**Available Operators:**
- `containsOperator` - For string contains filtering
- `equalsOperator` - For exact match filtering
- `startsWithOperator` - For prefix filtering
- `endsWithOperator` - For suffix filtering
- `greaterThanOperator` - For numeric comparison
- `lessThanOperator` - For numeric comparison
- `isBlankOperator` - For null/undefined checking
- `isPresentOperator` - For non-null checking
- `isAfterOperator` - For date comparison
- `isBeforeOperator` - For date comparison

## System Utilities

### createResourceId

Generates cryptographically secure random IDs for resources.

```javascript
import { createResourceId } from '/src/utils/create-resource-id';

// Generate unique IDs
const id1 = createResourceId(); // "a1b2c3d4e5f6g7h8i9j0k1l2"
const id2 = createResourceId(); // "9z8y7x6w5v4u3t2s1r0q9p8o"

// Use for temporary resources
const tempFormId = createResourceId();
const cacheKey = `user-data-${createResourceId()}`;
```

**Features:**
- 24-character hexadecimal string
- Cryptographically secure random generation
- Browser-compatible (uses `window.crypto`)
- Suitable for temporary IDs and cache keys

### wait

Promise-based delay utility for async operations.

```javascript
import { wait } from '/src/utils/wait';

// Delay execution
const delayedOperation = async () => {
  console.log("Starting...");
  await wait(1000); // Wait 1 second
  console.log("Continuing after delay...");
};

// Retry with backoff
const retryOperation = async (operation, maxRetries = 3) => {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await operation();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await wait(1000 * Math.pow(2, i)); // Exponential backoff
    }
  }
};

// Rate limiting
const rateLimitedRequests = async (requests) => {
  for (const request of requests) {
    await request();
    await wait(100); // 100ms between requests
  }
};
```

**Parameters:**
- `time` - Delay in milliseconds

**Use Cases:**
- Adding delays between API requests
- Implementing retry logic with backoff
- Rate limiting operations
- Animation timing

### noop

No-operation function for default callbacks and placeholders.

```javascript
import { noop } from '/src/utils/noop';

// Default callback
const Button = ({ onClick = noop, children }) => (
  <button onClick={onClick}>
    {children}
  </button>
);

// Array operations
const callbacks = [handleSuccess, handleError, noop]; // Safe to call all

// Conditional callbacks
const maybeCallback = shouldCallBack ? actualCallback : noop;
maybeCallback(data); // Safe to call regardless
```

**Features:**
- Accepts any number of arguments
- Returns undefined
- Safe to call in any context
- Zero overhead

## Best Practices

### 1. Performance Optimization

```javascript
// Use objFromArray for repeated lookups
const usersById = useMemo(() => objFromArray(users), [users]);
const user = usersById[userId]; // O(1) vs O(n) with find()

// Batch filtering and sorting
const processedData = useMemo(() => {
  const filtered = applyFilters(rawData, filters);
  const sorted = applySort(filtered, sortBy, sortDir);
  return applyPagination(sorted, page, pageSize);
}, [rawData, filters, sortBy, sortDir, page, pageSize]);
```

### 2. Error Handling

```javascript
// Always use getCippError for consistent error messages
const handleApiError = (error) => {
  const message = getCippError(error);
  showToast({ message, type: 'error' });
};

// Validate data before processing
const processUserData = (userData) => {
  if (!getCippValidator('email')(userData.email)) {
    throw new Error('Invalid email address');
  }
  // Process valid data...
};
```

### 3. Data Transformation

```javascript
// Use formatting utilities for consistent display
const UserTable = ({ users }) => (
  <table>
    {users.map(user => (
      <tr key={user.id}>
        <td>{getCippFormatting(user.name, 'displayName', 'text')}</td>
        <td>{getCippFormatting(user.licenses, 'assignedLicenses', 'component')}</td>
        <td>{getCippFormatting(user.lastLogin, 'lastModifiedDateTime', 'component')}</td>
      </tr>
    ))}
  </table>
);
```

### 4. Type Safety

```javascript
// Use proper type checking with utilities
const safeDeepCopy = (obj) => {
  if (typeof obj !== 'object' || obj === null) {
    console.warn('deepCopy called with non-object value');
    return obj;
  }
  return deepCopy(obj);
};

// Validate inputs
const safeGetInitials = (name) => {
  if (typeof name !== 'string') {
    console.warn('getInitials called with non-string value');
    return '';
  }
  return getInitials(name);
};
```

This comprehensive utility documentation provides developers with all the tools needed for consistent data processing, formatting, and manipulation throughout the CIPP application.