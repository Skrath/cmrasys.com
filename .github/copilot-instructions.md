# Copilot / AI Agent Instructions for cmrasys.com

This repository is a Hugo-based personal site. Keep changes scoped, preserve the existing style, and prefer source edits in `hugo/` over generated output or unrelated cleanup.

## Repository layout
- `hugo/` contains the actual site source: content, config, layouts, assets, and the generated site output at `hugo/public/`.
- `public/` is another generated build artifact in the repo root; treat it as derived output unless a task explicitly calls for updating the exported site.
- `hugo/themes/cmrasys/` holds the custom theme and template logic for layouts, partials, and shortcodes.
- `hugo/content/` is the canonical source for posts, pages, and examples.
- `.github/workflows/` owns deployment and validation pipelines.

## Working conventions
- Keep edits surgical: do not reformat unrelated files or mix broad cleanup into feature work.
- Prefer small, targeted changes that match the project’s existing structure and naming.
- Avoid hand-editing generated output when the source can be changed instead.
- If a change affects the theme, content, front matter, CSS, or build output, validate the relevant Hugo build or local preview before finishing.

## Local development and validation
- Start the local site with drafts enabled:
  - `hugo server -D -s hugo/ --poll 700ms`
- Build the site for production:
  - `hugo -s hugo/ --minify`
- The repo also has editor/task wiring in `frontmatter.json` and `.vscode/tasks.json`; follow those when available.

## Content and front matter
- New pages should follow the archetype in `hugo/archetypes/default.md` unless a task explicitly needs a different structure.
- Front matter is defined in `frontmatter.json`; common keys include `title`, `date`, `lastmod`, `publishdate`, `draft`, `tags`, `categories`, `background`, and `displayClass`.
- Use `draft: true` for work-in-progress posts and `draft: false` when publishing.
- Keep page metadata consistent with existing content patterns and existing taxonomy usage.

## Theme and styling
- The custom theme lives under `hugo/themes/cmrasys/` and is the main place to look for layout and shortcode behavior.
- Styles live in `hugo/assets/sass/`; edit those source files rather than generated CSS.
- If a visual change is required, inspect the relevant partials/shortcodes before editing styles so the fix matches the site’s patterns.

## Key files to inspect first
- `hugo/config/_default/hugo.toml` — site settings, menus, taxonomies, params.
- `frontmatter.json` — schema for content metadata and workspace commands.
- `hugo/archetypes/default.md` — default front matter for new content.
- `hugo/themes/cmrasys/layouts/` — templates, partials, shortcodes.
- `hugo/assets/sass/` — site-wide SCSS source.
- `.github/workflows/live.yml` and `.github/workflows/validation.yml` — deployment and validation flow.

## Safety and deployment notes
- Treat build artifacts as derived from Hugo source; do not edit them casually.
- Deployment uses GitHub Actions and FTP credentials via repo secrets; never hardcode or expose credentials.
- Keep CI-safe assumptions in mind: any changes that affect build or CSS should still work with the Hugo build used in automation.

## Before finishing a task
- Confirm the change is aligned to the current request and not broader churn.
- Prefer matching the repository’s existing conventions over introducing new patterns.
- Run the smallest relevant validation step (for example, a Hugo build or local preview) before closing a task.
- If generated output is intentionally updated, include that in the scope of the change and keep it consistent with the source templates.
