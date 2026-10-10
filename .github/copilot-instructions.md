# Copilot / AI Agent Instructions for cmrasys.com

Quick, actionable guidance to get productive in this repository.

## Project snapshot 🔎
- Static site built with **Hugo (extended)**. Source is in `hugo/`; generated site lives in `hugo/public/` (committed).
- Theme is a local theme at `hugo/themes/cmrasys` (custom layouts/shortcodes/partials).
- Content lives under `hugo/content/` (e.g., `posts/`, `writings/`, `programming/`).

## Common developer workflows ⚙️
- Local dev server (serve drafts):
  - `hugo server -D -s hugo/ --poll 700ms` (also defined in `frontmatter.json` and a workspace task `Serve Drafts`).
- Build for production (used in CI):
  - `hugo -s hugo/ --minify` (GitHub Actions run this in `Live` workflow).
- Build with drafts for testing:
  - `hugo -D` (task: `Build Drafts`).

## Deployment & CI 🔁
- Production deploy is in `.github/workflows/live.yml`:
  - Uses `peaceiris/actions-hugo@v3` (Hugo extended) to build.
  - Deploys artifacts via `SamKirkland/FTP-Deploy-Action` to `ftp.cmrasys.com` (credentials in repo secrets).
- Staging configuration exists in `hugo/config/staging/hugo.toml`.

## CI & Dependency Automation 🔁
- **Validation workflow** (`.github/workflows/validation.yml`) runs on `push` and `pull_request`. It runs linters (stylelint) and a short-circuited `site-build` job that runs `hugo -s hugo/ --minify` (the build uses `needs: linters`). This mirrors the live pipeline and helps catch build errors before deploy.
- **Node & caching:** The validation job pins Node (v18) and uses an npm cache to speed CI; stylelint dev-deps are installed via npm. Add a minimal `package.json` with stylelint devDependencies so Dependabot and `npm ci` can be used reliably.
- **Dependabot policy suggestions:** Add the `npm` package-ecosystem (directory `/`) to monitor front-end dependencies; set `target-branch: staging`, `open-pull-requests-limit: 5`, and `labels: ["dependabot"]`. Do **not** auto-merge `github-actions` updates into `master` without review because action updates can change deploy behavior.
- **Deploy secrets & safety:** Staging and Live use FTP deploys (via `SamKirkland/FTP-Deploy-Action`) and `secrets.FTP_PASSWORD`; consider separate secrets for staging vs production and prefer SFTP/SSH for stronger security where possible.

## Key patterns & conventions 📌
- Archetype and drafts:
  - New content `hugo/archetypes/default.md` sets `draft: true` by default. Use `draft: false` and `publishdate` to publish.
- Frontmatter schema and editor integration:
  - `frontmatter.json` contains schema and fields (e.g., `displayClass` with choices `default|poem|article|multiple`) and also includes the start command.
- Frontmatter common keys used in this site:
  - `title`, `date`, `lastmod`, `publishdate`, `draft`, `tags`, `categories`, `background`, `displayClass`.
- Shortcodes & examples:
  - Shortcodes live in `hugo/themes/cmrasys/layouts/shortcodes/` (e.g., `tax_listing.html`, `infoblock.html`, `excerpt.html`). Content uses them directly (see `hugo/content/posts/*`).
- Layouts & partials:
  - Look at `hugo/themes/cmrasys/layouts/partials/` (e.g., `metadata.html`, `head.html`, `ga.html`) to understand how config params are used (GA id, og image, etc.).
- SASS & assets:
  - Styles are in `hugo/assets/sass/` and processed via Hugo Pipes; ensure Hugo extended is used for SASS compilation.

## Files to inspect for context (start here) 📁
- `hugo/config/_default/hugo.toml` — site config (menus, taxonomies, params).
- `frontmatter.json` — canonical frontmatter schema and dev start command.
- `hugo/archetypes/default.md` — new-page defaults (draft behavior).
- `hugo/themes/cmrasys/layouts/shortcodes/*` and `hugo/themes/cmrasys/layouts/partials/*` — where site look & behavior are implemented.
- `hugo/content/**` — canonical usage examples of frontmatter, shortcodes, and content structure.
- `.github/workflows/live.yml` — production build + deploy pipeline.

## Tips for making changes 🛠️
- When editing templates/partials, run the local dev server and verify changes at `http://localhost:1313/`.
- If you change styles, edit `hugo/assets/sass/*.scss` and confirm Hugo rebuilds CSS (requires extended).
- To add a new post: `hugo new -s hugo/ posts/your-title.md` (archetype will set `draft: true`).
- For publishing, set `draft: false` and set `publishdate` if needed.
- Search for `displayClass` to see how different content types map to layout classes (`baseof.html` uses it to set container class and background).

## Safety & repo-specific cautions ⚠️
- `hugo/public/` is committed; CI deploys from `hugo/public/` produced by the build step — keep generated files consistent with templates before committing public/ changes.
- The FTP deploy in `live.yml` uses secrets — do not expose or hardcode credentials.

## If you need to make a code suggestion 🤖
- Prefer making small PRs that change one area (content, templates, or assets) and include a short checklist: build locally, confirm at `localhost:1313`, and update `hugo/public/` if you intend to commit generated output.

## Recent repo updates & recommendations (Jan 2026) 🔔
- **Stylelint config**: The repo contains a `.stylelintrc.json` that extends `stylelint-config-standard-scss`. I removed the workspace `stylelint.configBasedir` so the project config is authoritative.
- **Dependencies**: Updated devDependencies in `package.json` to align with the config (`stylelint` → `^16.23.1`, `stylelint-config-standard-scss` → `^16.0.0`). Run `npm install` to generate a `package-lock.json` (commit the lockfile for reproducible installs).
- **Node engine**: Some stylelint packages recommend **Node >= 20**. The dev container currently runs Node v18 and will show engine warnings; consider upgrading the dev container Node to >=20 to match package expectations.
- **Lint findings**: Running `npx stylelint 'hugo/assets/sass/**/*.scss'` reported 9 issues in `hugo/assets/sass/main.scss` (color-function-alias-notation and property-no-deprecated). Many issues are auto-fixable with `--fix`.
- **.vscode housekeeping**: Fixed `.vscode/tasks.json` so `Serve Drafts` is the only default `test` task, and simplified `.vscode/settings.json` to rely on the project `.stylelintrc.json`.
- **.gitignore**: Added `node_modules/`, `**/node_modules/`, and common npm/yarn/pnpm debug logs. Do **not** ignore `package-lock.json` (commit it).
- **Next steps you may want**:
  - Run `npx stylelint "hugo/assets/sass/**/*.scss" --fix` and commit the fixes.
  - Run `npm install` to generate and commit `package-lock.json` (or `npm ci` in CI which requires the lockfile).
  - If there are committed `node_modules/` directories, remove them from the index with `git rm -r --cached node_modules` and commit.
  - Consider upgrading the dev container Node to v20+ to remove engine warnings.

---
If you want, I can iterate and tighten any section (examples, more file links, or explicit do/don't rules). Please point to any unclear or missing parts to update. ✅
