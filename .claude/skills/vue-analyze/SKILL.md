---
name: vue-analyze
description: Analyze Vue 3 component structure and suggest optimizations for performance, code reuse, and best practices. Use this skill when reviewing Vue components for improvement opportunities.
---

# Vue Component Analysis Skill

Analyze Vue 3 Composition API components for performance issues, code reuse opportunities, and best practice violations. Produce a structured report with prioritized, actionable suggestions.

## When to Use

- User asks to optimize or review Vue components
- Before a major refactor of frontend code
- When adding new views that may duplicate existing patterns

## Analysis Process

### Step 1: Gather Components

Read ALL files in these locations:
- `client/src/views/*.vue`
- `client/src/components/*.vue`
- `client/src/composables/*.js`
- `client/src/api.js`

### Step 2: Run Each Analysis Check

For every component, evaluate each category below and record findings.

---

## Analysis Categories

### 1. Reactivity Correctness

Check for these anti-patterns:

| Issue | What to Look For | Fix |
|-------|-------------------|-----|
| Derived data in `ref()` | A `ref()` whose value is always recomputed from other refs | Convert to `computed()` |
| Missing `computed()` for filtered/sorted lists | `watch` or manual reassignment used to filter a list when a `computed` would suffice | Replace with `computed()` |
| Unnecessary watchers | `watch` that just copies one ref to another or triggers a recompute that `computed` handles | Remove watcher, use `computed()` |
| Mutating props | Directly modifying a prop value instead of emitting an event | Emit event, let parent own state |

### 2. Duplicate / Similar Code Across Components

Look for code that appears in two or more components and could be extracted:

- **Identical data-fetching patterns**: loading/error/try-catch blocks that follow the same shape. Suggest a `useFetchData(fetchFn)` composable if 3+ views repeat the pattern.
- **Shared computed logic**: Currency formatting, status badge mapping, date formatting functions duplicated across views. Suggest a shared utility or composable.
- **Repeated template fragments**: Identical card headers, table wrappers, empty-state blocks, or loading spinners. Suggest a shared component only if the fragment is non-trivial (>10 lines) and appears 3+ times.
- **Inline SVG icons**: The same SVG markup copied across files. Suggest extracting to a small `BaseIcon.vue` component or a lookup object.

When suggesting extraction, specify:
- Which files contain the duplicate
- The approximate lines involved
- A concrete name for the composable/component/utility

### 3. Performance

| Issue | Detection | Recommendation |
|-------|-----------|----------------|
| Large `v-for` without key | `v-for` using array index or missing `:key` | Use unique domain key (`id`, `sku`) |
| Expensive computations in template | Complex expressions (`.filter().map().reduce()`) directly in `{{ }}` or `:prop` bindings | Move to a `computed()` property |
| Missing `v-once` on static content | Large blocks of static HTML inside a component that re-renders frequently | Add `v-once` directive |
| Unnecessary re-renders from inline objects/arrays | `:style="{ width: x + '%' }"` or `:class="[a, b]"` recreated every render inside a tight `v-for` | Extract to computed or method |
| Components that should be lazy-loaded | Route-level views imported eagerly in the router | Use `defineAsyncComponent` or dynamic `import()` in router |
| Large component that should be split | A single `.vue` file with >300 lines of `<script>` or >400 lines of `<template>` | Identify natural sub-components to extract |

### 4. Props & Events Contract

- **Missing prop validation**: Props declared without `type`, `required`, or `default`. Suggest adding validation.
- **Overly broad prop types**: Props typed as `Object` or `Array` when a more specific shape is known. Suggest using `PropType<T>` or defining the shape.
- **Unused props**: Props declared but never referenced in template or script. Remove them.
- **Direct DOM manipulation**: Using `document.querySelector` or similar instead of template refs. Suggest `ref()` + `template ref`.

### 5. Composable Design

Evaluate existing composables (`useFilters`, `useAuth`, `useI18n`) and identify opportunities for new ones:

- Is there logic in a component that is not UI-specific and could be a composable?
- Are composables doing too much (>1 responsibility)?
- Are composable return values properly typed/documented?
- Could any composable benefit from accepting parameters for configurability?

### 6. Template Cleanliness

- **Long ternary expressions** in template: Move to a computed or method.
- **Deeply nested `v-if`/`v-else`**: Consider extracting inner blocks to sub-components.
- **Magic numbers/strings**: Hardcoded thresholds or values in template expressions. Extract to named constants.
- **Complex `class` bindings**: Long conditional class arrays. Extract to a computed.

---

## Output Format

Structure the report as follows:

```markdown
# Vue Component Analysis Report

## Summary
- Components analyzed: N
- Issues found: N (X critical, Y moderate, Z minor)
- Top recommendation: [one-line summary]

## Critical Issues
[Issues that cause bugs or significant performance problems]

### Issue: [Title]
- **Category**: [from categories above]
- **Files**: [file paths with line numbers]
- **Problem**: [what is wrong]
- **Suggestion**: [specific fix with code example]
- **Impact**: [what improves]

## Moderate Issues
[Issues that hurt maintainability or have minor performance cost]
[Same structure as above]

## Minor Issues / Suggestions
[Nice-to-haves and style improvements]
[Same structure as above]

## Code Reuse Opportunities
[Deduplicated patterns with suggested composables/components]

### Opportunity: [Name]
- **Pattern found in**: [list of files]
- **Suggested extraction**: [composable/component name and location]
- **Example implementation**: [brief code sketch]
```

## Severity Definitions

- **Critical**: Causes incorrect reactivity, memory leaks, broken rendering, or measurable performance degradation on typical data sizes.
- **Moderate**: Makes code harder to maintain, violates Vue best practices, or has potential performance cost at scale.
- **Minor**: Stylistic improvement, slight readability gain, or preventive optimization.

## Constraints

- Do NOT suggest adding new npm dependencies
- Do NOT suggest switching away from Composition API
- Do NOT suggest TypeScript migration (this is a JavaScript project)
- Do NOT flag patterns that are intentional per CLAUDE.md (e.g., singleton filter state is by design)
- Respect existing architecture: singleton composable pattern, scoped styles, global styles in App.vue
- Only suggest extracting a component/composable if the duplication is concrete and measurable — not speculative
