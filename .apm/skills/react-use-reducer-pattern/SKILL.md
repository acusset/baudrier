---
name: react-use-reducer-pattern
description: Refactor React components with sprawling local state into a single useReducer. Use when a component has more than 2 useState calls, or the user mentions "too many useState", "consolidate state", or "useReducer".
---

# React useReducer Pattern

## When to trigger

Refactor to `useReducer` when a component has **more than 2 `useState` calls** whose values are related — they change together, or one's update logic depends on another's current value.

Do NOT refactor a single `useState`, or 2 truly independent/unrelated ones.

## Workflow

1. **Inventory** every `useState` call in the component and how each setter is used.
2. **Design the state shape** as one object covering all the related fields.
3. **Design actions** as a discriminated union (`{ type: 'fieldChanged'; value: X } | { type: 'reset' } | ...`). Name actions after the event that occurred, not the state field being set (e.g. `submitted`, not `setLoadingTrue`).
4. **Write the reducer**: a pure `switch`/`if` over `action.type` returning new state. Where a single event previously required calling multiple setters, compute the full next state in one `case` so it becomes one atomic transition instead of several sequential updates.
5. **Replace** the `useState` calls with `const [state, dispatch] = useReducer(reducer, initialState)`.
6. **Update call sites**: event handlers call `dispatch({ type: ... })` instead of multiple setters; JSX reads `state.field` instead of separate variables.

## Example

See [REFERENCE.md](REFERENCE.md) for a full before/after: a form with 3 related states refactored into a single reducer.
