# Command: create-component

## Purpose

Scaffold a new React component in `packages/excalidraw/components/` that follows all project conventions: named export, CSS Module, TypeScript strict types, context hooks, and colocated test file.

## When to use

When adding a new UI component to the Excalidraw editor. Ensures the component starts with correct structure so it passes `yarn test:code` and `yarn test:typecheck` without manual fixes.

## Prompt

Create a new Excalidraw component named **$ARGUMENTS**.

Follow these steps exactly:

1. **Read** `packages/excalidraw/components/LayerUI.tsx` to understand how existing components use context hooks (`useExcalidrawAppState`, `useExcalidrawElements`) and CSS Modules.

2. **Create** `packages/excalidraw/components/$ARGUMENTS.tsx` with:
   - Functional component using hooks only (no class components)
   - Props interface named `$ARGUMENTSProps` (empty is fine if no external props)
   - Named export only: `export const $ARGUMENTS`
   - Context hooks for state access (do NOT pass AppState as props)
   - `import type` for all type-only imports
   - Import its own CSS Module: `import styles from "./$ARGUMENTS.module.scss"`

3. **Create** `packages/excalidraw/components/$ARGUMENTS.module.scss` with:
   - One root class matching the component name in camelCase
   - Use existing CSS custom properties from `packages/excalidraw/css/variables.module.scss` — do NOT introduce new color or spacing values

4. **Create** `packages/excalidraw/components/$ARGUMENTS.test.tsx` with:
   - Describe block named `$ARGUMENTS`
   - At minimum: one test "renders without crashing" using `@testing-library/react`
   - Test behavior, not implementation details

5. **Verify**: run `yarn test:typecheck` — fix any TypeScript errors before finishing.

## Example

```
/create-component ElementCoordinates
```

Creates:
- `packages/excalidraw/components/ElementCoordinates.tsx`
- `packages/excalidraw/components/ElementCoordinates.module.scss`
- `packages/excalidraw/components/ElementCoordinates.test.tsx`