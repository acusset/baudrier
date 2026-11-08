---
name: slice-large-components
description: Split an oversized React component or page into smaller sub-components, hooks, and helper functions to reduce cognitive complexity — without changing what it does. Use when a component/page file is large (roughly 150+ lines), deeply nested, or mixes several responsibilities (fetching, derived state, multiple JSX regions), or the user asks to split, slice, decompose, or simplify a component/file.
---

# Slice Large Components

## Ground rule

This is a **behavior-preserving** refactor only: same props in, same render out, same effects/timing/network calls. No renaming of user-facing behavior, no styling changes, no "while I'm here" fixes. If a change would alter behavior, stop and flag it instead of making it.

## Check for a more specific skill first

Some shapes of "this file is too big" have a dedicated skill that does a better job than generic extraction. Before doing general slicing, check whether the file matches one of these — if so, use that skill instead:

| Symptom | Use instead |
|---|---|
| Wrapper markup (header/nav/footer/background/max-width shell) mixed with page content | [extract-page-layout](../extract-page-layout/SKILL.md) |
| Component takes many `render*`/`on*` props or section props (`header`, `footer`, `actions`) describing sub-regions | [compound-pattern](../compound-pattern/SKILL.md) |
| 3+ related `useState` calls that change together | [react-use-reducer-pattern](../react-use-reducer-pattern/SKILL.md) |
| `fetch` inside `useEffect` with manual `data`/`loading`/`error` state | [fetch-effect-to-swr](../fetch-effect-to-swr/SKILL.md) |

If none apply, or the file needs a mix of general extraction alongside one of these, proceed below (and still apply the matching skill for that specific part).

## Workflow

1. **Inventory responsibilities**: read the file top to bottom and list its distinct pieces — JSX regions, event handlers, derived/computed values, effects, inline constants/config, data access. A component doing 4+ of these in one function body is a slicing candidate.
2. **Group by responsibility**, not by size. Each group becomes one extraction: a JSX region → a sub-component; imperative logic → a named function; a value derived from props/state → a `useMemo`'d helper or custom hook; a large literal (lookup table, options list) → a module-level constant.
3. **Decide placement**: keep the extracted piece in the same file (as an unexported function below the main component) if it's small and used only there. Move it to its own file only if it's substantial (roughly 30+ lines) or genuinely reusable elsewhere — don't create new files speculatively.
4. **Extract mechanically**: cut the block, name it for what it *is* or *does* (not `Section1`, `Helper`, `Part2`), and pass it only the props/args it actually reads — don't forward whole objects when one field is used.
5. **Rewire the parent** to call the new piece and confirm the JSX/logic it replaces is fully removed, not duplicated.
6. **Verify nothing changed**: typecheck/build the touched files; run existing tests if present; if the component renders UI, visually compare before/after (same DOM structure, same behavior on interaction).

## Checklist

- [ ] No behavior, styling, or prop contract changed for the top-level component
- [ ] Each extracted piece has one clear responsibility and a name that reflects it
- [ ] No new abstraction introduced beyond what this file needed (no speculative generalization)
- [ ] Extracted pieces receive only the props/args they use
- [ ] New files created only for substantial or genuinely reused pieces
- [ ] Old inline code fully removed, not left duplicated or dead
- [ ] Typecheck/build passes; existing tests still pass

See [REFERENCE.md](REFERENCE.md) for a worked before/after example.
