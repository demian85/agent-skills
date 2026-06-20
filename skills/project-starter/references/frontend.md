# Frontend Setup Reference

Complete guide for setting up a React + Vite + MUI frontend project using the project starter stack.

## Dependencies

Before installing, verify current stable versions with `npm view <package> version`. The commands below intentionally omit versions so npm resolves the latest stable release at scaffold time.

### Production

```bash
npm install react react-dom react-router @tanstack/react-query @tanstack/react-query-devtools
npm install @mui/material @mui/icons-material @emotion/react @emotion/styled
npm install @fontsource/roboto
npm install zod luxon lodash-es
```

### Development

```bash
npm install -D typescript @types/react @types/react-dom @types/luxon
npm install -D vite @vitejs/plugin-react
npm install -D eslint @eslint/js typescript-eslint eslint-config-prettier eslint-plugin-react eslint-plugin-react-hooks
npm install -D prettier
npm install -D vitest @vitest/coverage-v8 happy-dom
npm install -D @tsconfig/vite-react @tsconfig/strictest
```

## Config Files

### tsconfig.json

```json
{
  "extends": ["@tsconfig/vite-react/tsconfig.json", "@tsconfig/strictest/tsconfig.json"],
  "compilerOptions": {
    "verbatimModuleSyntax": true,
    "noEmit": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*", "tests/**/*"],
  "exclude": ["node_modules", "dist", "build"]
}
```

### vite.config.ts

```typescript
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
  build: {
    target: 'es2022',
    outDir: 'dist',
    sourcemap: true,
  },
  server: {
    port: 5173,
    open: true,
  },
  test: {
    globals: true,
    environment: 'happy-dom',
    include: ['tests/**/*.{test,spec}.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.d.ts',
        '**/__mocks__/**',
      ],
    },
  },
})
```

### eslint.config.mjs (React)

```javascript
// @ts-check
import eslint from '@eslint/js'
import tseslint from 'typescript-eslint'
import eslintConfigPrettier from 'eslint-config-prettier'
import reactPlugin from 'eslint-plugin-react'
import reactHooksPlugin from 'eslint-plugin-react-hooks'
import { defineConfig } from 'eslint/config'

export default defineConfig(
  {
    ignores: [
      'dist/',
      'build/',
      'node_modules/',
      'coverage/',
    ],
  },
  eslint.configs.recommended,
  ...tseslint.configs.recommended,
  ...tseslint.configs.recommendedTypeChecked,
  {
    files: ['**/*.{ts,tsx}'],
    languageOptions: {
      parserOptions: {
        project: './tsconfig.json',
        tsconfigRootDir: import.meta.dirname,
      },
    },
  },
  {
    files: ['**/*.{jsx,tsx}'],
    plugins: {
      react: reactPlugin,
      'react-hooks': reactHooksPlugin,
    },
    languageOptions: {
      parserOptions: {
        ecmaFeatures: {
          jsx: true,
        },
      },
    },
    rules: {
      ...reactPlugin.configs.recommended.rules,
      ...reactHooksPlugin.configs.recommended.rules,
      'react/react-in-jsx-scope': 'off',
      'react/jsx-uses-react': 'off',
    },
    settings: {
      react: {
        version: 'detect',
      },
    },
  },
  eslintConfigPrettier,
)
```

### index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>My App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

### src/main.tsx

```typescript
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import { RouterProvider } from 'react-router'
import '@fontsource/roboto/300.css'
import '@fontsource/roboto/400.css'
import '@fontsource/roboto/500.css'
import '@fontsource/roboto/700.css'
import { router } from './routes'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      refetchOnWindowFocus: false,
    },
  },
})

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <QueryClientProvider client={queryClient}>
      <RouterProvider router={router} />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  </StrictMode>
)
```

### src/routes/index.tsx

```typescript
import { createBrowserRouter } from 'react-router'
import { RootLayout } from '../components/RootLayout'
import { HomePage } from '../pages/HomePage'
import { ErrorPage } from '../pages/ErrorPage'

export const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    errorElement: <ErrorPage />,
    children: [
      {
        index: true,
        element: <HomePage />,
      },
    ],
  },
])
```

## MUI Setup Notes

- Import components from `@mui/material` (tree-shaking works with Vite)
- Use the `sx` prop for inline styling; prefer extracting to `SxProps<Theme>` objects for reusable styles
- Check the current MUI peer dependency requirements before choosing React and `react-is` versions
- Roboto font is required — import all four weights (300, 400, 500, 700) in your entry point

## React Router Notes

- Verify the current React Router major before scaffolding. Current modern versions use the `react-router` package for DOM routing.
- Use `createBrowserRouter` + `RouterProvider` for data API features (loaders, actions)
- `BrowserRouter` is still available for simpler use cases but lacks data API features
