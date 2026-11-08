A collection of [Claude Code skills](https://docs.claude.com/en/docs/claude-code/skills) for refactoring React and Next.js code, packaged as an [APM](https://microsoft.github.io/apm) package so it can be reused in other repositories.

## Install

Requires APM (developed against 0.35.0).

```bash
apm install acusset/baudrier              # deploy to the detected agent targets
apm install acusset/baudrier --target claude
```

Or declare it in your project's `apm.yml`:

```yaml
dependencies:
  apm:
    - acusset/baudrier
```

## Contributing

Skills live in `.apm/skills/<name>/` (`SKILL.md` plus optional `REFERENCE.md`); start from `templates/skill-template.md`. Test locally with `apm install <path-to-this-repo>` from a scratch project.

## Skills

Each skill lives in `.apm/skills/<name>/` and triggers on a specific code smell:

| Skill                                                                   | Triggers on                                                                         |
| ----------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| [`client-components`](.apm/skills/client-components)                 | A component errors because it needs `'use client'` to render                        |
| [`compound-pattern`](.apm/skills/compound-pattern)                   | A monolithic component with prop drilling that fits the Compound Pattern            |
| [`extract-page-layout`](.apm/skills/extract-page-layout)             | A `page.tsx` mixing structural chrome with page-specific content                    |
| [`fetch-effect-to-swr`](.apm/skills/fetch-effect-to-swr)             | Client-side `fetch` in `useEffect` with manual loading/error `useState`             |
| [`nextjs-form-actions`](.apm/skills/nextjs-form-actions)             | A form using client-side `useState`/`useEffect` + `fetch` instead of Server Actions |
| [`pragmatic-navigation`](.apm/skills/pragmatic-navigation)           | An `onClick` handler that only calls `router.push`/`router.replace`                 |
| [`react-use-reducer-pattern`](.apm/skills/react-use-reducer-pattern) | A component with sprawling local state (more than a couple of `useState` calls)     |
| [`slice-large-components`](.apm/skills/slice-large-components)       | An oversized component/page mixing several responsibilities                         |
| [`use-use`](.apm/skills/use-use)                                         | A Server Component that awaits data before rendering its subtree                    |
