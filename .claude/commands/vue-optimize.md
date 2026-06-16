---
description: Analyze Vue component structure and suggest optimizations for performance and code reuse
---

Analyze the Vue 3 components in this project for performance bottlenecks and code reuse opportunities. Produce a prioritized report with concrete, actionable suggestions — not generic advice.

---

## Phase 1 — Inventory the components

Read every `.vue` file under `client/src/` (views and components). For each file record:
- Component name and file path
- `<script setup>` vs Options API
- All `ref()`, `reactive()`, `computed()`, `watch()`, `watchEffect()` calls
- Props, emits, and slots defined
- Any `v-for` loops and whether they use `:key`
- Any async data fetching (API calls, `await`, `onMounted`)
- Imports shared across multiple files (api.js calls, composables, utilities)

Also read `client/src/api.js` to map which endpoints each component calls.

---

## Phase 2 — Identify performance issues

Look for these specific patterns and flag each occurrence with file path and line number:

**Reactivity overhead**
- `reactive()` used for a flat object that could be separate `ref()`s, or vice versa
- `watch()` with no `{ immediate }` / `{ deep }` justification on complex objects
- Computed properties with side effects (anything other than a pure derivation)
- Missing `computed()` where a value is re-derived in the template on every render (e.g., `.filter()`, `.map()`, `.sort()` directly in the template)

**Rendering inefficiency**
- `v-for` without `:key`, or `:key="index"` on a list that can reorder
- Large lists rendered without virtual scrolling consideration (flag lists > ~50 items)
- Heavy components not wrapped in `<Suspense>` or lazily imported via `defineAsyncComponent`
- Watchers that trigger API calls without debouncing

**Prop drilling / tight coupling**
- The same data fetched independently in multiple sibling components (could be hoisted or shared via composable)
- Props passed more than two levels deep

---

## Phase 3 — Identify code reuse opportunities

Look for these patterns:

**Duplicated logic**
- The same API call made in more than one component — candidate for a shared composable
- Identical or near-identical filter/sort/transform logic repeated across views
- Copy-pasted `onMounted` + `ref` + `loading/error` scaffolding — extract to a `useFetch` composable

**Duplicated markup**
- Table or list structures that differ only in column definitions — candidate for a generic `<DataTable>` component
- Modal scaffolding (overlay, close button, header/body/footer slots) repeated — candidate for a `<BaseModal>` wrapper
- Card/stat-box patterns repeated — candidate for a `<StatCard>` component

**Composable extraction candidates**
- Any `ref` + `watch` + side-effect block that appears in more than one component
- Filter state management (if filters are read/written in multiple places)

---

## Phase 4 — Produce the report

Output a structured report with three sections:

### Performance Issues
For each finding:
- **File**: path:line
- **Issue**: one sentence describing the problem
- **Fix**: one sentence describing the concrete change (include the exact API/pattern to use)
- **Priority**: High / Medium / Low

### Code Reuse Opportunities
For each finding:
- **Files affected**: list the components involved
- **Pattern**: what is duplicated
- **Refactor**: what to extract (composable name, component name) and where to put it (`client/src/composables/` or `client/src/components/`)
- **Effort**: Small (< 30 min) / Medium (< 2 hrs) / Large (half day+)

### Quick Wins
List the top 3 changes that offer the best impact-to-effort ratio. These should be actionable immediately without a broader refactor.

---

## Rules

- Cite specific file paths and line numbers for every finding. Do not give generic advice without a concrete location.
- Do not suggest changes that require adding new dependencies.
- Do not flag patterns that are intentional and clearly correct (e.g., a `watch` that is documented as intentional).
- If a pattern appears in more than three files, report it once with all affected files listed rather than as separate findings.
