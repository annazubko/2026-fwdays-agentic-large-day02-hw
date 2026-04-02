# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Excalidraw is an open-source collaborative virtual whiteboard with a hand-drawn, sketch-like aesthetic. It is built as a **layered monorepo**: the core drawing logic lives in `@excalidraw/excalidraw` (and its sub-packages), while `excalidraw-app/` is the full web application deployed at excalidraw.com.

**Two distinct use cases exist simultaneously:**
1. **Library consumer** — embedding `@excalidraw/excalidraw` in a third-party React app via npm
2. **Application developer** — working on excalidraw.com features (collab, Firebase, persistence)

Always determine which context applies before making changes. Library changes affect all consumers; app changes affect only excalidraw.com.

## Commands

```bash
# Development
yarn install           # Install all workspace dependencies
yarn start             # Dev server at http://localhost:3001
yarn start:production  # Build + serve production at http://localhost:5001

# Build
yarn build:packages    # Build all packages (required before build:app)
yarn build             # Full production build

# Testing
yarn test              # Vitest watch mode
yarn test:all          # Typecheck + lint + format check + tests
yarn test:coverage     # Vitest with coverage report

# Single test file
yarn vitest run packages/excalidraw/tests/foo.test.ts

# Code quality
yarn test:code         # ESLint (0 warnings allowed)
yarn test:typecheck    # TypeScript check only
yarn fix               # Prettier + ESLint auto-fix
```

**Package build order** (enforced by dependencies):

```
common → math → element → excalidraw
```

## Architecture

Layered monorepo — each layer only imports from layers below it:

```
excalidraw-app/           ← Application layer: collab, Firebase, persistence
  └── @excalidraw/excalidraw  ← Component layer: React UI, actions, rendering
        ├── @excalidraw/element  ← Domain layer: element model, transforms, binding
        │     ├── @excalidraw/math    ← Foundation: Point, Vector, curves (no deps)
        │     └── @excalidraw/common  ← Foundation: constants, type guards, utils
        └── @excalidraw/utils   ← Consumer utilities: export PNG/SVG
```

### Key Entry Points

| Purpose | File |
| --- | --- |
| App bootstrap | `excalidraw-app/index.tsx` |
| App component (collab, Firebase) | `excalidraw-app/App.tsx` |
| Core canvas logic | `packages/excalidraw/components/App.tsx:617` (class component) |
| Public library export | `packages/excalidraw/index.tsx` |
| Element type definitions | `packages/element/src/types.ts` |
| AppState type (~200 props) | `packages/excalidraw/types.ts:272` |
| Default AppState | `packages/excalidraw/appState.ts:22` |

### Rendering Pipeline

Two stacked canvases:

- **StaticCanvas** (bottom): grid, all scene elements → `renderStaticScene()` throttled via RAF
- **InteractiveCanvas** (top): selection, handles, cursors, snap → `renderInteractiveScene()` each frame

Shape generation uses RoughJS for hand-drawn aesthetic; shapes are cached in `ShapeCache` (WeakMap keyed by element) and invalidated on version increment.

### State Management

- **AppState**: large object (~200 props) passed through component tree; mutated via `updateScene()`
- **Scene** (`packages/element/src/Scene.ts`): owns `elementsMap: Map<id, ExcalidrawElement>`, provides `getNonDeletedElements()`, `getSelectedElements()`, `getSceneNonce()`
- **Elements are immutable**: every change creates new element with incremented `version`
- **Jotai atoms**: two isolated stores — `packages/excalidraw/editor-jotai.ts` (editor scope) and `excalidraw-app/app-jotai.ts` (app scope: collab, offline state). Do not import `jotai` directly; use the app-specific modules (ESLint enforced).

### Action System

Actions (`packages/excalidraw/actions/`) are objects with:

- `perform(elements, appState, value, app)` → `{elements, appState, captureUpdate}`
- `keyTest(event, appState, elements, app)` → boolean
- Optional `PanelComponent` for toolbar rendering

`ActionManager` (`packages/excalidraw/actions/manager.tsx:52`) handles keyboard dispatch, priority resolution, and view-mode gating.

### Collaboration

- `excalidraw-app/collab/Collab.tsx` — room lifecycle
- `excalidraw-app/collab/Portal.tsx` — Socket.io transport with AES encryption (room key in URL fragment, never sent to server)
- `packages/excalidraw/data/reconcile.ts:73` — conflict resolution: higher `version` wins; same version uses `versionNonce` as deterministic tiebreaker

### Persistence

| Store        | What                      | When                       |
| ------------ | ------------------------- | -------------------------- |
| localStorage | elements + appState JSON  | 300ms debounced            |
| IndexedDB    | images/files (idb-keyval) | on change, 24h TTL cleanup |
| Firebase     | shared scenes             | collab sessions only       |

## Code Conventions

- TypeScript strict mode; functional React components with hooks
- `PascalCase` components/types, `camelCase` variables, `ALL_CAPS` constants
- CSS Modules for all component styling
- Use `Point` type from `packages/math/src/types.ts` for all coordinates
- Imports: `@typescript-eslint/consistent-type-imports` enforced (use `import type`)

### Naming

| Kind | Convention | Example |
| --- | --- | --- |
| React component file | PascalCase | `LayerUI.tsx` |
| Utility / helper file | kebab-case | `element-utils.ts` |
| Component props type | `{Name}Props` | `type LayerUIProps = ...` |
| Constants | ALL_CAPS | `SAVE_TO_LOCAL_STORAGE_TIMEOUT` |
| Jotai atoms | camelCase + `Atom` suffix | `collabAPIAtom` |

### Exports

- **Named exports only** — `export default` is forbidden (ESLint enforced)
- Re-export public API from `packages/excalidraw/index.tsx` only — do not expose internals

### Types

- Prefer `type` over `interface` for simple shapes
- Never use `any` or `@ts-ignore` — fix the type properly
- Use `import type { X }` for all type-only imports
- For coordinates always use `Point` from `@excalidraw/math`, never `{ x: number; y: number }` inline

### State Mutations

- Elements are **immutable** — always create a new element with `mutateElement()` or spread + version bump
- AppState updates go through `actionManager.dispatch()` or `updateScene()` — never direct `setState()`
- Do not import `jotai` directly — use `packages/excalidraw/editor-jotai.ts` or `excalidraw-app/app-jotai.ts`

## What to Avoid Reading

Unless the task directly requires them:

- `node_modules/`, `packages/excalidraw/fonts/`, `packages/excalidraw/locales/`
- `packages/excalidraw/tests/fixtures/`, `**/__snapshots__/`
- `packages/excalidraw/subset/wasm/`, `scripts/wasm/`
- Build outputs: `build/`, `dist/`, `dev-dist/`, `coverage/`
- `repomix-output.xml`, `memory-bank/`

## Testing Notes

- Test runner: Vitest with jsdom environment
- Coverage thresholds: 60% lines/statements, 63% functions, 70% branches
- Run package-local tests before full-suite when changes are contained to one package

### Test Structure Rules

- Colocate tests next to the source: `ComponentName.test.tsx` beside `ComponentName.tsx`
- `kebab-case.test.ts` for utility tests, `PascalCase.test.tsx` for component tests
- Use `@testing-library/react` for components — test behavior, not implementation
- **Never snapshot-test dynamic UI** — use explicit assertions
- For actions: test both `perform()` output and `keyTest()` logic
- Cover: happy path + at least one edge case + error condition for pure functions

### Running Tests

```bash
# Watch mode (dev)
yarn test

# Single file (fastest feedback)
yarn vitest run packages/excalidraw/tests/foo.test.ts

# Full validation before commit
yarn test:all

# Coverage report
yarn test:coverage
```

## Memory Bank

The `docs/memory/` directory contains structured context files that agents should read before starting any non-trivial task:

| File | Purpose |
| --- | --- |
| `docs/memory/techContext.md` | Tech stack, tooling, build system, key dependencies — **read first** |
| `docs/memory/systemPatterns.md` | Architectural patterns, state management conventions, rendering pipeline — **read first** |
| `docs/memory/productContext.md` | Product goals, user workflows, feature priorities |
| `docs/memory/activeContext.md` | Current work in progress, recent decisions |
| `docs/memory/decisionLog.md` | Record of architectural and design decisions with rationale |
| `docs/memory/progress.md` | Task completion status and next steps |
| `docs/memory/projectbrief.md` | High-level project overview and constraints |

**Recommended reading order** before implementing features or fixes:

1. `techContext.md` — understand the stack and tooling constraints
2. `systemPatterns.md` — understand how state, rendering, and actions work
3. `activeContext.md` — understand what is currently being worked on

## Rules and Guardrails

Active rules live in `.claude/rules/`. Each rule specifies which file globs it applies to and includes a "How to verify" section with concrete commands.

| Rule file | Applies to | Key constraint |
| --- | --- | --- |
| `architecture.mdс` | `packages/excalidraw/**` | No Redux/Zustand/MobX; state via `actionManager` only |
| `conventions.mdс` | `packages/**/*.ts,tsx` | Named exports, `import type`, no `any`, kebab/PascalCase files |
| `layer-boundaries.mdс` | All packages + app | Strict one-way import chain; no upward imports |
| `security.mdс` | `excalidraw-app/collab/**` | Room key stays in URL fragment; no `eval`/`innerHTML` |
| `testing.mdс` | `**/*.test.ts,tsx` | No snapshot tests for dynamic UI; colocated test files |
| `do-not-touch.mdс` | `packages/excalidraw/**` | Protected files require explicit approval before modification |

### Protected Files

Never modify these without explicit approval and full test suite run:

- `packages/excalidraw/scene/renderer.ts` — render pipeline
- `packages/excalidraw/data/restore.ts` — file format compatibility
- `packages/excalidraw/actions/manager.ts` — action system core
- `packages/excalidraw/types.ts` — AppState and core types

### Layer Boundaries (hard constraints)

```
excalidraw-app       → may import from any @excalidraw/* package
@excalidraw/excalidraw → may import from element, math, common only
@excalidraw/element  → may import from math, common only
@excalidraw/math     → may import from common only
@excalidraw/common   → no internal @excalidraw/* imports
```

Upward imports (e.g. `element` importing from `excalidraw`) are **forbidden** and will cause TypeScript resolution errors.

## AI Assistant Expectations

These apply to Claude Code and any other AI agent working in this repo.

### Before starting any non-trivial task

1. Read `docs/memory/techContext.md` and `docs/memory/systemPatterns.md`
2. Read `docs/memory/activeContext.md` to understand current work in progress
3. Read the relevant rule files in `.claude/rules/` for the files you will touch

### When writing code

- Do not add `export default` — use named exports
- Do not use `any` or `@ts-ignore` — find the correct type
- Do not import across layer boundaries
- Do not modify protected files without confirmation
- Do not add new npm packages without explicit approval — check `packages/utils/` first
- Do not use `jotai` directly — use the project-specific atom modules
- Always use `Point` from `@excalidraw/math` for coordinate types

### When unsure

- Prefer reading existing code over guessing patterns
- Prefer smaller, focused changes over large refactors
- If a protected file needs to change, stop and ask for confirmation
- Run `yarn test:typecheck` and `yarn test:code` before declaring a task done

### What counts as done

A task is complete when:
1. `yarn test:typecheck` passes with zero errors
2. `yarn test:code` passes with zero warnings
3. `yarn vitest run <affected-test-file>` passes
4. No protected files were modified without approval
5. No new npm packages were added without approval
