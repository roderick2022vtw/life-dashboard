# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file personal life dashboard (Dutch). No build step, no dependencies to install, no test suite. Everything lives in `index.html`.

**Live URL:** https://roderick2022vtw.github.io/life-dashboard/
**Deploy:** `git add -A && git commit -m "..." && git push` — GitHub Pages auto-deploys from `main` root within ~1 min.

## Architecture

`index.html` is structured in three sections:

1. **`<style>`** — All CSS using custom properties (`--bg`, `--accent`, `--blue`, `--teal`, etc.). Dark theme. Layout: sidebar (220px fixed) + scrollable main. Responsive breakpoint at 700px hides the sidebar and shows a bottom tab bar instead.

2. **`<main>`** — One `<div id="page-{id}" class="page">` per section. Only the `.active` page is visible. Pages: `today`, `habits`, `week`, `maand`, `plan`, `import`, `spaans`, `investing`, `reading`, `dagstructuur`, `review`, `doelen`.

3. **`<script>`** — All logic. Key globals:
   - `data` — the full in-memory state, persisted to `localStorage('fitdash_v2')` as JSON
   - `selectedDate` — which date the habits/workouts/metrics panels are editing (defaults to today)
   - `selectedPlanSession` — which workout is shown in the plan detail panel (default: `'push'`)
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
| `COLOR_MAP` | Maps color names to `[bg, fg]` CSS variable pairs — includes: `blue`, `green`, `amber`, `purple`, `teal`, `gray` |

## Training schema (current — Hybrid PPL)

| Dag | Sessie | Label |
|-----|--------|-------|
| Ma  | push   | Push — borst, schouders, triceps |
| Di  | run1   | Run — Zone 2 |
| Wo  | pull   | Pull — rug, biceps |
| Do  | legs   | Legs — quads, hamstrings, glutes |
| Vr  | upper  | Upper — combi druk + trek (volume) |
| Za  | bike   | Flex / Kiten / Ride |
| Zo  | rest   | Rust |

**WORKOUTS array** (for logging): `push`, `pull`, `legs`, `upper`, `run1`, `bike`, `rest`

**PLAN_SESSIONS** keys: `push`, `pull`, `legs`, `upper`, `run1`, `run2`, `bike`, `rest`
- `push` / `pull` / `legs` / `upper` — gym sessions with `exercises[]`
- `run1` / `run2` / `bike` — cardio sessions with `runBlocks[]` and `isRun:true`
- `rest` — rest day with `restItems[]` and `isRest:true`

**Type labels in weekschema** are derived from session ID (not color):
```js
const GYM = ['push','pull','legs','upper','upper_a','upper_b','lower_a','lower_b'];
typeLabel = session==='rest' ? 'rust' : GYM.includes(session) ? 'gym' : session==='bike' ? 'flex' : 'cardio';
```

## Navigation pattern

```js
goto('habits')  // shows page-habits, hides all others, triggers render, syncs sidebar + bottom nav
swTab(btn, 'inv-metrics', 'inv-tabs')  // switches tab within a page
```

`goto()` calls the appropriate `render*()` functions on demand. Charts are only initialized when their page is first visited. `goto()` also syncs `.bnav-item.active` for the mobile bottom nav.

## Mobile navigation

On screens ≤ 700px the sidebar is hidden and a **bottom tab bar** (`.bottom-nav`) appears with 5 items:
- 🏠 Vandaag · ✅ Habits · 💪 Plan · 📋 Systeem · ☰ Meer

"Meer" opens a modal overlay (`#mob-menu`) with all remaining pages (Week, Maand, Spaans, Investeren, Lezen, Review, Doelen, Import). Functions: `openMobMenu()` / `closeMobMenu()`.

## Service worker (sw.js)

PWA with **network-first strategy for HTML** — always fetches fresh `index.html` from GitHub Pages, falls back to cache only when offline. Static assets (manifest, icon) use cache-first. Current cache version: `life-os-v3`.

**Important:** bump the cache version in `sw.js` whenever a breaking change requires a forced cache bust on existing installs.

## Editing content

- **Habits list:** edit the `HABITS` array
- **Training sessions:** edit `WEEK_PLAN` (schedule) and `PLAN_SESSIONS` (exercise details). When adding a new session type, also add it to `WORKOUTS`, `W_COLOR`, `W_BG`, and the `GYM` array in `renderPlan()` if it's a gym session.
- **Goals / dagstructuur / doelen pages:** edit the HTML directly — these are static content blocks
- **Countdown dates:** edit `getTargets()` defaults or they can be overridden via `localStorage('life_targets')`
- **Dagstructuur → Systeemregels tab:** contains Atomic Habits principles (habit stacking, Instagram friction/blokkeren). Edit the HTML directly.

## Token efficiency rules

- Always use `Edit` (not `Write`) for changes to existing files — only sends the diff
- Never re-read a file after editing it
- Split large changes into multiple small `Edit` calls rather than rewriting the whole file
- Use `Bash` only for git operations and shell tasks, not for reading/writing files
- After any change, sanity-check that labels, colors, and session IDs are consistent across `WORKOUTS`, `W_COLOR`, `W_BG`, `WEEK_PLAN`, `PLAN_SESSIONS`, `COLOR_MAP`, and the `GYM` array in `renderPlan()`
