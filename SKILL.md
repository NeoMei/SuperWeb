---
name: superweb
description: Create a website or web app through three explicit decision gates (direction, sample, acceptance). Use when the user asks for a landing page, product page, portfolio, event page, one-pager, admin UI, dashboard, settings screen, or any web interface that should look designed rather than templated. Produces a static index.html by default and verifies it in local Chrome.
---

# SuperWeb

SuperWeb turns "make me a web page" into a gated workflow. You own the page
content and code. The `superweb` CLI owns task state, decisions, browser
acceptance, and delivery.

Talk to the user in Simplified Chinese. Write the page copy in the language of
its intended audience.

## Non-negotiables

- Do not start coding before the direction gate is confirmed.
- Do not build the full page before the sample gate is confirmed.
- Do not accept or deliver without an explicit user decision. Acceptance is
  the user's decision, never yours.
- Never edit `superweb.json` by hand. Every state change goes through the CLI.
- Default output is static HTML/CSS/JS that opens by double-click. Do not add
  React, Vite, a build step, or a package manager unless the user explicitly
  asks.
- Use local assets or visibly marked placeholders. Never invent statistics,
  customers, logos, testimonials, or screenshots. Never draw a fake product
  image in CSS/SVG.
- Do not fetch images from stock libraries or read the user's browser session.
- Use every asset the user provides before any placeholder.

## Workflow

### 0. Create the task

If the user has not given enough detail, ask for: what it is, who it is for,
the content that must appear, and which assets exist. Then:

```bash
superweb start --dir <task-dir> --name "<short name>" --brief "<the brief>"
```

If the brief is long, put it in a file and use `--brief-file`. Route is
detected from the brief (`page` or `app`); override with `--route page|app`
only when the detection is wrong. `需求.md` is the task brief; you may append
clarified details the user actually gave, never invented ones.

Run `superweb continue --dir <task-dir> --json` whenever you resume. The state
file is the source of truth, not the conversation.

### 1. Direction gate

Read `需求.md` and run:

```bash
superweb directions --dir <task-dir> [--seed <seed>]
```

This writes three route-compatible directions plus a content structure to
`设计方向.md`. Present all three to the user with their concept, composition,
type, palette, motion, and suitable scenarios. Present the section structure
and what each section must prove. Ask the user to choose one direction and
confirm the structure. Do not choose for them and do not blend directions.

After an explicit choice:

```bash
superweb decide --dir <task-dir> --gate direction --choice <direction-id> \
  --revision <current revision> --decision-id <new-id>
```

The revision must come from `continue`/`status`. If it is stale, re-read the
state and ask the user again; never silently retry with a new revision.

Follow the chosen recipe in `references/direction-library.md`. It is a recipe,
not a template: derive the actual layout from the content.

### 2. Sample gate

Before writing code, write the design tokens into the sample: colors,
typefaces, spacing scale, radii, shadows, and motion. Use the confirmed
direction, exact copy, and real assets.

- `page` route: build only the first viewport plus one representative section
  in `预览/v0.html`. It must use the final type, color, and spacing, not a
  throwaway mock.
- `app` route: build the application shell plus one representative screen in
  `预览/v0.html`.

Open the sample in a browser and show the user a screenshot or the file. Only
after explicit approval:

```bash
superweb decide --dir <task-dir> --gate sample --choice accept \
  --revision <current revision> --decision-id <new-id>
```

If the user wants changes, keep working in `v0.html`; the stage stays `sample`
and no decision is recorded.

### 3. Full page

Extend the confirmed sample into `预览/index.html`. Keep the tokens. Every
section must answer "what does it prove"; delete filler. Complete all menus,
prices, people, states, and content — no "Lorem ipsum" and no half-finished
sections. Follow `references/page-blueprint.md` or
`references/app-blueprint.md` for the route.

Responsive behavior is part of the page, not a later step. Check at 1440,
768, and 390 CSS pixels. Include hover, focus, active, disabled, loading,
empty, and error states where they apply.

When the page is ready, run:

```bash
superweb inspect --dir <task-dir>
```

This starts a temporary local server, launches local Chrome, checks three
viewports, writes screenshots and `验收/report.json`, and moves the task to
`acceptance`. Present the screenshots and the issue list to the user. Do not
hide failures.

If Chrome is missing, explain the exact reason, offer to fix `CHROME_PATH` or
`--chrome`, and only continue if the user explicitly accepts delivery without
browser acceptance. That decision requires `--acknowledge-issues`.

To rework, use the acceptance decision with `rework` (see
`references/state-machine.md`) and only rebuild the affected sections.

### 4. Acceptance and delivery

Only after the user accepts the result:

```bash
superweb decide --dir <task-dir> --gate acceptance --choice accept \
  --revision <current revision> --decision-id <new-id>

superweb deliver --dir <task-dir>
```

`交付/` contains `index.html`, assets, acceptance evidence, and `说明.md`.
Verify it opens directly and that the readme is honest about any remaining
issues.

## Route rules

### page

Use for landing pages, product pages, portfolios, event pages, one-pagers.
Start from the concrete object: a book, a photographer, a festival, a tool.
The first viewport must state what it is, who it is for, and the next action.
For events, time, place, and action belong above the fold. Read
`references/page-blueprint.md`.

### app

Use for admin panels, dashboards, settings, lists, editors, and work
interfaces. Use quiet panels, aligned data, clear table hierarchy, and honest
empty states. Do not turn a workspace into a marketing page. Read
`references/app-blueprint.md`.

## Anti-patterns (reject on sight)

- Purple-pink gradients, glassmorphism, generic SaaS hero + three feature
  cards, emoji as icons.
- Fake logo walls, invented metrics, invented testimonials, lorem ipsum.
- CSS-drawn "product screenshots", stock-photo filler, decorative charts with
  no data.
- Centered everything, cards inside cards, excessive radii and shadows.
- A sample that uses different fonts/colors from the full page.
- A beautiful hero followed by a template section.

## Commands

| Command | Purpose |
| --- | --- |
| `superweb start` | Create task state, `需求.md`, preview and acceptance dirs |
| `superweb directions` | Extract 3 seeded directions + content structure |
| `superweb decide` | Record a gated decision; validates stage and revision |
| `superweb inspect` | Local Chrome acceptance; writes screenshots + report |
| `superweb deliver` | Build self-contained `交付/`, archive older delivery |
| `superweb continue` | Read stage, decisions, artifacts, next action |
| `superweb status` | Read-only compact state |

Add `--json` for machine-readable output. Exit codes: `0` success, `1`
acceptance/delivery check failed, `2` usage, `3` state or decision rejected,
`4` local Chrome unavailable.

## References

- `references/state-machine.md` — stages, revisions, decisions, rework,
  delivery semantics.
- `references/page-blueprint.md` — page sections and what each must prove.
- `references/app-blueprint.md` — app shell, primary screen, states.
- `references/direction-library.md` — the 12 direction recipes.
- `references/acceptance.md` — browser checks and report interpretation.
- `references/delivery.md` — self-contained delivery rules.
