---
name: extract-page-layout
description: Split a Next.js page.tsx that mixes structural chrome (wrapper divs, backgrounds, max-width containers, header/nav/footer) with page-specific content into two components — a layout and the content. Use when a page.tsx has wrapper markup around its actual content, or the user asks to "extract the layout", "split this page", or says a page's layout and content are tangled together.
---

# Extract Page Layout

## Goal

A `page.tsx` should describe what's *on* the route, not how the route is framed. When wrapper markup (background color, centering, max-width, header/nav/footer) lives in the same file as the page's actual content, pull the wrapper out into a layout component and leave `page.tsx` holding only content. See [REFERENCE.md](REFERENCE.md) for the full worked example.

## Step 1 — Draw the boundary: layout vs content

Read top-down and find the first element that is genuinely specific to this page (not reusable positioning). Everything above that line, plus its closing tags, is layout.

**Layout** (structural chrome — generic, would look identical on a sibling page):
- Outer wrapper `<div>`s whose classes are only background/centering/max-width/flex-shell
- `<header>`, `<nav>`, `<footer>` shared UI
- `<html>`/`<body>` (root layout only — never move these into a co-located component, see Step 2)

**Content** (page-specific — would need to change for a sibling page):
- Headings, copy, images, forms, data specific to this route
- Any element whose props/text are about *what this page is*, not *how pages are framed*

If nothing above the content-specific element does real structural work (it's just one div with no styling), there's nothing worth extracting — stop and say so rather than manufacturing a split.

## Step 2 — Decide the destination

`layout.tsx` is a special Next.js filename that wraps **every** route under that folder, not just this page — so use it only when the wrapper is genuinely shared:

- **`layout.tsx` already exists here and the wrapper is genuinely shared:** merge into it.
- **Sibling `page.tsx` files exist in subfolders (check by listing the directory, don't guess), or the wrapper is explicitly route-segment-wide chrome:** create `layout.tsx`. Next.js wires the wrapping automatically — no import needed in `page.tsx`.
- **No siblings, or the wrapper has anything page-specific, or you're unsure it should apply beyond this page:** extract into a co-located, ordinarily-named component (e.g. `foo-layout.tsx`) instead. `page.tsx` must explicitly import and wrap its content — nothing happens automatically.

Ask the user when unsure — this decision changes which routes are affected. See [REFERENCE.md](REFERENCE.md) for both cases worked out.

## Step 3 — Extract

Both paths produce a component accepting `children: React.ReactNode` and rendering the wrapper markup around `{children}`.

- **Route-level `layout.tsx`:** create/edit `layout.tsx` in the folder with the wrapper JSX, `{children}` where the content used to be. Remove that wrapper JSX from `page.tsx`, leaving only content. Move any imports/hooks used exclusively by the wrapper (e.g. a nav component, a `usePathname` call) into `layout.tsx`.
- **Co-located component:** create the new file with the same shape, then edit `page.tsx` to import it and wrap the remaining content JSX: `<FooLayout>{/* content */}</FooLayout>`.

Preserve `'use client'` correctly: if the wrapper needs interactivity (e.g. a client-side nav) but the content doesn't, or vice versa, the directive moves with whichever piece actually needs it — don't leave it on both, and don't add it to a Server Component-capable layout just because the original file had it. See the [client-components](../client-components/SKILL.md) skill for the "push 'use client' down" rule.

## Step 4 — Verify

Typecheck/build the touched route and confirm the rendered output is unchanged (same DOM structure, same styling) — this is a structural refactor, not a visual one.
