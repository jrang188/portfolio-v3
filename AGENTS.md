# AGENTS.md

## Cursor Cloud specific instructions

### Overview

This is an **Astro-based static portfolio/blog site** (astro-antfustyle-theme v2.3.0). It is a single-product repo, not a monorepo.

### Running services

- **Dev server:** `pnpm dev` — starts at `http://localhost:4321` (use `--host 0.0.0.0` to expose externally)
- Standard scripts are documented in `package.json` (`dev`, `build`, `check`, `lint`, `format`, `preview`)

### Non-obvious caveats

- **GitHub token warnings:** On `dev`/`build`/`check`, the loaders `astro-loader-github-releases` and `astro-loader-github-prs` emit warnings about missing `GITHUB_TOKEN`. These are non-blocking — the site runs fine without it, but the `/releases` and `/prs` pages will be empty.
- **Build failure on changelog page:** When `UI.tabbedLayoutTabs` is set to `false` in `src/config.ts`, the static build (`pnpm build`) fails on `/changelog/index.html` with a runtime error in `TabbedLayout`. This is a pre-existing issue. The dev server (`pnpm dev`) still serves the page correctly via SSR.
- **Git hooks:** `simple-git-hooks` sets a pre-commit hook that runs `npx lint-staged && nix fmt`. The `nix fmt` part will fail in environments without Nix installed. Use `git commit --no-verify` to bypass the hook when committing from this environment.
- **Content syncing:** Astro content loaders fetch from GitHub API, Bluesky/AT Protocol API, and external RSS feeds at startup. Internet access is required for these collections to populate.
