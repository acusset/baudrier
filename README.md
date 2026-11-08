A collection of [Claude Code skills](https://docs.claude.com/en/docs/claude-code/skills) for refactoring React and Next.js code, packaged as an [APM](https://microsoft.github.io/apm) package so it can be reused in other repositories.

## Using these skills in another repository

Requires [APM](https://microsoft.github.io/apm) (developed against 0.35.0).

1. In the consuming repository, create an `apm.yml` if it has none (`apm init -y`).
2. Install the package:

   ```bash
   apm install acusset/baudrier                  # deploy to the detected agent targets
   apm install acusset/baudrier --target claude  # or pick targets explicitly (comma-separated)
   ```

   This adds the package to `dependencies.apm` in your `apm.yml`, so you can also declare it by hand and run `apm install`:

   ```yaml
   dependencies:
     apm:
       - acusset/baudrier
   ```

3. The skills are deployed into the agent's skills folder: `.claude/skills/` for Claude Code, `.agents/skills/` for Copilot and Cursor. The package sources are fetched into `apm_modules/` (APM adds it to `.gitignore`).

Skills link to each other (for example `slice-large-components` points to `compound-pattern`). On install APM rewrites those relative links to point into `apm_modules/`, so keep `apm_modules/` on disk wherever the deployed skills are used.

### Keeping up to date

- `apm outdated` shows whether a newer version is available; `apm update` refreshes the dependency (`apm update --dry-run` previews it).
- Commit `apm.yml` and `apm.lock.yaml` so teammates and CI get the same version: `apm install --frozen` fails if the lockfile is out of sync.
- Whether to also commit the deployed `.claude/skills/` is your choice; if you do not, teammates run `apm install` after cloning.

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
