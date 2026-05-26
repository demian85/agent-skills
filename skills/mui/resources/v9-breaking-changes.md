# MUI v9 Breaking Changes Reference

Complete reference of breaking changes when migrating from Material UI v7 to v9.

## Browser Support

Minimum browser versions increased in v9:

| Browser | v7 Minimum | v9 Minimum |
|---------|------------|------------|
| Chrome | 109 | 117 |
| Edge | — | 121 |
| Firefox | 115 | 121 |
| Safari (macOS/iOS) | 15.4 | 17.0 |

## Component Breaking Changes

### Autocomplete

**Listbox toggle on right click**: The listbox no longer toggles open/closed on right-click. Only left-click triggers toggle.

**`freeSolo` type changes**: When `freeSolo` is `true`, `getOptionLabel` and `isOptionEqualToValue` now accept `string` types:

```typescript
// Before (v7)
isOptionEqualToValue?: (option: Value, value: Value) => boolean;
getOptionLabel?: (option: Value | AutocompleteFreeSoloValueMapping<FreeSolo>) => string;

// After (v9)
isOptionEqualToValue?: (option: Value, value: AutocompleteValueOrFreeSoloValueMapping<Value, FreeSolo>) => boolean;
getOptionLabel?: (option: AutocompleteValueOrFreeSoloValueMapping<Value, FreeSolo>) => string;
```

### Backdrop

- `aria-hidden="true"` moved from the root element to the child element

### ButtonBase

- The `event` passed to `onClick` is now a `MouseEvent` instead of the `KeyboardEvent` captured in keyboard handlers. This matches expected behavior.

### Dialog & Modal

- `keepMounted` behavior changed: content now unmounts when closed unless `keepMounted` is explicitly set to `true`

### Grid (Legacy)

- Legacy Grid (`@mui/material/Grid`) is deprecated. Use Grid v2 (`@mui/material/Grid2`) or PigmentGrid instead.

### List

- `ListItemIcon` default min-width changed from `56px` to `36px` to be consistent with menu items. Now uses `theme.spacing` instead of a hardcoded number.

### Menu & MenuList

- DOM structure updated for improved accessibility and platform alignment

### Slider

- Uses pointer events instead of mouse events
- To prevent a drag from starting, use `onPointerDown` instead of `onMouseDown`:

```typescript
// Before (v7)
<Slider onMouseDown={(event) => event.preventDefault()} />

// After (v9)
<Slider onPointerDown={(event) => event.preventDefault()} />
```

### Stepper, Step, and StepButton

- `Stepper` returns an `<ol>` element instead of `<div>`
- `Step` returns an `<li>` element instead of `<div>`
- Improves semantic structure and accessibility

### TablePagination

- Numbers are now formatted by default using `Intl.NumberFormat`

### Tabs

- DOM structure and accessibility updates

### TextField

- When using `<TextField select />`, the underlying `InputLabel` renders a `<div>` instead of a native `<label>` element. This does not affect `InputLabel` used on its own.

## Theme Changes

- `theme.shadows` type tightened
- Some default theme values may have changed

## jsdom Support

- Minimum jsdom version requirement increased

## Deprecated APIs Removed

The following deprecated APIs have been removed in v9. Run the codemod to auto-fix most of these:

```bash
npx @mui/codemod@latest v9.0.0/deprecated-apis <path/to/folder>
```

### Accordion

- `expanded` state classes removed
- `TransitionProps` removed

### AccordionSummary

- Several CSS classes renamed/removed

### Alert

- `variant="standard"` behavior changed
- `iconMapping` structure updated
- CSS classes renamed

### Avatar & AvatarGroup

- `imgProps` removed
- `spacing` prop changes

### Autocomplete

- `getOptionDisabled` removed
- `getOptionLabel` signatures updated

### Backdrop

- `components` and `componentsProps` removed (use `slots` and `slotProps`)

### Badge

- `components` and `componentsProps` removed
- `sx` on `badgeContent` removed

### Button

- `disableElevation` and `disableFocusRipple` CSS classes removed

### ButtonGroup

- CSS classes renamed

### CardHeader

- `titleTypographyProps` and `subheaderTypographyProps` removed

### Checkbox

- `components` and `componentsProps` removed

### Chip

- `avatar` and `deleteIcon` CSS class changes

### CircularProgress

- CSS classes renamed

### Dialog

- `components` and `componentsProps` removed

## System Props Codemod

If you were using system props directly on components, run:

```bash
npx @mui/codemod@latest v9.0.0/system-props <path/to/folder>
```

To target specific components:

```bash
npx @mui/codemod@latest v9.0.0/system-props <path/to/folder> -- --jsx=Box,Typography,Stack,Link,Grid,DialogContentText
```

## Grid v2 Migration

If still using the legacy Grid, migrate to Grid v2:

```bash
npx @mui/codemod@latest v9.0.0/grid-v2-props <path/to/folder>
```

## Additional Resources

- [Official v9 Migration Guide](https://mui.com/material-ui/migration/upgrade-to-v9/)
- [MUI v9 Blog Post](https://mui.com/blog/introducing-mui-v9/)
- [Codemods Repository](https://github.com/mui/material-ui/tree/master/packages/mui-codemod)
