# mui Knowledge

Generated: 2026-06-23

## OVERVIEW

`mui` is the Material UI v9 skill. It is intentionally organized around `resources/`, not `references/`, and covers sx styling, theming, responsive patterns, container queries, cascade layers, and v9 migration guidance.

## STRUCTURE

```text
mui/
|-- SKILL.md
`-- resources/
    |-- component-library.md
    |-- styling-guide.md
    |-- theme-customization.md
    `-- v9-breaking-changes.md
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Change trigger scope or quick examples | `SKILL.md` | Keep MUI v9, sx, theme, container queries, cascade layers, and migration triggers visible. |
| Edit component usage guidance | `resources/component-library.md` | Component APIs and patterns. |
| Edit sx/responsive styling guidance | `resources/styling-guide.md` | Styling conventions. |
| Edit theme/token guidance | `resources/theme-customization.md` | Theme and customization surface. |
| Edit migration guidance | `resources/v9-breaking-changes.md` | Deprecated/removed APIs and codemod notes. |

## CONVENTIONS

- Preserve `resources/` as the support-folder name for this skill; do not normalize it to `references/` without updating `skills.sh.json`.
- Keep v9-specific claims isolated from generic MUI advice.
- Treat deprecated API removal and legacy Grid migration as high-risk guidance; update migration docs together with entry-point summaries.
- Keep examples focused on MUI idioms such as `sx`, theme-aware values, and typed `SxProps<Theme>`.

## ANTI-PATTERNS

- Do not add React app scaffolding instructions here; route new-project setup to `project-starter`.
- Do not bury v9 breaking changes only in resources; surface important migration triggers in `SKILL.md`.
- Do not change support file names without updating `skills.sh.json`.
