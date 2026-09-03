This is a collection of [Claude Code skills](https://docs.claude.com/en/docs/claude-code/skills) for refactoring React and Next.js code, built on top of a [Next.js](https://nextjs.org) project bootstrapped with [`create-next-app`](https://nextjs.org/docs/app/api-reference/cli/create-next-app).

## Skills

Each skill lives in `.claude/skills/<name>/` and triggers on a specific code smell:

| Skill                                                                   | Triggers on                                                                         |
| ----------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| [`client-components`](.claude/skills/client-components)                 | A component errors because it needs `'use client'` to render                        |
| [`compound-pattern`](.claude/skills/compound-pattern)                   | A monolithic component with prop drilling that fits the Compound Pattern            |
| [`extract-page-layout`](.claude/skills/extract-page-layout)             | A `page.tsx` mixing structural chrome with page-specific content                    |
| [`fetch-effect-to-swr`](.claude/skills/fetch-effect-to-swr)             | Client-side `fetch` in `useEffect` with manual loading/error `useState`             |
| [`nextjs-form-actions`](.claude/skills/nextjs-form-actions)             | A form using client-side `useState`/`useEffect` + `fetch` instead of Server Actions |
| [`pragmatic-navigation`](.claude/skills/pragmatic-navigation)           | An `onClick` handler that only calls `router.push`/`router.replace`                 |
| [`react-use-reducer-pattern`](.claude/skills/react-use-reducer-pattern) | A component with sprawling local state (more than a couple of `useState` calls)     |
| [`slice-large-components`](.claude/skills/slice-large-components)       | An oversized component/page mixing several responsibilities                         |
