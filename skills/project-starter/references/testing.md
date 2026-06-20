# Testing Setup Reference

Complete guide for configuring Vitest with this tech stack.

## Dependencies

Verify current stable versions first:

```bash
for package in vitest @vitest/coverage-v8 jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event; do
  npm view "$package" version
done
```

Then install the packages needed for the selected project:

```bash
npm install -D vitest @vitest/coverage-v8 jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

For backend-only projects, omit `jsdom` and `@testing-library/*` packages.

## Vite Configuration for Testing

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'node:path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  test: {
    globals: true,
    environment: 'jsdom',
    include: ['tests/**/*.{test,spec}.{ts,tsx}', 'src/**/*.{test,spec}.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.d.ts',
        '**/__mocks__/**',
        '**/index.ts',
        '**/main.tsx',
      ],
    },
    setupFiles: ['./tests/setup.ts'],
  },
})
```

## Test Setup File

```typescript
// tests/setup.ts
import '@testing-library/jest-dom/vitest'
import { cleanup } from '@testing-library/react'
import { afterEach } from 'vitest'

// Clean up after each test (DOM + React Testing Library)
afterEach(() => {
  cleanup()
})
```

## Writing Tests

### Unit Test Example

```typescript
// src/utils/calculate.test.ts
import { describe, it, expect } from 'vitest'
import { calculateTotal } from './calculate'

describe('calculateTotal', () => {
  it('sums items correctly', () => {
    const items = [{ price: 10, quantity: 2 }, { price: 5, quantity: 1 }]
    expect(calculateTotal(items)).toBe(25)
  })

  it('returns 0 for empty array', () => {
    expect(calculateTotal([])).toBe(0)
  })
})
```

### React Component Test Example

```typescript
// src/components/Button.test.tsx
import { describe, it, expect, vi } from 'vitest'
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from './Button'

describe('Button', () => {
  it('renders with label', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByText('Click me')).toBeInTheDocument()
  })

  it('calls onClick when clicked', () => {
    const handleClick = vi.fn()
    render(<Button onClick={handleClick}>Click me</Button>)
    fireEvent.click(screen.getByText('Click me'))
    expect(handleClick).toHaveBeenCalledOnce()
  })
})
```

### Async Test Example

```typescript
// src/queries/useUsers.test.ts
import { describe, it, expect } from 'vitest'
import { renderHook, waitFor } from '@testing-library/react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { useUsers } from './useUsers'

function createWrapper() {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  })
  return function Wrapper({ children }: { children: React.ReactNode }) {
    return <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  }
}

describe('useUsers', () => {
  it('fetches users', async () => {
    const { result } = renderHook(() => useUsers(), {
      wrapper: createWrapper(),
    })

    await waitFor(() => expect(result.current.isSuccess).toBe(true))
    expect(result.current.data).toBeDefined()
  })
})
```

## Running Tests

```bash
# Run tests in watch mode (default)
npm run test

# Run tests once (CI)
npm run test:run

# Run with coverage
npm run coverage

# Run specific file
npx vitest run src/components/Button.test.tsx
```

## Coverage Configuration

The `vite.config.ts` coverage block above is usually sufficient. For fine-grained control, create `vitest.config.ts`:

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import { mergeConfig } from 'vite'
import viteConfig from './vite.config'

export default mergeConfig(
  viteConfig,
  defineConfig({
    test: {
      coverage: {
        thresholds: {
          lines: 80,
          functions: 80,
          branches: 70,
          statements: 80,
        },
      },
    },
  })
)
```

## Mocking in Vitest

```typescript
// Mock a module
vi.mock('./api', () => ({
  fetchUser: vi.fn().mockResolvedValue({ id: '1', name: 'Test' }),
}))

// Mock global fetch
globalThis.fetch = vi.fn().mockResolvedValue({
  json: vi.fn().mockResolvedValue({ data: [] }),
  ok: true,
})

// Spy on console
const consoleSpy = vi.spyOn(console, 'error').mockImplementation(() => {})

// Restore all mocks after test
afterEach(() => {
  vi.restoreAllMocks()
})
```
