# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file personal life dashboard (Dutch). No build step, no dependencies to install, no test suite. Everything lives in `index.html`.

**Live URL:** https://roderick2022vtw.github.io/life-dashboard/
**Deploy:** `git add -A && git commit -m "..." && git push` — GitHub Pages auto-deploys from `main` root within ~1 min.

## Architecture

`index.html` is structured in three sections:

1. **`<style>`** — All CSS using custom properties (`--bg`, `--accent`, `--blue`, etc.). Dark theme. Layout: sidebar (220px fixed) + scrollable main. Responsive breakpoint at 700px hides the sidebar.

2. **`<main>`** — One `<div id="page-{id}" class="page">` per section. Only the `.active` page is visible. Pages: `today`, `habits`, `week`, `maand`, `plan`, `import`, `spaans`, `investing`, `reading`, `dagstructuur`, `review`, `doelen`.

3. **`<script>`** — All logic. Key globals:
   - `data` — the full in-memory state, persisted to `localStorage('fitdash_v2')` as JSON
   - `selectedDate` — which date the habits/workouts/metrics panels are editing (defaults to today)
   - `selectedPlanSession` — which workout is shown in the plan detail panel
   - `charts` — holds Chart.js instances so they can be `.destroy()`ed before re-render

## Data model

```
data.days[YYYY-MM-DD] = {
  habits:  { [habitId]: boolean },
  workout: string | null,          // one of WORKOUTS[].id
  metrics: { protein_g, water_l, sleep_h, steps, kcal, carbs_g, fat_g, fiber_g }
}
```

Countdown targets are stored separately: `localStorage('life_targets')` → `{ vakantie, vakantieLabel, baan, baanLabel, doel, doelLabel }`.

Extra books added by the user: `localStorage('extra-books')` → JSON array of strings.

Weekly reviews: `localStorage('review-YYYY-MM-DD')` → review object.

## Key constants (all in `<script>`)

| Constant | Purpose |
|---|---|
| `HABITS` | Array of `{ id, name, target }` — the 7 daily habits |
| `WORKOUTS` | Array of `{ id, label, cls }` — workout types for logging |
| `METRICS_DEF` | Array of `{ id, label, unit, max, goal }` — metric inputs |
| `WEEK_PLAN` | 7-day schedule mapping days to sessions |
| `PLAN_SESSIONS` | Full workout detail for each session type |
| `W_COLOR` / `W_BG` | Color maps for workout badges |
| `COLOR_MAP` | Maps color names to `[bg, fg]` CSS variable pairs |

## Navigation pattern

```js
goto('habits')  // shows page-habits, hides all others, triggers render
swTab(btn, 'inv-metrics', 'inv-tabs')  // switches tab within a page
```

`goto()` calls the appropriate `render*()` functions on demand. Charts are only initialized when their page is first visited.

## Editing content

- **Habits list:** edit the `HABITS` array
- **Workout plan:** edit `WEEK_PLAN` (schedule) and `PLAN_SESSIONS` (exercise details)
- **Goals / dagstructuur / doelen pages:** edit the HTML directly — these are static content blocks
- **Countdown dates:** edit `getTargets()` defaults or they can be overridden via `localStorage('life_targets')`

## Token efficiency rules

- Always use `Edit` (not `Write`) for changes to existing files — only sends the diff
- Never re-read a file after editing it
- Split large changes into multiple small `Edit` calls rather than rewriting the whole file
- Use `Bash` only for git operations and shell tasks, not for reading/writing files
