# samuelduchesne.github.io Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the Jekyll résumé site with a static Astro site: a quiet, text-led "research notebook" home page whose index of work is the page, with schema-checked content that agents maintain through repo skills.

**Architecture:** Astro static build. Content lives in Markdown/YAML content collections validated by Zod schemas kept in `src/lib/schemas.ts` (so they are unit-testable outside Astro). Pure functions in `src/lib/` assemble the index and fetch GitHub metadata with committed fallbacks. Pages are thin templates over those functions. A post-build checker (`scripts/check-dist.mjs`) enforces the anti-goals on the built HTML. GitHub Actions builds, checks and deploys to Pages on push and weekly.

**Tech Stack:** Node 22, Astro `^7.4` (latest is 7.4.12), TypeScript, Vitest, Playwright (pre-installed; use `require('playwright')`, do not install browsers), `yaml` npm package for the refresh script.

**Spec:** `docs/superpowers/specs/2026-11-13-site-redesign-design.md`. Read it before starting any task; its Anti-goals section is binding.

## Global Constraints

- Branch: `feature/focused-albattani-9vhaey`. Do not push elsewhere.
- Site URL `https://samuelduchesne.github.io`, Astro `site` set to it, no `base`. `trailingSlash: 'always'`, `build.format: 'directory'`.
- Routes the site may emit, and no others: `/`, `/cv/`, `/work/<slug>/`, `/writing/` (only when ≥1 writing entry), `/404.html`, plus static files under `/images/` and `/_astro/`.
- Home page: no images, no figures, no animation, no client JS. The **only** script allowed on any page is Google Analytics `G-JPF0EN6DC1` (gtag), in the shared layout.
- No email address and no `mailto:` anywhere in the built site. Contact links: LinkedIn `https://www.linkedin.com/in/samuel-letellier-duchesne-40670538`, GitHub `https://github.com/samuelduchesne`, ORCID `https://orcid.org/0000-0001-5790-878X`, ResearchGate `https://www.researchgate.net/profile/Samuel-Letellier-Duchesne`.
- No colophon, no "built with" credit, no AI framing in copy, no chatbot, skill bars, logo walls, carousels or scroll arrows.
- Private projects (`private: true`) never render a repo link or stars.
- Look: warm off-white background (≈`#f6f4ef`), near-black text, serif display headline, sans body, monospace only for labels/metadata/captions, one accent colour for links. Dark mode via `prefers-color-scheme: dark`: warm charcoal, not `#000`. Colours as CSS custom properties on `:root`.
- Headline copy (owner may reword later): "Building physics," / *"written as software."* Sub-line: "Simulation engineer, open-source author, co-founder of Carbon Signal."
- Profile photo: on `/cv/` only (small), and as the `og:image`. Never on `/`.
- Keep `/images/favicon.ico` at that path.
- Commit messages end with the two attribution lines from the session (`Co-Authored-By: …`, `Claude-Session: …`). No model names in commits or files.

## Review Focus

1. **GitHub API unavailable or rate-limited** (local builds have no token; 60 req/h unauthenticated): the build must succeed using the fallback `stars`/`updated` in each project file. Test in Task 3.
2. **A private project accidentally given a `repo`**: schema must reject it so a private link can never ship. Test in Task 2.
3. **Zero writing entries** (the launch state): no Writing group on `/`, no `/writing/` page, no Writing nav link. Tests in Task 2 and Task 5.
4. **Narrow phone (375 px) with the longest claim**: rows wrap, no horizontal scroll. Claim length capped at 90 chars by schema (Task 2); Playwright check in Task 5.
5. **Personal details leaking into output** (`mailto:`, `@me.com`, old email in CV data or a skill-added entry): `check-dist` fails the build. Test in Task 5.

---

## File Structure

```
astro.config.mjs            site, trailingSlash, build format
package.json                scripts: dev, build, test, check, refresh
tsconfig.json               extends astro/tsconfigs/strict
vitest.config.ts
src/content.config.ts       collections wired to schemas (thin)
src/lib/schemas.ts          Zod schemas: project, writing, groups, cv
src/lib/index.ts            buildIndex(): home-page groups
src/lib/github.ts           fetchRepoMeta(): stars/updated with fallback
src/lib/format.ts           formatMonth(), formatStars()
src/content/groups.yaml     group ids, labels, order
src/content/projects/*.md   one per project (frontmatter + body)
src/content/writing/        empty at launch (.gitkeep)
src/content/cv.yaml         roles, education, awards, affiliations, links
src/layouts/Base.astro      head, GA, nav, footer, tokens
src/styles/global.css       tokens, type, layout, dark mode
src/components/IndexRow.astro
src/components/Figure.astro numbered figure with caption
src/pages/index.astro
src/pages/cv.astro
src/pages/work/[slug].astro
src/pages/[writing]/index.astro   emits /writing/ only if entries exist
src/pages/404.astro
public/images/favicon.ico, profile_photo_bw.jpg
scripts/check-dist.mjs      post-build anti-goal checks
scripts/check-phone.mjs     Playwright 375 px overflow check
scripts/check-routes.mjs    collision check against repos with Pages
scripts/refresh-meta.mjs    writes fresh stars/updated into frontmatter
tests/*.test.ts
CLAUDE.md
.claude/skills/{add-project,add-writing,refresh}/SKILL.md
.github/workflows/deploy.yml
```

Deleted: `_config.yml`, `Gemfile`, `index.md`, `assets/` (after Task 4 moves awards into `cv.yaml`), `images/landscape-trees.jpg`.

---

### Task 1: Astro scaffold replaces Jekyll

**Files:**
- Create: `package.json`, `astro.config.mjs`, `tsconfig.json`, `vitest.config.ts`, `src/pages/index.astro` (placeholder), `public/images/`
- Delete: `_config.yml`, `Gemfile`, `index.md`, `images/landscape-trees.jpg`
- Move: `images/favicon.ico`, `images/profile_photo_bw.jpg` → `public/images/`
- Modify: `.gitignore` (add `node_modules/`, `dist/`, `.astro/`; keep `.superpowers/`)

**Interfaces:**
- Produces: npm scripts `dev`, `build` (`astro build`), `test` (`vitest run`), `check` (`node scripts/check-dist.mjs && node scripts/check-phone.mjs`, wired in Task 5), `refresh` (Task 6).

- [ ] **Step 1:** `npm init -y`, then `npm i astro@^7.4` and `npm i -D vitest typescript @astrojs/check yaml`. Confirm Astro 7's content-layer API (`defineCollection`, `glob`/`file` loaders from `astro/loaders`, `z` from `astro/zod`) against the installed package's docs/types before Task 2; if names differ, use the installed ones and note it in the commit.
- [ ] **Step 2:** Config per Global Constraints. Placeholder `index.astro` renders `<h1>ok</h1>`.
- [ ] **Step 3:** Run `npm run build`. Expected: `dist/index.html` and `dist/images/favicon.ico` exist.
- [ ] **Step 4:** Commit `chore: replace Jekyll with Astro scaffold`. Keep `assets/` for now (Task 4 reads `assets/awards.md`).

### Task 2: Schemas and index assembly

**Files:**
- Create: `src/lib/schemas.ts`, `src/lib/index.ts`, `src/lib/format.ts`, `src/content.config.ts`
- Test: `tests/schemas.test.ts`, `tests/index.test.ts`, `tests/format.test.ts`

**Interfaces:**
- Produces (`src/lib/schemas.ts`, using `z` from `astro/zod`):
  - `projectSchema`: `{ title: string; claim: string (1–90 chars); group: string; order: number; repo?: string /^[\w.-]+\/[\w.-]+$/; url?: string (absolute URL or path starting "/"); live: boolean = false; private: boolean = false; stars?: number int ≥0; updated: Date; figures: {src: string starting "/"; caption: string}[] = [] }`, refined: `private && repo` → error message `"private projects must not have a repo"`.
  - `writingSchema`: `{ title: string; date: Date; excerpt: string (1–280); url: string URL; source: string = "LinkedIn" }`.
  - `groupsSchema`: array of `{ id: string; label: string; order: number }`, ids unique.
  - `cvSchema`: `{ roles: {org, role, years, line, url?}[]; education: {degree, school, years, line}[]; affiliations: string[]; languages: string[]; awards: {name, org, amount, year}[]; thesisUrl: string; links: {label, url}[] }` (all strings unless noted).
  - Types `Project`, `Writing`, `Group`, `CV` via `z.infer`.
- Produces (`src/lib/index.ts`): `type IndexRow = { slug: string; title: string; claim: string; href: string; meta: string[] }`, `type IndexGroup = { id: string; label: string; rows: IndexRow[] }`, `buildIndex(projects: {id: string; data: Project}[], writing: {id: string; data: Writing}[], groups: Group[], meta: Record<string, {stars?: number; updated: Date}>): IndexGroup[]`.
  - Project groups in `groups` order, rows by `order`; groups with no rows omitted; unknown `group` id throws `Error("unknown group '<id>' in <slug>")`.
  - Project `href` = `/work/<slug>/`. `meta` = `["★<stars>"]` if public with stars, then `formatMonth(updated)`, then `"live"` if `live`, or `"private"` if private.
  - Writing group (`id: "writing"`, label `"Writing"`) appended last with the 3 newest entries, only if `writing.length > 0`; row `href` = entry `url`, `meta` = `[formatMonthYear(date)]`.
- Produces (`src/lib/format.ts`): `formatMonth(d: Date): string` → `"Oct"` style when in the current year, else `"Oct 2025"`; `formatMonthYear(d: Date): string` → `"Sep 2026"`; `formatStars(n: number): string` → `"★20"`, `"★1.2k"` at ≥1000.
- `src/content.config.ts`: `projects` (glob `src/content/projects/*.md`), `writing` (glob `src/content/writing/*.md`), `groups` and `cv` (file loaders). Thin: imports the schemas.

- [ ] **Step 1: Write failing tests.** Names and assertions:
  - `projectSchema rejects private project with repo` → `safeParse({...valid, private: true, repo: "a/b"}).success === false` and issue message matches.
  - `projectSchema rejects claim over 90 chars`.
  - `projectSchema rejects repo without owner` (`"archetypal"`).
  - `writingSchema rejects non-URL url`.
  - `buildIndex omits Writing when there are no entries` → no group with `id === "writing"`.
  - `buildIndex appends Writing with newest 3 when entries exist` → last group id `"writing"`, 3 rows, newest first.
  - `buildIndex hides stars and shows "private" for private projects` → meta has no `★`, includes `"private"`.
  - `buildIndex throws on unknown group`.
  - `buildIndex orders groups by groups.yaml order and rows by order`.
  - `formatStars(1234) === "★1.2k"`, `formatMonthYear(new Date("2026-09-03")) === "Sep 2026"`.
- [ ] **Step 2:** `npm test` → FAIL (modules missing).
- [ ] **Step 3:** Implement the three modules and `content.config.ts`.
- [ ] **Step 4:** `npm test` → PASS. `npm run build` → PASS (empty collections allowed).
- [ ] **Step 5:** Commit `feat: content schemas and index assembly`.

### Task 3: GitHub metadata with fallback

**Files:**
- Create: `src/lib/github.ts`
- Test: `tests/github.test.ts`

**Interfaces:**
- Consumes: `Project` from Task 2.
- Produces: `fetchRepoMeta(project: Project, opts?: { fetch?: typeof fetch; token?: string }): Promise<{ stars?: number; updated: Date }>` and `fetchAllMeta(entries: {id: string; data: Project}[], opts?): Promise<Record<string, {stars?: number; updated: Date}>>`.
  - No `repo` or `private` → return `{ stars: data.stars, updated: data.updated }` without calling fetch.
  - Calls `https://api.github.com/repos/<repo>` with `Accept: application/vnd.github+json`, `Authorization: Bearer <token>` when `token` (default `process.env.GITHUB_TOKEN`). 5 s timeout via `AbortSignal.timeout(5000)`.
  - Success → `{ stars: stargazers_count, updated: new Date(pushed_at) }`.
  - Any failure (non-2xx, network, timeout, bad JSON) → fallback values and one `console.warn("github meta fallback for <repo>: <reason>")`. Never throws.

- [ ] **Step 1: Write failing tests** with an injected fake `fetch`:
  - `uses API values on 200`.
  - `falls back on 403 rate limit` → returns frontmatter values, warns once.
  - `falls back on network error` (fake fetch rejects).
  - `does not call fetch for private projects` (spy call count 0).
  - `sends bearer token when provided`.
- [ ] **Step 2:** `npm test` → FAIL. **Step 3:** implement. **Step 4:** `npm test` → PASS.
- [ ] **Step 5:** Commit `feat: GitHub metadata with committed fallback`.

### Task 4: Content migration

**Files:**
- Create: `src/content/groups.yaml`, `src/content/projects/*.md` (9 files), `src/content/writing/.gitkeep`, `src/content/cv.yaml`
- Delete: `assets/` (after awards are moved)

**Interfaces:**
- Consumes: schemas from Task 2 (the build validates every file).

`groups.yaml`: `tools` "Tools" 1, `agents` "Agents" 2, `live` "Live" 3.

Projects (slug = filename). Fallback `stars`/`updated` come from `gh api -X GET search/repositories -f q='repo:<owner>/<name>'` (`stargazers_count`, `pushed_at`); this also works for the private repos, which get `updated` only. Claims are drafts; collect them in the final report for the owner to confirm:

| slug | group/order | repo | url | private | draft claim |
|---|---|---|---|---|---|
| archetypal | tools/1 | samuelduchesne/archetypal | https://archetypal.readthedocs.io/ | no | Retrieve, build, simulate and convert building energy model templates |
| trnsystor | tools/2 | samuelduchesne/trnsystor | — | no | A Python scripting language for TRNSYS |
| energy-pandas | tools/3 | samuelduchesne/energy-pandas | — | no | pandas for building time series: loads, temperatures, energy |
| idfkit | agents/1 | idfkit/idfkit | https://idfkit.com | no | A fast EnergyPlus toolkit for Python, with an MCP server for AI agents |
| project-kit | agents/2 | samuelduchesne/project-kit | — | no | Skills and hooks that let AI agents create, update and audit projects |
| openstudio-sketchup-mcp | agents/3 | — | — | yes | An MCP server that lets AI agents drive OpenStudio in SketchUp |
| eplusout-dashboard | live/1 | samuelduchesne/eplusout-dashboard | /eplusout-dashboard/ | no, `live: true` | Explore EnergyPlus SQL output in the browser |
| geo-exchange-feasibility-tool | live/2 | — | — | yes | Early-stage feasibility for geo-exchange systems |
| ma-buildings-decarbonization | live/3 | — | — | yes | Data pipeline for Massachusetts building decarbonization scenarios |

- Each body: 1–3 short paragraphs drawn from the repo README (public) or the claim (private, no internals). The `idfkit` body lists `idfkit/idfkit`, `idfkit-mcp`, `idfkit-js`, `idfkit-shoebox`, `idfkit-lsp` with their homepages. Read public READMEs with the GitHub MCP `get_file_contents` tool (the `gh` CLI only works for search here). Private bodies use no README content.
- `cv.yaml` from `_config.yml` (git history, commit `c04dd58`): all 7 roles, one line each; PhD and B.Eng.; ASHRAE (2014), IBPSA (2015); French and English; awards = the full list in `assets/awards.md`, translated to English, amounts kept; `thesisUrl` = the current Dropbox link; `links` = the four contact links in Global Constraints. No email field exists in the schema.

- [ ] **Step 1:** Write the files.
- [ ] **Step 2:** `npm run build` → PASS (schemas validate all content). Then temporarily set `private: true` on `trnsystor` and confirm the build fails with "private projects must not have a repo"; revert.
- [ ] **Step 3:** `git rm -r assets` and commit `content: migrate projects and CV`.

### Task 5: Design, pages and output checks

**Files:**
- Create: `src/styles/global.css`, `src/layouts/Base.astro`, `src/components/IndexRow.astro`, `src/components/Figure.astro`, `src/pages/index.astro`, `src/pages/cv.astro`, `src/pages/work/[slug].astro`, `src/pages/[writing]/index.astro`, `src/pages/404.astro`, `scripts/check-dist.mjs`, `scripts/check-phone.mjs`
- Test: `tests/check-dist.test.ts`

**Interfaces:**
- Consumes: `buildIndex`, `fetchAllMeta`, `formatMonthYear`, collections.
- Produces: `Base.astro` props `{ title: string; description: string; ogImage?: string }`. `Figure.astro` props `{ n: number; src: string; caption: string }` → `<figure>` with `<figcaption>Fig. {n} — {caption}</figcaption>`. `checkDist(distDir: string): string[]` (exported from `scripts/check-dist.mjs`; returns violations, CLI exits 1 if any).

Layout, from the spec and the approved mockup (`.superpowers/brainstorm/*/content/hero-figure-v2.html`, option 3):
- Top line in monospace: `SLD` left; nav right = one anchor per index group (`#tools`, `#agents`, `#live`), then `writing` (only if entries) and `cv`.
- Headline (serif, second line italic), sub-line (sans), then the index. Each group: label in a left column (monospace, muted), rows on the right: **title** — claim, metadata right-aligned in monospace. Thin rule between rows. On < 640 px the label sits above its rows and metadata drops below the claim.
- Footer: `CV · Carbon Signal · Introba · GitHub · LinkedIn · ORCID · ResearchGate`. Nothing else.
- `/work/<slug>/`: title, claim, metadata, links (repo unless private, `url` if set), body, then `figures` as numbered `Figure`s. Back link to `/`.
- `/cv/`: small photo, roles, education, affiliations, languages, awards table, thesis link.
- `/writing/`: all entries, newest first, title → LinkedIn, date, excerpt.
- Typefaces: pick a serif display + sans + mono, self-hosted or system stacks; no external font CDN other than Google Fonts.

`checkDist` violations (each a test):
- `flags mailto links` and `flags email addresses` (regex `[\w.+-]+@[\w-]+\.[\w.]+` in text and attributes).
- `flags <script> other than gtag` (allowed: `googletagmanager.com/gtag/js?id=G-JPF0EN6DC1` and the inline `gtag('config'…)` snippet).
- `flags <img>, <svg>, <figure>, <video>, <canvas> on the home page`.
- `flags routes outside the allowed set` (walk `dist/`; allowed per Global Constraints).
- `flags /writing/ present with zero writing entries` (checker reads `src/content/writing/*.md` count).
- `flags repo links on private project pages` (read frontmatter; private page HTML must not contain `github.com/<anything>/<slug>`).
- `passes on a clean fixture` (tests build small fixture dirs in a tmp folder).

- [ ] **Step 1:** Write `tests/check-dist.test.ts` → FAIL.
- [ ] **Step 2:** Implement `scripts/check-dist.mjs` → tests PASS.
- [ ] **Step 3:** Build the layout, components and pages.
- [ ] **Step 4:** `scripts/check-phone.mjs`: start `astro preview`, open `/`, `/cv/`, and every `/work/*/` at 375×812 and 1280×800, in light and dark (`colorScheme`), assert `document.documentElement.scrollWidth <= innerWidth`, save screenshots to `dist-shots/` (git-ignored).
- [ ] **Step 5:** `npm run build && npm run check` → no violations, no overflow.
- [ ] **Step 6:** Look at the screenshots against the approved mockup and the anti-goals: quiet, typographic, index-first, dark mode warm charcoal. Fix and repeat until it holds.
- [ ] **Step 7:** Commit `feat: notebook design, pages and output checks`.

### Task 6: Agent maintenance (refresh script, skills, CLAUDE.md)

**Files:**
- Create: `scripts/refresh-meta.mjs`, `CLAUDE.md`, `.claude/skills/add-project/SKILL.md`, `.claude/skills/add-writing/SKILL.md`, `.claude/skills/refresh/SKILL.md`
- Modify: `README.md` (what the site is, `npm run dev|build|test|check|refresh`)
- Test: `tests/refresh-meta.test.ts`

**Interfaces:**
- Consumes: `fetchRepoMeta` (Task 3).
- Produces: `updateFrontmatter(markdown: string, meta: {stars?: number; updated: Date}): string` — rewrites only the `stars` and `updated` keys, preserves every other line byte-for-byte; and the CLI `npm run refresh` that applies it to every public project file, skipping ones where the fetch fell back.

- [ ] **Step 1: Tests:** `updates stars and updated only`, `preserves body and other keys exactly`, `adds stars when absent`, `leaves private projects untouched`. → FAIL, implement, PASS.
- [ ] **Step 2:** `CLAUDE.md` (≤ 80 lines): what the site is and is for (four visitors, one page); the anti-goals verbatim from the spec; where content lives; the schema rules (claim ≤ 90, private ⇒ no repo); "never add an email address"; commands; "every change ends with `npm run build && npm run check`".
- [ ] **Step 3:** Skills, each with frontmatter `name` + `description` (when to use), steps, and a final "run `npm run build && npm run check`; show the owner the new row":
  - `add-project`: input repo name/URL or a description for a private project → ask whether private → read README via GitHub tools → draft claim (≤ 90 chars) and group → **owner confirms claim** → write `src/content/projects/<slug>.md` with fallback stars/updated.
  - `add-writing`: input LinkedIn URL + pasted text → title, date, excerpt (≤ 280, owner's words, not rewritten) → write `src/content/writing/<yyyy-mm-dd>-<slug>.md`. Note that the first entry makes `/writing/` and the Writing group appear.
  - `refresh`: `npm run refresh`, review the diff, build, check, commit.
- [ ] **Step 4:** Dry-run `add-writing` with a fake entry, confirm `/writing/` appears and checks pass, then delete the entry and confirm it disappears.
- [ ] **Step 5:** Commit `feat: agent skills and refresh script`.

### Task 7: Deploy workflow and route-collision check

**Files:**
- Create: `.github/workflows/deploy.yml`, `scripts/check-routes.mjs`
- Test: `tests/check-routes.test.ts`

**Interfaces:**
- Produces: `findCollisions(siteRoutes: string[], pagesRepos: string[]): string[]` — returns repo names equal to a first path segment the site uses (`cv`, `work`, `writing`). CLI: lists `GET /users/samuelduchesne/repos?per_page=100` (paginated), keeps `has_pages === true`, exits 1 on any collision. Unreachable API → warn and exit 0.

- [ ] **Step 1: Tests:** `reports a repo named cv with Pages`, `ignores repos without Pages` (filtering happens before the call, so test the CLI's filter helper `pagesRepoNames(repos)`), `no collisions for eplusout-dashboard and argonne-diet`. → FAIL, implement, PASS.
- [ ] **Step 2:** `deploy.yml`: triggers `push` to `master`, `schedule` weekly (`cron: '17 6 * * 1'`), `workflow_dispatch`. Permissions `contents: read, pages: write, id-token: write`; concurrency group `pages`. Job: checkout → `actions/setup-node` (22, npm cache) → `npm ci` → `npm test` → `npm run build` (env `GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}`) → `npm run check` → `node scripts/check-routes.mjs` → `actions/upload-pages-artifact` (`dist`) → `actions/deploy-pages`. Use current major versions of each action.
- [ ] **Step 3:** Run every workflow command locally in order; all pass. Run `node scripts/check-routes.mjs` against the live API; expect no collisions.
- [ ] **Step 4:** Commit `ci: build, check and deploy to GitHub Pages`, push the branch.
- [ ] **Step 5:** Final report to the owner: the draft claims table for confirmation; that Settings → Pages → Source must be switched to "GitHub Actions" before merging to `master`; screenshots of `/` light and dark, phone and desktop.
