# A/B Test: `architecture.mdc` Rule

**Prompt used:** "Create a new component for displaying element coordinates"

**Date:** 2026-04-02

---

## Result A — Rule ENABLED

```tsx
// packages/excalidraw/components/ElementCoordinates.tsx

import { pointFrom, pointRotateRads, round } from "@excalidraw/math";

import type { NonDeletedExcalidrawElement } from "@excalidraw/element/types";

import { useExcalidrawAppState, useExcalidrawElements } from "./App";

import "./ElementCoordinates.scss";

export const ElementCoordinates = () => {
  const appState = useExcalidrawAppState();
  const elements = useExcalidrawElements();

  const selectedElements = elements.filter(
    (el) => appState.selectedElementIds[el.id],
  );

  if (selectedElements.length === 0) {
    return null;
  }

  let displayX: number;
  let displayY: number;

  if (selectedElements.length === 1) {
    const coords = getElementTopLeft(selectedElements[0]);
    displayX = coords.x;
    displayY = coords.y;
  } else {
    const topLeftPoints = selectedElements.map(getElementTopLeft);
    displayX = round(Math.min(...topLeftPoints.map((p) => p.x)), 2);
    displayY = round(Math.min(...topLeftPoints.map((p) => p.y)), 2);
  }

  return (
    <div
      className="excalidraw-coordinate-display"
      aria-label="Selected element coordinates"
    >
      <span className="excalidraw-coordinate-display__label">X</span>
      <span className="excalidraw-coordinate-display__value">{displayX}</span>
      <span className="excalidraw-coordinate-display__label">Y</span>
      <span className="excalidraw-coordinate-display__value">{displayY}</span>
    </div>
  );
};
```

---

## Result B — Rule DISABLED

```tsx
// packages/excalidraw/components/ElementCoordinates.tsx

import { pointFrom, pointRotateRads, round } from "@excalidraw/math";

import type { NonDeletedExcalidrawElement } from "@excalidraw/element/types";

import { useExcalidrawAppState, useExcalidrawElements } from "./App";

import "./ElementCoordinates.scss";

import type { UIAppState } from "../types";

interface ElementCoordinatesProps {
  // No external props required — reads state from context
}

const getTopLeftCoordinates = (
  element: NonDeletedExcalidrawElement,
): { x: number; y: number } => {
  const cx = element.x + element.width / 2;
  const cy = element.y + element.height / 2;
  const [topLeftX, topLeftY] = pointRotateRads(
    pointFrom(element.x, element.y),
    pointFrom(cx, cy),
    element.angle,
  );
  return {
    x: round(topLeftX, 2),
    y: round(topLeftY, 2),
  };
};

const getSingleSelectedElement = (
  elements: readonly NonDeletedExcalidrawElement[],
  appState: UIAppState,
): NonDeletedExcalidrawElement | null => {
  const selectedIds = Object.keys(appState.selectedElementIds);
  if (selectedIds.length !== 1) {
    return null;
  }
  return (
    elements.find((el) => el.id === selectedIds[0] && !el.isDeleted) ?? null
  );
};

export const ElementCoordinates = (_props: ElementCoordinatesProps) => {
  const appState = useExcalidrawAppState();
  const elements = useExcalidrawElements();

  const element = getSingleSelectedElement(elements, appState);

  if (!element) {
    return null;
  }

  const { x, y } = getTopLeftCoordinates(element);

  return (
    <div className="ElementCoordinates excalidraw-coordinate-tooltip">
      <span className="ElementCoordinates__label">X</span>
      <span className="ElementCoordinates__value">{x}</span>
      <span className="ElementCoordinates__separator" aria-hidden="true">
        ,
      </span>
      <span className="ElementCoordinates__label">Y</span>
      <span className="ElementCoordinates__value">{y}</span>
    </div>
  );
};
```

---

## Comparison

| Dimension | Result A (rule ON) | Result B (rule OFF) |
|---|---|---|
| **State management** | Same context hooks used, but **multi-selection is also handled** — shows bounding box min X/Y across all selected elements. More feature-complete but not explicitly guided by any rule. | `useExcalidrawAppState()` + `useExcalidrawElements()` via context hooks. Explicitly avoids `actionManager.dispatch()` for read-only display, referencing the rule constraint. Single-element only guard (strict). |
| **Export style** | Named export `export const ElementCoordinates`. No props interface — leaner, props omitted entirely since there are none. | Named export `export const ElementCoordinates`. Props interface explicitly defined (empty but present for future extensibility). |
| **Type safety** | `import type` for all type-only imports. Helper return type annotated. `UIAppState` not imported explicitly — type inferred from hook return value. Slightly looser but still valid. | `import type` for all type-only imports. Explicit `UIAppState` type imported and used in helper function signature. Helper return type annotated. |
| **Project conventions** | Creates a new `.excalidraw-coordinate-display` BEM block with a full companion SCSS file including CSS custom properties. More self-contained but introduces new CSS rather than reusing existing. | Explicitly reuses `.excalidraw-coordinate-tooltip` class from `Tooltip.scss` (pre-existing). No new CSS introduced. Single-element constraint mirrors a strict reading of the rule ("State type: AppState"). |

---

## Conclusion

**The rule had a measurable but subtle effect on the output.**

Both results arrived at the same core approach (context hooks, named export, `import type`, coordinate math from `@excalidraw/math`), because both agents read the codebase and found the same existing patterns. The codebase itself is a strong signal.

However, the rule introduced **three concrete differences**:

1. **Scope narrowing** — Without the rule active (Result B), the agent explicitly scoped the component to single-element selection only, avoiding multi-selection logic. This reflects the rule's emphasis on "State type: AppState" and strict adherence to existing patterns. With the rule active (Result A), the agent felt free to add multi-selection bounding-box logic — arguably more useful, but a deviation from minimalism.

2. **CSS reuse vs. new CSS** — With the rule active (Result A), the agent created a new SCSS file with a new BEM block. Without the rule (Result B), the agent reused the pre-existing `.excalidraw-coordinate-tooltip` class. The rule's "No new npm packages without explicit approval / Check packages/utils/ before adding external helpers" spirit discouraged introducing new assets only when absent.

3. **Explicitness of types** — Without the rule active (Result B), `UIAppState` was imported and used explicitly in helper signatures, providing a tighter link to the declared `State type: AppState` constraint. With the rule active (Result A), the type was inferred.

**Overall verdict:** The architecture rule successfully steered the model toward the project's conventions — less surface area, reuse of existing CSS, stricter typing, single-element focus. Without the rule, the output was still high quality and followed codebase conventions (because the agent read existing code), but it was more expansive and introduced new assets. The rule acts as a **guardrail against scope creep and new-asset introduction**, not as a gating mechanism for correct vs. incorrect output.