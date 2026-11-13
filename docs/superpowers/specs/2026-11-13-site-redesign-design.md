# samuelduchesne.github.io redesign

Date: 2026-11-13
Status: draft, awaiting approval (reviewed by a builder check; owner answered its open questions)

## Intent

The site is a 2021 Jekyll résumé theme whose content lives in `_config.yml`.
It undersells its owner and has not kept up with his work: it features pyumi
and UBEM.io, and none of the libraries, AI tooling and live tools he has built
since.

The new site has one job: make the case that Samuel Letellier-Duchesne is
both a building-energy-simulation expert and a serious software developer.
It makes that case with the work itself, presented with restraint.

"In this age of AI" means how the site is **made and maintained**: designed
and built with an AI agent, and kept current by agents following skills that
live in the repository. It does not mean AI as the theme of the page.

## Audience

One page serves four visitors equally; there are no per-audience pages or
routes:

- a potential Carbon Signal customer,
- a hiring manager,
- an open-source collaborator,
- an Introba client.

Each should leave believing the same two things: he knows the physics, and he
builds real software around it.

## Goals

- Lead with a two-sided claim, not a job title.
- Use the work as proof: a readable index of projects, tools and writing.
- Keep the CV material (jobs, PhD, MIT postdoc, awards), shorter and lower.
- Keep project facts current without hand-editing (stars, last update).
- Make every routine update (new project, new LinkedIn post, refresh) a
  one-line request to an agent, carried out by a skill in the repo.

## Non-goals

- Per-audience landing pages.
- A blog engine with full posts on the site (the writing list links out to
  LinkedIn; on-site posts can come later, see Future).
- Comments, newsletter, search.

## Anti-goals (would make it a failure even if it works)

- A hero image, figure or animation on the home page. The owner wants the
  page to surprise by restraint: visitors read and choose deliberately.
- AI gimmicks: chatbot ("ask my résumé"), animated "AI" effects, AI framing
  in the copy.
- Skill bars, logo walls, carousels, "scroll down" arrows.
- A colophon or any "built with AI" credit line. (Owner's explicit call.)
- Breaking the GitHub Pages project sites that already live under this domain
  (see Constraints).

## Design (decided)

**Direction A, "research notebook", with no hero figure.**

- Warm off-white page (`#f6f4ef`-ish), near-black text.
- Serif display headline; clean sans-serif body; monospace only for small
  labels, metadata and captions.
- One restrained accent colour, for links and highlights only.
- Dark mode follows the system setting: warm charcoal, not pure black.

**Home page, top to bottom:**

```
SLD                                   tools · live · writing · cv
Building physics,
written as software.
Simulation engineer, open-source author, co-founder of Carbon Signal.

Tools      archetypal — simulation templates            ★20 · Oct
           trnsystor — scripting TRNSYS                 ★17 · Oct
           idfkit — EnergyPlus toolkit for Python …     ★21 · Nov
Live       eplusout-dashboard — try it →                      Nov
Writing    <post title> (LinkedIn)                       Sep 2026
──
CV · Carbon Signal · Introba · GitHub · contact
```

- The headline copy above is the working draft; final wording is the owner's.
- The index **is** the page. One line per entry: name, a one-line claim,
  monospace metadata (stars, last update month, "live" when there is
  something to try).
- Entries are grouped like a table of contents. Group names are content, set
  in the data, not hard-coded.
- Figures, screenshots, agent transcripts and longer descriptions live **one
  click deeper**, on each entry's own page, as numbered figures with
  captions ("Fig. 2 — …"). The visitor who chose to click gets the evidence.
- Navigation: one home-page anchor per index group (generated from the
  group names in the data), plus `/writing/` and `/cv/`. No `/work/` or
  `/tools/` index pages. *(Builder's call.)*
- Every index entry has its own `/work/<slug>/` page at launch. Figures are
  optional in the schema; a page may launch with text only and gain figures
  later. *(Builder's call.)*
- CV is its own page: roles, education, affiliations, awards, condensed from
  the current `_config.yml`.

## Content

**Showcase (owner's picks):**

1. Simulation libraries: archetypal, trnsystor, energy-pandas.
2. AI-era tooling: idfkit, project-kit, openstudio-sketchup-mcp.
   idfkit is **one** index line (stars from `idfkit/idfkit`, link to
   idfkit.com); its entry page lists the ecosystem repos (`idfkit/idfkit`,
   `idfkit-mcp`, `idfkit-js`, `idfkit-shoebox`, `idfkit-lsp`). *(Builder's call.)*
3. Live tools: eplusout-dashboard, geo-exchange feasibility tool,
   Massachusetts decarbonization pipeline.

**Private projects** (owner's decision): openstudio-sketchup-mcp,
geo-exchange-feasibility-tool and the Massachusetts decarbonization pipeline
are private repos. Each gets an index line and an entry page that describes
it, marked "private", with **no repo link and no stars**. The schema allows a
project with no public repo; metadata for these is the month in the content
file, maintained by hand.

Public repos used: `samuelduchesne/archetypal`, `samuelduchesne/trnsystor`,
`samuelduchesne/energy-pandas`, `samuelduchesne/project-kit`,
`samuelduchesne/eplusout-dashboard` (live at `/eplusout-dashboard/`),
`idfkit/idfkit`.

**Moved to the CV page as background:** UBEM.io, pyumi, umitemplatedb, the
MIT postdoc, the PhD thesis.

**Writing:** a curated list of LinkedIn posts. Each entry: title, date,
one- or two-line excerpt, link. LinkedIn has no feed and blocks scraping, so
entries are added by hand (or by the `add-writing` skill from a pasted link
and text). No posts are supplied yet: the Writing group and `/writing/` stay
hidden until the first entry exists, and appear automatically after.
*(Builder's call.)*

**CV scope** *(builder's calls)*: keep all roles, each condensed to one or
two lines; keep education, affiliations and languages; awards use the fuller
list in `assets/awards.md` (translated to English, amounts kept); keep the
thesis link as is. The profile photo appears small on `/cv/` only and doubles
as the social share image; never on the home page.

**Contact** (owner's decision): no email address anywhere on the site.
Contact goes through LinkedIn; footer links are LinkedIn, GitHub, ORCID and
ResearchGate.

**Dropped:** the Jekyll config and theme, `assets/main.scss` (the February
2026 dark-theme override, superseded by this design), `assets/awards.md` once
its content is moved into the CV data, the Gemfile.

## Build (decided): Astro

- Static Astro site, deployed to GitHub Pages by a GitHub Actions workflow.
- Content in Markdown under content collections: one file per project, one
  per writing entry, and the CV as structured data. Each collection has a
  schema; a missing field or bad date fails the build with a clear message.
- No client-side JavaScript on the home page. Entry pages may embed a small
  interactive piece later without affecting the rest of the site.
- GitHub metadata (stars, last push) fetched at build time for public repos.
  A failed fetch must not fail the build: fall back to the last known values
  committed in the content files.
- A scheduled workflow rebuilds the site periodically (for example weekly)
  so stars and dates stay current without a commit.

## Agent maintenance (decided)

- `CLAUDE.md` at the repo root: what the site is, the design rules above
  (especially the anti-goals), where content lives, how to build and check.
- Skills in `.claude/skills/`:
  - `add-project`: create a project entry from a repo name or URL, draft the
    one-line claim for the owner to confirm, place it in a group.
  - `add-writing`: add a LinkedIn post from a pasted link and excerpt.
  - `refresh`: update committed fallback metadata (stars, dates) and rebuild.
- Each skill ends by running the build so the schema check catches mistakes.

## Constraints

- GitHub Pages user site: `https://samuelduchesne.github.io`. Other repos
  publish project sites under it (for example `/eplusout-dashboard/`,
  `/argonne-diet/`, and any repo with Pages enabled). The new site must not
  create routes that collide with existing project-site paths. Keep the site's
  own routes few and specific (`/`, `/cv/`, `/work/<slug>/`, `/writing/`).
  No current repo is named `cv`, `work` or `writing`; the plan includes a
  check before launch. Keep `/images/favicon.ico` at its path; other old
  asset paths and `#anchors` need no redirects. *(Builder's call.)*
- Pages must be switched from "deploy from branch" to "GitHub Actions" in the
  repo settings; the owner does this.
- Build from branch `feature/focused-albattani-9vhaey` (equal to `master`
  plus this spec).
- Keep Google Analytics (`G-JPF0EN6DC1`) unless the owner says otherwise.
- Phone layout must work: the index rows wrap cleanly; metadata may drop
  below the claim on narrow screens.

## Approaches considered

| Approach | Trade-off | Outcome |
|---|---|---|
| **Astro** | Content schemas catch agent mistakes at build time; zero JS by default; room for interactive pieces on entry pages. | **Chosen** by the owner. |
| Eleventy | Simpler and light, but no schema checks; malformed entries fail quietly; interactive pieces need hand work. | Not chosen. |
| Next.js static export | Built for apps; ships React to a text page. | Not chosen. |
| Stay on Jekyll | The theme-bound Ruby setup the owner wants to leave; no content checks. | Ruled out. |

Visual directions shown: A research notebook (chosen), B terminal/agent-native,
C data-as-hero. Hero options for A: technical drawing, one chart, no figure
(chosen).

## Left to the builder

- Exact typefaces (serif display, sans body, mono labels), accent colour,
  spacing scale, within the direction above.
- Content file layout and schema field names.
- How each entry page is laid out, as long as figures are numbered and
  captioned.
- Wording of entry claims (drafted, then confirmed by the owner).

## Future (not in this build)

- Full posts published on the site, alongside LinkedIn links.
- Embedded live tools on entry pages.
