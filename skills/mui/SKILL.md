---
name: mui
description: Material-UI v9 component library patterns including sx prop styling, theme integration, responsive design, CSS container queries, cascade layers, and new components (NumberField, Menubar). Use when working with MUI v9 components, styling with sx prop, theme customization, migrating from v7, or MUI utilities.
---

# MUI v9 Patterns

## Purpose

Material-UI v9 (released April 2026) patterns for component usage, styling with sx prop, theme integration, responsive design, and new v9 features.

**Key v9 changes from v7:**
- New components: `NumberField` and `Menubar` (require `@base-ui/react`)
- CSS container queries support via `theme.containerQueries`
- CSS cascade layers via `enableCssLayer` and `modularCssLayers`
- Native color support with `InitColorSchemeScript`
- Next.js App Router and Pages Router integrations
- Browser support bumped: Chrome 117+, Edge 121+, Firefox 121+, Safari 17+
- Many deprecated APIs removed (see [resources/v9-breaking-changes.md](resources/v9-breaking-changes.md) for full list)

## When to Use This Skill

- Styling components with MUI sx prop
- Using MUI components (Box, Grid, Paper, Typography, NumberField, Menubar, etc.)
- Theme customization and usage
- Responsive design with MUI breakpoints and container queries
- CSS cascade layers and Tailwind CSS v4 integration
- Migrating from MUI v7 to v9
- MUI-specific utilities and hooks

---

## Quick Start

### Basic MUI Component

```typescript
import { Box, Typography, Button, Paper } from '@mui/material';
import type { SxProps, Theme } from '@mui/material';

const styles: Record<string, SxProps<Theme>> = {
  container: {
    p: 2,
    display: 'flex',
    flexDirection: 'column',
    gap: 2,
  },
  header: {
    mb: 3,
    fontSize: '1.5rem',
    fontWeight: 600,
  },
};

function MyComponent() {
  return (
    <Paper sx={styles.container}>
      <Typography sx={styles.header}>
        Title
      </Typography>
      <Button variant="contained">
        Action
      </Button>
    </Paper>
  );
}
```

---

## Styling Patterns

### Inline Styles (< 100 lines)

For components with simple styling, define styles at the top:

```typescript
import type { SxProps, Theme } from '@mui/material';

const componentStyles: Record<string, SxProps<Theme>> = {
  container: {
    p: 2,
    display: 'flex',
    flexDirection: 'column',
  },
  header: {
    mb: 2,
    color: 'primary.main',
  },
  button: {
    mt: 'auto',
    alignSelf: 'flex-end',
  },
};

function Component() {
  return (
    <Box sx={componentStyles.container}>
      <Typography sx={componentStyles.header}>Header</Typography>
      <Button sx={componentStyles.button}>Action</Button>
    </Box>
  );
}
```

### Separate Styles File (>= 100 lines)

For complex components, create separate style file:

```typescript
// UserProfile.styles.ts
import type { SxProps, Theme } from '@mui/material';

export const userProfileStyles: Record<string, SxProps<Theme>> = {
  container: {
    p: 3,
    maxWidth: 800,
    mx: 'auto',
  },
  header: {
    display: 'flex',
    justifyContent: 'space-between',
    alignItems: 'center',
    mb: 3,
  },
  // ... many more styles
};

// UserProfile.tsx
import { userProfileStyles as styles } from './UserProfile.styles';

function UserProfile() {
  return <Box sx={styles.container}>...</Box>;
}
```

---

## Common Components

### Layout Components

```typescript
// Box - Generic container
<Box sx={{ p: 2, bgcolor: 'background.paper' }}>
  Content
</Box>

// Paper - Elevated surface
<Paper elevation={2} sx={{ p: 3 }}>
  Content
</Paper>

// Container - Centered content with max-width
<Container maxWidth="lg">
  Content
</Container>

// Stack - Flex container with spacing
<Stack spacing={2} direction="row">
  <Item />
  <Item />
</Stack>
```

### Grid System

```typescript
import { Grid } from '@mui/material';

// 12-column grid
<Grid container spacing={2}>
  <Grid item xs={12} md={6}>
    Left half
  </Grid>
  <Grid item xs={12} md={6}>
    Right half
  </Grid>
</Grid>

// Responsive grid
<Grid container spacing={3}>
  <Grid item xs={12} sm={6} md={4} lg={3}>
    Card
  </Grid>
  {/* Repeat for more cards */}
</Grid>
```

### Typography

```typescript
<Typography variant="h1">Heading 1</Typography>
<Typography variant="h2">Heading 2</Typography>
<Typography variant="body1">Body text</Typography>
<Typography variant="caption">Small text</Typography>

// With custom styling
<Typography
  variant="h4"
  sx={{
    color: 'primary.main',
    fontWeight: 600,
    mb: 2,
  }}
>
  Custom Heading
</Typography>
```

### Buttons

```typescript
// Variants
<Button variant="contained">Contained</Button>
<Button variant="outlined">Outlined</Button>
<Button variant="text">Text</Button>

// Colors
<Button variant="contained" color="primary">Primary</Button>
<Button variant="contained" color="secondary">Secondary</Button>
<Button variant="contained" color="error">Error</Button>

// With icons
import { Add as AddIcon } from '@mui/icons-material';

<Button startIcon={<AddIcon />}>Add Item</Button>
```

---

## New in v9

### NumberField

Requires `@base-ui/react`.

```typescript
import { NumberField } from './components/NumberField'; // Copied from MUI docs

// Outlined field with min/max
<NumberField label="Quantity" min={0} max={100} defaultValue={1} />

// Small size with error state
<NumberField
  label="Amount"
  min={10}
  max={40}
  defaultValue={100}
  size="small"
  error
/>

// Spinner field
import { NumberSpinner } from './components/NumberSpinner';
<NumberSpinner label="Items" min={1} max={10} />
```

**Note:** NumberField is composed from Base UI NumberField and styled with Material UI form components (`FormControl`, `OutlinedInput`, `InputLabel`, `FormHelperText`). See the [NumberField docs](https://mui.com/material-ui/react-number-field/) for full implementation examples.

### Menubar

Requires `@base-ui/react`.

```typescript
import { Menubar } from './components/Menubar'; // Copied from MUI docs

// Basic menubar with keyboard navigation
<Menubar>
  <Menu label="File">
    <MenuItem onClick={handleNew}>New</MenuItem>
    <MenuItem onClick={handleOpen}>Open</MenuItem>
    <Divider />
    <MenuItem onClick={handleExit}>Exit</MenuItem>
  </Menu>
  <Menu label="Edit">
    <MenuItem hint="Ctrl+Z" onClick={handleUndo}>Undo</MenuItem>
    <MenuItem hint="Ctrl+Y" onClick={handleRedo}>Redo</MenuItem>
  </Menu>
</Menubar>
```

**Note:** Menubar is composed from Base UI Menubar and styled with Material UI components. See the [Menubar docs](https://mui.com/material-ui/react-menubar/) for full implementation examples.

### CSS Container Queries

Use `theme.containerQueries` or the `@<size>` shorthand in `sx`:

```typescript
// Using theme.containerQueries
<Box
  sx={{
    padding: {
      '@xs': 2,
      '@md': 4,
      '@lg': 6,
    },
  }}
>
  Responsive padding via container query
</Box>

// Named containment context
<Box sx={{ containerType: 'inline-size' }}>
  <Box
    sx={{
      width: {
        '@sm/sidebar': '100%',
        '@md/sidebar': '50%',
      },
    }}
  >
    Content
  </Box>
</Box>
```

### CSS Cascade Layers

Enable for better integration with Tailwind CSS v4 and other styling solutions:

```typescript
// Single layer (Vite/SPA)
import { StyledEngineProvider } from '@mui/material/styles';
import GlobalStyles from '@mui/material/GlobalStyles';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <StyledEngineProvider enableCssLayer>
    <GlobalStyles styles="@layer theme, base, mui, components, utilities;" />
    <App />
  </StyledEngineProvider>
);

// Multiple modular layers
const theme = createTheme({
  modularCssLayers: true,
});

// Generated layers:
// @layer mui.global, mui.components, mui.theme, mui.custom, mui.sx
```

**Note:** Enabling `modularCssLayers` may change specificity behavior for existing theme style overrides. Test thoroughly after enabling.

---

## Breaking Changes from v7 to v9

### Browser Support

Minimum versions increased:
- Chrome: 109 → 117
- Edge: new minimum 121
- Firefox: 115 → 121
- Safari: 15.4 → 17.0

### Component Changes

**Autocomplete**
- `freeSolo` type changes: `getOptionLabel` and `isOptionEqualToValue` accept `string` when `freeSolo` is true
- Listbox no longer toggles on right-click

**Backdrop**
- `aria-hidden="true"` moved from root to child element

**ButtonBase**
- `onClick` event is now `MouseEvent` instead of `KeyboardEvent` (matches expected behavior)

**Dialog & Modal**
- `keepMounted` behavior changes: content unmounts when closed unless `keepMounted` is true

**Grid (Legacy)**
- Legacy Grid (`@mui/material/Grid`) is deprecated; use Grid v2 (`@mui/material/Grid2`) or PigmentGrid

**List**
- `ListItemIcon` default min-width changed from `56px` to `36px` (uses `theme.spacing`)

**Menu & MenuList**
- DOM structure updates for accessibility

**Slider**
- Uses pointer events instead of mouse events
- Use `onPointerDown` instead of `onMouseDown` to prevent drag start

**Stepper**
- `Stepper` returns `<ol>` instead of `<div>`
- `Step` returns `<li>` instead of `<div>`

**TablePagination**
- Numbers are formatted by default using `Intl.NumberFormat`

**Tabs**
- DOM and accessibility updates

**TextField**
- When using `<TextField select />`, the underlying `InputLabel` renders `<div>` instead of `<label>`

### Theme
- `theme.shadows` type tightened
- Some default theme values may have changed

### jsdom
- Minimum jsdom version requirement increased

### Removed Deprecated APIs

Many deprecated props and CSS classes from v6/v7 have been removed. Common removals include:
- Accordion: `expanded` state classes, `TransitionProps`
- Alert: `variant="standard"` default, `iconMapping` structure
- Avatar/AvatarGroup: `imgProps`, `spacing`
- Autocomplete: `getOptionDisabled`, `getOptionLabel` signatures
- Backdrop: `components`, `componentsProps`
- Badge: `components`, `componentsProps`, `sx` on `badgeContent`
- Button: `disableElevation`, `disableFocusRipple` CSS classes
- CardHeader: `titleTypographyProps`, `subheaderTypographyProps`
- Checkbox: `components`, `componentsProps`
- Chip: `avatar`, `deleteIcon` CSS class changes
- Dialog: `components`, `componentsProps`

For a complete list of removed APIs, run the MUI codemod:
```bash
npx @mui/codemod@latest v9.0.0/deprecated-apis <path/to/folder>
```

Or see the full migration guide at [resources/v9-breaking-changes.md](resources/v9-breaking-changes.md).

---

## Theme Integration

### Using Theme Values

```typescript
import { useTheme } from '@mui/material';

function Component() {
  const theme = useTheme();

  return (
    <Box
      sx={{
        p: 2,
        bgcolor: theme.palette.primary.main,
        color: theme.palette.primary.contrastText,
        borderRadius: theme.shape.borderRadius,
      }}
    >
      Themed box
    </Box>
  );
}
```

### Theme in sx Prop

```typescript
<Box
  sx={{
    // Access theme in sx
    color: 'primary.main',          // theme.palette.primary.main
    bgcolor: 'background.paper',     // theme.palette.background.paper
    p: 2,                            // theme.spacing(2)
    borderRadius: 1,                 // theme.shape.borderRadius
  }}
>
  Content
</Box>

// Callback for advanced usage
<Box
  sx={(theme) => ({
    color: theme.palette.primary.main,
    '&:hover': {
      color: theme.palette.primary.dark,
    },
  })}
>
  Hover me
</Box>
```

---

## Responsive Design

### Breakpoints

```typescript
// Mobile-first responsive values
<Box
  sx={{
    width: {
      xs: '100%',    // 0-600px
      sm: '80%',     // 600-900px
      md: '60%',     // 900-1200px
      lg: '40%',     // 1200-1536px
      xl: '30%',     // 1536px+
    },
  }}
>
  Responsive width
</Box>

// Responsive display
<Box
  sx={{
    display: {
      xs: 'none',    // Hidden on mobile
      md: 'block',   // Visible on desktop
    },
  }}
>
  Desktop only
</Box>
```

### Responsive Typography

```typescript
<Typography
  sx={{
    fontSize: {
      xs: '1rem',
      md: '1.5rem',
      lg: '2rem',
    },
    lineHeight: {
      xs: 1.5,
      md: 1.75,
    },
  }}
>
  Responsive text
</Typography>
```

---

## Forms

```typescript
import { TextField, Stack, Button } from '@mui/material';

<Box component="form" onSubmit={handleSubmit}>
  <Stack spacing={2}>
    <TextField
      label="Email"
      type="email"
      value={email}
      onChange={(e) => setEmail(e.target.value)}
      fullWidth
      required
      error={!!errors.email}
      helperText={errors.email}
    />
    <Button type="submit" variant="contained">Submit</Button>
  </Stack>
</Box>
```

---

## Common Patterns

### Card Component

```typescript
import { Card, CardContent, CardActions, Typography, Button } from '@mui/material';

<Card>
  <CardContent>
    <Typography variant="h5" component="div">
      Title
    </Typography>
    <Typography variant="body2" color="text.secondary">
      Description
    </Typography>
  </CardContent>
  <CardActions>
    <Button size="small">Learn More</Button>
  </CardActions>
</Card>
```

### Dialog/Modal

```typescript
import { Dialog, DialogTitle, DialogContent, DialogActions, Button } from '@mui/material';

<Dialog open={open} onClose={handleClose}>
  <DialogTitle>Confirm Action</DialogTitle>
  <DialogContent>
    Are you sure you want to proceed?
  </DialogContent>
  <DialogActions>
    <Button onClick={handleClose}>Cancel</Button>
    <Button onClick={handleConfirm} variant="contained">
      Confirm
    </Button>
  </DialogActions>
</Dialog>
```

### Loading States

```typescript
import { CircularProgress, Skeleton } from '@mui/material';

// Spinner
<Box sx={{ display: 'flex', justifyContent: 'center', p: 3 }}>
  <CircularProgress />
</Box>

// Skeleton
<Stack spacing={1}>
  <Skeleton variant="text" width="60%" />
  <Skeleton variant="rectangular" height={200} />
  <Skeleton variant="text" width="40%" />
</Stack>
```

---

## MUI-Specific Hooks

### useMuiSnackbar

```typescript
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';

function Component() {
  const { showSuccess, showError, showInfo } = useMuiSnackbar();

  const handleSave = async () => {
    try {
      await saveData();
      showSuccess('Saved successfully');
    } catch (error) {
      showError('Failed to save');
    }
  };

  return <Button onClick={handleSave}>Save</Button>;
}
```

---

## Icons

```typescript
import { Add as AddIcon, Delete as DeleteIcon } from '@mui/icons-material';
import { Button, IconButton } from '@mui/material';

<Button startIcon={<AddIcon />}>Add</Button>
<IconButton onClick={handleDelete}><DeleteIcon /></IconButton>
```

---

## Best Practices

### 1. Type Your sx Props

```typescript
import type { SxProps, Theme } from '@mui/material';

// ✅ Good
const styles: Record<string, SxProps<Theme>> = {
  container: { p: 2 },
};

// ❌ Avoid
const styles = {
  container: { p: 2 }, // No type safety
};
```

### 2. Use Theme Tokens

```typescript
// ✅ Good: Use theme tokens
<Box sx={{ color: 'primary.main', p: 2 }} />

// ❌ Avoid: Hardcoded values
<Box sx={{ color: '#1976d2', padding: '16px' }} />
```

### 3. Consistent Spacing

```typescript
// ✅ Good: Use spacing scale
<Box sx={{ p: 2, mb: 3, mt: 1 }} />

// ❌ Avoid: Random pixel values
<Box sx={{ padding: '17px', marginBottom: '25px' }} />
```

---

## Additional Resources

For more detailed patterns, see:
- [resources/styling-guide.md](resources/styling-guide.md) - Advanced styling patterns
- [resources/component-library.md](resources/component-library.md) - Component examples
- [resources/theme-customization.md](resources/theme-customization.md) - Theme setup
- [resources/v9-breaking-changes.md](resources/v9-breaking-changes.md) - Complete v7 to v9 breaking changes
