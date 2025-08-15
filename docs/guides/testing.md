# CIPP Testing Guide

This guide covers testing strategies, tools, and best practices for the CIPP application. While CIPP currently uses ESLint for code quality, this guide provides recommendations for implementing comprehensive testing practices.

## Current Testing Setup

### ESLint Configuration

CIPP currently uses ESLint for code quality and consistency:

```javascript
// .eslintrc.cjs
module.exports = {
  env: {
    browser: true,
    es6: true,
    node: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:import/recommended',
    'plugin:react/recommended',
    'plugin:react/jsx-runtime',
    'plugin:react-hooks/recommended',
    'plugin:prettier/recommended',
  ],
  plugins: ['react-hooks', 'import'],
  rules: {
    'no-unused-vars': 'off',
    'react/prop-types': 'warn',
    'react/no-unescaped-entities': 'off',
  },
}
```

### Available Scripts

```json
{
  "scripts": {
    "lint": "next lint",
    "lint-fix": "next lint --fix"
  }
}
```

## Recommended Testing Strategy

### 1. Unit Testing Setup

#### Install Testing Dependencies

```bash
npm install --save-dev @testing-library/react @testing-library/jest-dom @testing-library/user-event jest jest-environment-jsdom
```

#### Jest Configuration

Create `jest.config.js`:

```javascript
const nextJest = require('next/jest')

const createJestConfig = nextJest({
  dir: './',
})

const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  testEnvironment: 'jest-environment-jsdom',
  moduleNameMapping: {
    '^/src/(.*)$': '<rootDir>/src/$1',
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx}',
    '!src/**/*.stories.{js,jsx}',
    '!src/**/*.test.{js,jsx}',
    '!src/pages/_app.js',
    '!src/pages/_document.js',
  ],
  testPathIgnorePatterns: [
    '<rootDir>/.next/',
    '<rootDir>/node_modules/',
  ],
}

module.exports = createJestConfig(customJestConfig)
```

#### Jest Setup File

Create `jest.setup.js`:

```javascript
import '@testing-library/jest-dom'

// Mock window.crypto for createResourceId
Object.defineProperty(window, 'crypto', {
  value: {
    getRandomValues: jest.fn((arr) => {
      for (let i = 0; i < arr.length; i++) {
        arr[i] = Math.floor(Math.random() * 256);
      }
      return arr;
    }),
  },
})

// Mock IntersectionObserver
global.IntersectionObserver = jest.fn(() => ({
  observe: jest.fn(),
  disconnect: jest.fn(),
  unobserve: jest.fn(),
}))

// Mock ResizeObserver
global.ResizeObserver = jest.fn(() => ({
  observe: jest.fn(),
  disconnect: jest.fn(),
  unobserve: jest.fn(),
}))
```

### 2. Component Testing

#### Basic Component Test Template

```javascript
// src/components/__tests__/CippInfoCard.test.jsx
import { render, screen } from '@testing-library/react'
import { ThemeProvider } from '@mui/material/styles'
import { createTheme } from '../../theme'
import { CippInfoCard } from '../CippCards/CippInfoCard'

const mockTheme = createTheme({
  direction: 'ltr',
  paletteMode: 'light',
  colorPreset: 'blue',
  contrast: 'normal',
})

const TestWrapper = ({ children }) => (
  <ThemeProvider theme={mockTheme}>
    {children}
  </ThemeProvider>
)

describe('CippInfoCard', () => {
  it('renders with required props', () => {
    render(
      <CippInfoCard
        label="Test Label"
        value={42}
      />,
      { wrapper: TestWrapper }
    )

    expect(screen.getByText('Test Label')).toBeInTheDocument()
    expect(screen.getByText('42')).toBeInTheDocument()
  })

  it('shows loading skeleton when isFetching is true', () => {
    render(
      <CippInfoCard
        label="Test Label"
        value={42}
        isFetching={true}
      />,
      { wrapper: TestWrapper }
    )

    // Look for skeleton elements
    expect(document.querySelector('.MuiSkeleton-root')).toBeInTheDocument()
  })

  it('renders action link when provided', () => {
    render(
      <CippInfoCard
        label="Test Label"
        value={42}
        actionLink="/test-link"
        actionText="View Details"
      />,
      { wrapper: TestWrapper }
    )

    const link = screen.getByRole('link', { name: /view details/i })
    expect(link).toHaveAttribute('href', '/test-link')
  })
})
```

#### Testing Components with API Calls

```javascript
// src/components/__tests__/UserComponent.test.jsx
import { render, screen, waitFor } from '@testing-library/react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { jest } from '@jest/globals'
import { UserComponent } from '../UserComponent'

// Mock the API call
jest.mock('../../api/ApiCall', () => ({
  ApiGetCall: jest.fn(() => ({
    data: [
      { id: 1, name: 'John Doe', email: 'john@example.com' },
      { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
    ],
    isLoading: false,
    isError: false,
    isSuccess: true
  }))
}))

const createTestQueryClient = () => new QueryClient({
  defaultOptions: {
    queries: { retry: false },
    mutations: { retry: false },
  },
})

describe('UserComponent', () => {
  let queryClient

  beforeEach(() => {
    queryClient = createTestQueryClient()
  })

  it('renders user list when data is loaded', async () => {
    render(
      <QueryClientProvider client={queryClient}>
        <UserComponent />
      </QueryClientProvider>
    )

    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument()
      expect(screen.getByText('Jane Smith')).toBeInTheDocument()
    })
  })
})
```

### 3. Hook Testing

```javascript
// src/hooks/__tests__/use-selection.test.js
import { renderHook, act } from '@testing-library/react'
import { useSelection } from '../use-selection'

describe('useSelection', () => {
  const items = [
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' },
    { id: 3, name: 'Item 3' }
  ]

  it('initializes with empty selection', () => {
    const { result } = renderHook(() => useSelection(items))
    
    expect(result.current.selected).toEqual([])
  })

  it('selects all items', () => {
    const { result } = renderHook(() => useSelection(items))
    
    act(() => {
      result.current.handleSelectAll()
    })
    
    expect(result.current.selected).toEqual(items)
  })

  it('selects single item', () => {
    const { result } = renderHook(() => useSelection(items))
    
    act(() => {
      result.current.handleSelectOne(items[0])
    })
    
    expect(result.current.selected).toEqual([items[0]])
  })

  it('deselects all items', () => {
    const { result } = renderHook(() => useSelection(items))
    
    act(() => {
      result.current.handleSelectAll()
      result.current.handleDeselectAll()
    })
    
    expect(result.current.selected).toEqual([])
  })

  it('resets selection when items change', () => {
    const { result, rerender } = renderHook(
      ({ items }) => useSelection(items),
      { initialProps: { items } }
    )
    
    act(() => {
      result.current.handleSelectAll()
    })
    
    expect(result.current.selected).toEqual(items)
    
    const newItems = [{ id: 4, name: 'Item 4' }]
    rerender({ items: newItems })
    
    expect(result.current.selected).toEqual([])
  })
})
```

### 4. Utility Function Testing

```javascript
// src/utils/__tests__/apply-filters.test.js
import { applyFilters } from '../apply-filters'

describe('applyFilters', () => {
  const testData = [
    { name: 'John Doe', age: 30, department: 'IT' },
    { name: 'Jane Smith', age: 25, department: 'HR' },
    { name: 'Bob Johnson', age: 35, department: 'IT' }
  ]

  it('returns all data when no filters applied', () => {
    const result = applyFilters(testData, [])
    expect(result).toEqual(testData)
  })

  it('filters by equals operator', () => {
    const filters = [
      { property: 'department', operator: 'equals', value: 'IT' }
    ]
    const result = applyFilters(testData, filters)
    
    expect(result).toHaveLength(2)
    expect(result.every(item => item.department === 'IT')).toBe(true)
  })

  it('filters by contains operator (case insensitive)', () => {
    const filters = [
      { property: 'name', operator: 'contains', value: 'john' }
    ]
    const result = applyFilters(testData, filters)
    
    expect(result).toHaveLength(2)
    expect(result.map(item => item.name)).toEqual(['John Doe', 'Bob Johnson'])
  })

  it('filters by greater than operator', () => {
    const filters = [
      { property: 'age', operator: 'greaterThan', value: 28 }
    ]
    const result = applyFilters(testData, filters)
    
    expect(result).toHaveLength(2)
    expect(result.every(item => item.age > 28)).toBe(true)
  })

  it('applies multiple filters (AND logic)', () => {
    const filters = [
      { property: 'department', operator: 'equals', value: 'IT' },
      { property: 'age', operator: 'greaterThan', value: 30 }
    ]
    const result = applyFilters(testData, filters)
    
    expect(result).toHaveLength(1)
    expect(result[0].name).toBe('Bob Johnson')
  })

  it('handles isBlank operator', () => {
    const dataWithNulls = [
      ...testData,
      { name: 'Test User', age: null, department: 'Finance' }
    ]
    const filters = [
      { property: 'age', operator: 'isBlank' }
    ]
    const result = applyFilters(dataWithNulls, filters)
    
    expect(result).toHaveLength(1)
    expect(result[0].name).toBe('Test User')
  })
})
```

### 5. API Integration Testing

```javascript
// src/api/__tests__/ApiCall.test.jsx
import { renderHook, waitFor } from '@testing-library/react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import axios from 'axios'
import { ApiGetCall } from '../ApiCall'

jest.mock('axios')
const mockedAxios = axios

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  })
  
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}

describe('ApiGetCall', () => {
  beforeEach(() => {
    jest.clearAllMocks()
  })

  it('makes successful API call', async () => {
    const mockData = { users: ['user1', 'user2'] }
    mockedAxios.get.mockResolvedValueOnce({ data: mockData })

    const { result } = renderHook(
      () => ApiGetCall({
        url: '/api/test',
        queryKey: 'test',
        data: { param: 'value' }
      }),
      { wrapper: createWrapper() }
    )

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true)
    })

    expect(result.current.data).toEqual(mockData)
    expect(mockedAxios.get).toHaveBeenCalledWith('/api/test', {
      signal: expect.any(AbortSignal),
      params: { param: 'value' },
      headers: { 'Content-Type': 'application/json' }
    })
  })

  it('handles API errors', async () => {
    const mockError = new Error('API Error')
    mockedAxios.get.mockRejectedValueOnce(mockError)

    const { result } = renderHook(
      () => ApiGetCall({
        url: '/api/test',
        queryKey: 'test-error',
        retry: 0 // Disable retries for test
      }),
      { wrapper: createWrapper() }
    )

    await waitFor(() => {
      expect(result.current.isError).toBe(true)
    })

    expect(result.current.error).toBe(mockError)
  })
})
```

### 6. Page Testing

```javascript
// src/pages/__tests__/index.test.jsx
import { render, screen } from '@testing-library/react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { Provider } from 'react-redux'
import { store } from '../../store'
import HomePage from '../index'

jest.mock('../../api/ApiCall')

const TestProviders = ({ children }) => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  })

  return (
    <Provider store={store}>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </Provider>
  )
}

describe('HomePage', () => {
  it('renders without crashing', () => {
    render(<HomePage />, { wrapper: TestProviders })
    
    // Test for key elements that should be present
    expect(screen.getByRole('main')).toBeInTheDocument()
  })
})
```

### 7. Integration Testing with Mock Service Worker (MSW)

#### Install MSW

```bash
npm install --save-dev msw
```

#### MSW Setup

```javascript
// src/mocks/handlers.js
import { rest } from 'msw'

export const handlers = [
  rest.get('/api/ListUsers', (req, res, ctx) => {
    return res(
      ctx.json([
        { id: 1, name: 'John Doe', email: 'john@example.com' },
        { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
      ])
    )
  }),

  rest.post('/api/AddUser', (req, res, ctx) => {
    return res(
      ctx.json({ success: true, id: 3 })
    )
  }),

  rest.get('/api/GetUser', (req, res, ctx) => {
    const userId = req.url.searchParams.get('userId')
    return res(
      ctx.json({
        id: userId,
        name: 'Test User',
        email: 'test@example.com'
      })
    )
  })
]
```

```javascript
// src/mocks/server.js
import { setupServer } from 'msw/node'
import { handlers } from './handlers'

export const server = setupServer(...handlers)
```

```javascript
// jest.setup.js (add to existing file)
import { server } from './src/mocks/server'

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

### 8. End-to-End Testing with Playwright

#### Install Playwright

```bash
npm install --save-dev @playwright/test
```

#### Playwright Configuration

```javascript
// playwright.config.js
module.exports = {
  testDir: './tests/e2e',
  timeout: 30000,
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://127.0.0.1:3000',
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],
  webServer: {
    command: 'npm run dev',
    port: 3000,
    reuseExistingServer: !process.env.CI,
  },
}
```

#### E2E Test Example

```javascript
// tests/e2e/user-management.spec.js
import { test, expect } from '@playwright/test'

test.describe('User Management', () => {
  test('should display users list', async ({ page }) => {
    await page.goto('/identity/administration/users')
    
    // Wait for the table to load
    await page.waitForSelector('[data-testid="users-table"]')
    
    // Check if table headers are present
    await expect(page.locator('text=Display Name')).toBeVisible()
    await expect(page.locator('text=User Principal Name')).toBeVisible()
  })

  test('should add new user', async ({ page }) => {
    await page.goto('/identity/administration/users/add')
    
    // Fill out the form
    await page.fill('[data-testid="displayName"]', 'Test User')
    await page.fill('[data-testid="userPrincipalName"]', 'testuser@example.com')
    
    // Submit the form
    await page.click('[data-testid="submit-button"]')
    
    // Verify success message or redirect
    await expect(page.locator('text=User created successfully')).toBeVisible()
  })
})
```

## Testing Best Practices

### 1. Test Structure

```javascript
// Follow AAA pattern: Arrange, Act, Assert
describe('Component Name', () => {
  it('should do something when condition', () => {
    // Arrange
    const props = { /* test props */ }
    
    // Act
    render(<Component {...props} />)
    
    // Assert
    expect(screen.getByText('Expected Text')).toBeInTheDocument()
  })
})
```

### 2. Test Data Management

```javascript
// Create test data factories
const createUser = (overrides = {}) => ({
  id: 1,
  name: 'Test User',
  email: 'test@example.com',
  ...overrides
})

const createUserList = (count = 3) => 
  Array.from({ length: count }, (_, i) => createUser({ id: i + 1 }))

// Use in tests
const users = createUserList(5)
```

### 3. Mock Strategies

```javascript
// Mock external dependencies at module level
jest.mock('../../api/ApiCall')
jest.mock('../../utils/get-cipp-error')

// Create reusable mock functions
const createMockApiResponse = (data, loading = false, error = null) => ({
  data,
  isLoading: loading,
  isError: !!error,
  error,
  isSuccess: !loading && !error
})
```

### 4. Custom Render Functions

```javascript
// Create custom render with providers
import { render } from '@testing-library/react'
import { AllProviders } from '../test-utils/providers'

const customRender = (ui, options) =>
  render(ui, { wrapper: AllProviders, ...options })

export { customRender as render }
export * from '@testing-library/react'
```

### 5. Accessibility Testing

```javascript
// Add accessibility tests
import { axe, toHaveNoViolations } from 'jest-axe'

expect.extend(toHaveNoViolations)

test('should not have accessibility violations', async () => {
  const { container } = render(<Component />)
  const results = await axe(container)
  expect(results).toHaveNoViolations()
})
```

## Package.json Test Scripts

Add these scripts to your `package.json`:

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "lint": "next lint",
    "lint:fix": "next lint --fix",
    "type-check": "tsc --noEmit"
  }
}
```

## CI/CD Integration

### GitHub Actions Example

```yaml
# .github/workflows/test.yml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '22.13.0'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linting
        run: npm run lint
      
      - name: Run unit tests
        run: npm run test:coverage
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

## Testing Checklist

### Before Implementing Tests

- [ ] Install testing dependencies
- [ ] Configure Jest and testing-library
- [ ] Set up MSW for API mocking
- [ ] Create test utilities and providers
- [ ] Add test scripts to package.json

### For Each Component

- [ ] Test rendering with required props
- [ ] Test user interactions
- [ ] Test error states
- [ ] Test loading states
- [ ] Test accessibility

### For Each Hook

- [ ] Test initial state
- [ ] Test state changes
- [ ] Test cleanup
- [ ] Test edge cases

### For Each Utility

- [ ] Test with valid inputs
- [ ] Test with invalid inputs
- [ ] Test edge cases
- [ ] Test error handling

This testing guide provides a comprehensive foundation for implementing robust testing practices in CIPP. Start with unit tests for critical components and utilities, then gradually add integration and end-to-end tests as needed.