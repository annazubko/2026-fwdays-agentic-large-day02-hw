# Command: generate-tests

## Purpose

Generate a complete test file for an existing source file that has no tests or insufficient coverage. Follows Excalidraw testing conventions: Vitest + jsdom, `@testing-library/react` for components, behavior-focused assertions, no snapshots for dynamic UI.

## When to use

When `yarn test:coverage` shows a file below threshold (60% lines, 63% functions, 70% branches), or when a new utility/action was added without tests. Pass the source file path as argument.

## Prompt

Generate tests for the file: **$ARGUMENTS**

Follow these steps:

1. **Read** the source file `$ARGUMENTS` fully. Identify:
   - All exported functions/components
   - Their input types and return types
   - Side effects (state updates, DOM mutations, dispatches)
   - Edge cases visible in the implementation (null checks, empty arrays, error branches)

2. **Check** if a test file already exists at the colocated path (same directory, same base name with `.test.ts` or `.test.tsx`). If it exists, read it to avoid duplicating existing tests.

3. **Determine test type**:
   - `.tsx` file with JSX → use `@testing-library/react`, `render`, `screen`, `fireEvent`
   - Pure `.ts` utility → plain Vitest `describe`/`it`/`expect` with no DOM
   - Action file (has `perform` and `keyTest`) → test `perform()` output shape and `keyTest()` boolean logic

4. **Write the test file** at the colocated path with:
   - Top-level `describe` block named after the unit under test
   - Test names starting with a verb: "returns", "renders", "throws", "calls", "handles"
   - Happy path + at least 2 edge cases per exported function
   - NO snapshot tests for components that render dynamic data
   - NO tests for implementation details (internal state, private helpers)
   - Coverage of all branches visible in the source (if/else, ternaries, null guards)

5. **Run** `yarn vitest run <test-file-path>` — all tests must pass. Fix any failures before finishing.

6. **Report**: list each test case added and confirm the run result.

## Example

```
/generate-tests packages/excalidraw/utils/export.ts
```

```
/generate-tests packages/excalidraw/components/Tooltip.tsx
```