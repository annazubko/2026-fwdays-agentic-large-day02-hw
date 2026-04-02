# Command: review-changes

## Purpose

Review all staged/unstaged changes (or a specific file passed as argument) against the project's rules in `.claude/rules/` before committing. Reports violations grouped by rule, with exact file and line references.

## When to use

Before `git commit` or after finishing a feature/fix. Catches rule violations — wrong imports, missing `import type`, forbidden dependencies, layer boundary breaks, security issues — before they reach CI or code review.

## Prompt

Review the current code changes for rule compliance.

$ARGUMENTS

Follow these steps:

1. **Get the diff**: run `git diff HEAD` (or `git diff --cached` if changes are staged). If a specific file was passed as argument, run `git diff HEAD -- $ARGUMENTS` instead.

2. **Load all active rules** from `.claude/rules/`:
   - `architecture.mdс` — state management, rendering, forbidden dependencies
   - `conventions.mdс` — exports, TypeScript strict mode, `import type`, file naming
   - `do-not-touch.mdс` — protected files list
   - `layer-boundaries.mdс` — forbidden cross-package imports
   - `security.mdс` — collab encryption, no eval/innerHTML, no secrets
   - `testing.mdс` — test structure, no snapshots for dynamic UI

3. **For each changed file**, check which rules apply based on the file path and the rule's `globs` field.

4. **Report findings** in this format:

   ### ✅ Compliant
   List files with no violations.

   ### ❌ Violations
   For each violation:
   - **File**: `path/to/file.tsx` line N
   - **Rule**: `rule-name.mdс`
   - **Issue**: exact description of what is wrong
   - **Fix**: concrete action to resolve it

   ### ⚠️ Warnings (non-blocking)
   Anything worth noting but not a hard rule violation.

5. **Summary line**: `N violations found in M files` — if 0 violations, confirm the changes are ready to commit.

## Example

```
/review-changes
```

Reviews all current `git diff HEAD` changes.

```
/review-changes packages/excalidraw/components/Toolbar.tsx
```

Reviews only `Toolbar.tsx` against applicable rules.