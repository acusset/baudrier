---
name: compound-pattern
description: Identifies React components that are good candidates for the Compound Pattern and refactors them using Context API — turning monolithic components with prop drilling into composable sub-component APIs. Use when user wants to apply the Compound Pattern, refactor a React component into sub-components, or mentions "compound component", "component composition", or "prop drilling".
---

# Compound Pattern Refactor

## Quick start

Two modes:
1. **Scan** — find candidates in the codebase, ranked by suitability
2. **Refactor** — convert a specific component to the Compound Pattern

## Identification workflow

When the user hasn't named a specific component, scan for candidates:

1. Search for components with 4+ props that describe sub-regions (`title`, `header`, `footer`, `label`, `content`, `actions`, `icon`)
2. Search for components that accept multiple render prop or callback variants (`renderX`, `onX`)
3. Look for components where siblings share state (parent holds state just to pass it down)
4. Score each candidate: `+1` per prop that maps to a visual section, `+1` for shared state, `+1` for complex conditional rendering inside the component
5. Present top candidates ranked by score, with one-line rationale each
6. Ask the user which one to refactor

**Do not** flag components that are purely presentational, have only 1–2 configurable parts, or whose children are fully independent (no shared state needed).

## Refactoring workflow

Given a target component:

1. **Read the component** — identify: shared state, which children consume which state, current prop interface
2. **Plan sub-components** — map each logical section to a named sub-component (e.g. `Modal.Header`, `Modal.Body`, `Modal.Footer`)
3. **Create context** — one context per compound root; include only state/handlers that children actually need
4. **Build the root** — root component owns state, wraps children in the context provider
5. **Build sub-components** — each consumes context via `useContext`; throw a clear error if used outside the root
6. **Attach sub-components** — use `Object.assign` (works for both JS and TS); see [REFERENCE.md](REFERENCE.md) for templates
7. **Update call sites** — rewrite all usages to the new API
8. **Match source language** — if the file is `.ts`/`.tsx`, type the compound component; see [REFERENCE.md](REFERENCE.md) for TS template

## Checklist

- [ ] Context throws a descriptive error when consumed outside the root
- [ ] Sub-components have `displayName` set for React DevTools
- [ ] Root component's original prop interface is removed or slimmed down
- [ ] All call sites updated
- [ ] No prop drilling remains inside the component tree

See [REFERENCE.md](REFERENCE.md) for code templates and a before/after example.
