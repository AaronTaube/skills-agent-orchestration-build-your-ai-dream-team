# Mona's Project Pulse Dashboard — Implementation Plan

## 1. Summary

Project Pulse is a small, runnable, browser-based dashboard that helps Mona see the state of her active projects at a glance. It renders a responsive grid of **project cards**, each showing a project name, short description, **status badge** (e.g. On Track / At Risk / Blocked / Done), **priority** indicator (e.g. Low / Medium / High / Critical), owner, and progress. Data is sourced from a static `app/project-data.json` file so the dashboard is fully self-contained with no backend. The goal is a polished, accessible single-page dashboard that a learner can launch from VS Code and immediately see Mona's projects laid out as a real product surface — not a bare HTML page.

## 2. Ordered implementation steps

1. **Define the project data model** in `app/project-data.json` (fields, allowed status values, allowed priority values, sample projects).
2. **Build the HTML structure** in `app/index.html`: page shell, `<header>` with dashboard title/summary, `<main class="dashboard">` container, and the rendering logic (small inline `<script>`) that fetches `project-data.json` and produces `.project-card` elements with deterministic class hooks the Designer can style.
3. **Design and implement styling** in `app/styles.css`: layout, typography, cards, status badge variants, priority treatment, responsive breakpoints, focus/hover states, accessibility.
4. **Add run configuration** in `.vscode/launch.json` so the learner can launch the dashboard with one click — opening `app/index.html` with `cwd` set to `${workspaceFolder}/app`.
5. **Integrated validation**: launch the app via the debug config, confirm cards render, JSON is valid, styling matches design intent, no console errors.

## 3. File assignments

| File | Owner | What it contains |
|---|---|---|
| `app/project-data.json` | **Coder** | Strict JSON array of sample project objects (id, name, description, status, priority, owner, progress, dueDate) — the single source of truth for the dashboard. |
| `app/index.html` | **Coder** (structure + rendering logic) with **Designer** input on class hooks/semantics | Semantic HTML shell, dashboard header, main dashboard container, and a small inline `<script>` (or static markup) that consumes `project-data.json` and outputs `.project-card` nodes with the agreed class hooks. |
| `app/styles.css` | **Designer** | Full visual design: dashboard layout/grid, card styling, status badge color variants, priority treatment, typography, spacing, responsive breakpoints, accessible focus states. |
| `.vscode/launch.json` | **Coder** | Strict JSON (no comments) launch configuration that opens `index.html` with `cwd` set to `${workspaceFolder}/app`. |

## 4. Designer responsibilities

Scope: `app/styles.css` primarily, plus advisory input on class names and semantic structure in `app/index.html`.

- Establish a clear visual identity: rounded corners, subtle shadows, generous spacing, strong typography hierarchy — this must read as a real dashboard, not a bare page.
- Style the top-level `.dashboard` container as a responsive CSS grid (auto-fill / `minmax`) of `.project-card` items.
- `.project-card` styling: padding, border-radius, background, elevation on hover/focus, header (name + status badge) and body (description, meta row) hierarchy.
- **Status badges** (`.status-badge` with modifier classes such as `.status-on-track`, `.status-at-risk`, `.status-blocked`, `.status-done`): distinct background/foreground pairs meeting WCAG AA contrast; pill shape; small caps or medium weight; never rely on color alone (include text label and/or icon).
- **Priority treatment** (`.priority` with modifiers `.priority-low`, `.priority-medium`, `.priority-high`, `.priority-critical`): a clear ordinal cue — e.g. left border accent on the card and/or a labeled chip. Critical must be visually strongest without shouting.
- Progress indicator styling (e.g. `.progress` bar) with accessible labeling hooks.
- **Responsive layout**: single column on narrow screens (<640px), two columns on tablets, three+ on desktop; readable line lengths; no horizontal scroll.
- **Accessibility**: visible `:focus-visible` outlines, sufficient color contrast, respect `prefers-reduced-motion`, ensure hit targets ≥ 44px where interactive.
- Coordinate with Coder on the exact class-hook contract before the Coder finalizes the render template.

## 5. Coder responsibilities

Scope: `app/project-data.json`, `app/index.html`, `.vscode/launch.json`.

**`app/project-data.json`**
- Strict JSON (no comments, no trailing commas).
- Top-level array of 5–8 sample project objects covering all status and priority variants so the Designer's styles are all exercised.
- Suggested shape per project: `{ "id": string, "name": string, "description": string, "status": "on-track" | "at-risk" | "blocked" | "done", "priority": "low" | "medium" | "high" | "critical", "owner": string, "progress": number 0–100, "dueDate": "YYYY-MM-DD" }`.
- Include at least one long name and one long description to exercise overflow behavior.

**`app/index.html`**
- Semantic structure: `<header>` with dashboard title and short summary/count, `<main class="dashboard" aria-label="Projects">`.
- Small inline `<script type="module">` that `fetch`es `./project-data.json`, renders `.project-card` nodes using the class-hook contract agreed with Designer (`.project-card`, `.project-card__header`, `.project-card__title`, `.status-badge status-<value>`, `.priority priority-<value>`, `.project-card__description`, `.project-card__meta`, `.progress`).
- Empty-state markup and error-state markup rendered when JSON is missing/invalid or the list is empty.
- Meta tags: `<meta charset>`, `<meta name="viewport">`, `<title>Project Pulse</title>`, link to `styles.css`.
- No frameworks, no bundler, no external network requests — must run when opened via the launch config.
- Do not put visual styling inline; leave that to `styles.css`.

**`.vscode/launch.json`**
- Strict JSON, no comments, no trailing commas.
- One configuration that opens `index.html` in a browser preview/simple-browser style command available in the environment.
- Set `"cwd": "${workspaceFolder}/app"` so relative fetch of `project-data.json` resolves.
- Deterministic `name` (e.g. `"Launch Project Pulse"`), stable across runs.
- Must not depend on tooling not present in the Codespace baseline.

## 6. Dependencies between steps

- **`project-data.json` shape must be agreed before `index.html` render logic is finalized** — the Coder can draft both together, but the JSON schema is the contract.
- **Designer's CSS class-hook contract must be agreed before `index.html` markup/render template is frozen** — otherwise Designer targets classes that don't exist.
- **`styles.css` depends on the class-hook contract**, not on `index.html` being complete — Designer can build against the agreed hooks in parallel.
- **`.vscode/launch.json` depends on `index.html` existing** at `app/index.html` and on the decision to run from `cwd = ${workspaceFolder}/app`.
- **Final integrated validation depends on all four files being present.**

## 7. Parallel vs. sequential work

**Phase 0 (sequential, blocking):** Coder + Designer agree on a small shared contract:
- JSON field names and allowed status/priority values (owned by Coder).
- CSS class-hook names for dashboard, card, status badge, priority, progress (owned by Designer).

**Phase 1 (parallel — no file overlap):**
- Coder writes `app/project-data.json`.
- Coder writes `app/index.html` render structure using the agreed class hooks.
- Designer writes `app/styles.css` against the agreed class hooks.

These three files have disjoint scopes and can be produced simultaneously once Phase 0 is done.

**Phase 2 (sequential — depends on Phase 1):**
- Coder writes `.vscode/launch.json` once `app/index.html` exists at its final path.

**Phase 3 (sequential — integration):**
- Orchestrator verifies all four files integrate cleanly.

**Must be sequential:** class-hook contract before CSS/HTML; `index.html` before `launch.json`; integration last.

## 8. Validation expectations

- `app/project-data.json` parses as strict JSON (e.g. `python -m json.tool` or `jq . app/project-data.json` exits 0); every object has all required fields; every `status` and `priority` value is in the allowed enum.
- `.vscode/launch.json` parses as strict JSON with no comments; `cwd` resolves to `${workspaceFolder}/app`; the configuration launches without error.
- Launching the debug config opens `app/index.html` and the dashboard renders:
  - Header shows title and project count.
  - All sample projects appear as `.project-card` elements.
  - Each card shows a status badge with a status-specific style and a priority treatment.
  - Layout reflows cleanly at ~360px, ~768px, and ~1280px widths.
- Browser DevTools console shows **no errors and no failed network requests**.
- Keyboard `Tab` navigation shows a visible focus ring on any interactive elements.
- Empty-state markup shows if `project-data.json` is an empty array; error-state shows if it fails to load.

## 9. Edge cases to handle

- **Empty project list** (`[]`): render a friendly empty state, not a blank page.
- **Missing or invalid JSON fields**: render the card defensively (fallback text like "Untitled project", "No description", default status "unknown"); log a single console warning, not a crash.
- **Unknown `status` or `priority` values** not covered by CSS modifier classes: fall back to a neutral badge style so nothing renders unstyled.
- **Very long project names**: truncate with ellipsis or wrap gracefully; must not break card grid.
- **Very long descriptions**: clamp to N lines (e.g. `-webkit-line-clamp: 3`) with accessible full text still available.
- **Progress out of range** (`< 0` or `> 100`): clamp to 0–100.
- **`fetch('./project-data.json')` fails** (e.g. opened via `file://`): show an error state with guidance to use the launch config; this is why `cwd` matters.
- **Responsive breakpoints**: verify no horizontal scroll at 320px width.
- **Reduced motion**: any hover/transition effects respect `prefers-reduced-motion: reduce`.
- **Color-only signaling**: status and priority must also be conveyed by text/shape, not just color.

## 10. Open questions

1. **Exact status enum** — proposed: `on-track`, `at-risk`, `blocked`, `done`. Confirm wording and whether `not-started` is needed.
2. **Exact priority enum** — proposed: `low`, `medium`, `high`, `critical`. Confirm whether `critical` is desired or if `high` is the top tier.
3. **Number of sample projects** — proposed 6 (enough to cover every status × priority combination the Designer must style). Acceptable?
4. **Static vs. dynamic rendering** — proposed: small inline JS `fetch` of `project-data.json`. Alternative: static markup with data duplicated in HTML. `fetch` is preferred because it exercises the `cwd` in `launch.json`; confirm.
5. **Launch mechanism** — should `.vscode/launch.json` use VS Code's Simple Browser, the Edge/Chrome debug adapter, or a task that runs a lightweight static server? The `fetch` approach may require a server rather than `file://`. **This is the biggest open risk** and should be resolved before Coder writes `launch.json`.
6. **Interactivity scope** — is filtering/sorting by status or priority in scope for v1, or is read-only display sufficient? Plan currently assumes read-only.
7. **Branding** — any color palette, logo, or typography constraint from Mona, or is the Designer free to choose?
