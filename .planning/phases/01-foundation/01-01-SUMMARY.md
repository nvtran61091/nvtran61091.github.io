---
phase: 01-foundation
plan: "01"
subsystem: index.html
tags: [html, css, javascript, puzzle-generator, ui-renderer, foundation]
dependency_graph:
  requires: []
  provides: [index.html, CSS-design-system, PuzzleGenerator, UIRenderer, GameController-stubs]
  affects: [01-02-PLAN.md, Phase-2-Core-Game-Loop]
tech_stack:
  added: [HTML5, Vanilla-JS-ES2020, inline-CSS, CSS-custom-properties, clamp(), 100dvh]
  patterns: [plain-object-modules, innerHTML-swap, constrained-puzzle-generation, dedup-Set]
key_files:
  created: [index.html]
  modified: []
decisions:
  - "Zero-answer puzzles filtered (answer === 0 continue) — young children may not recognize 0 as valid"
  - "Subtraction constraint: b = Math.floor(Math.random() * (a + 1)) guarantees b <= a and answer >= 0"
  - "Dedup via Set keyed on '${a}${op}${b}' — prevents duplicate expressions per game session"
  - "font-size: 16px on all inputs — iOS Safari auto-zooms viewports below 16px threshold"
  - "min-height: 100dvh (not 100vh) — dvh accounts for dynamic mobile browser chrome"
  - "numpad-btn min-height: 64px (exceeds 44px a11y minimum) — fine motor imprecision in ages 5–9"
  - "No maximum-scale in viewport meta — WCAG 1.4.4 accessibility requirement"
metrics:
  duration_minutes: 2
  tasks_completed: 1
  tasks_total: 2
  files_created: 1
  files_modified: 0
  completed_date: "2026-03-23"
---

# Phase 1 Plan 1: Build complete index.html — CSS design system, PuzzleGenerator, static screens

**One-liner:** Single-file HTML5 game shell with constrained puzzle generation (dedup+zero-filter+subtraction constraint), 8-token CSS design system with fluid typography, and all 4 UIRenderer static screens.

---

## Objective

Build the complete `index.html` — the single file that is the entire game application for all four phases. Establishes the CSS design system, puzzle generation engine, static screen templates, and module architecture that Phases 2–4 will extend. Nothing in this plan needs to be undone or redone later — every decision here is final architecture.

---

## Tasks Completed

| Task | Name | Commit | Files | Status |
|------|------|--------|-------|--------|
| 1 | Build complete index.html — CSS design system, puzzle engine, static screens | 2ee2fa7 | index.html (366 lines) | ✅ Done |
| 2 | Local browser verification — all 4 screens, DevTools, console puzzle stress test | — | — | ⏸ Awaiting human verification |

---

## What Was Built

### index.html (~366 lines, fully self-contained)

**CSS Design System** (`<style>` block):
- 8 CSS custom properties: `--color-bg`, `--color-primary`, `--color-correct`, `--color-wrong`, `--color-text`, `--color-card`, `--color-accent`, `--color-timer-mid`
- Fluid typography via `clamp()`: `--font-base`, `--font-large`, `--font-puzzle`, `--font-emoji`
- `min-height: 100dvh` body (dvh handles iOS/Android browser chrome)
- `width: min(480px, 100%)` game card (responsive, no breakpoints)
- Button touch targets: `min-height: 56px` (general), `min-height: 64px` (numpad)
- `font-size: 16px` on `input` elements (iOS auto-zoom prevention)
- CSS transitions on timer bar fill (width + background color)

**JavaScript Modules** (dependency order in single `<script>` block):
1. `CONFIG` — `PUZZLE_COUNT: 10`, `TIMER_SECONDS: 15`, `MAX_RESULT: 20`, `OPERATORS: ['+', '-']`
2. `PuzzleGenerator.generate(count)` — pure function, all three guards present
3. `GameState` — stub with phase, puzzles, currentIndex, score, timeRemaining, timerID
4. `TimerController` — Phase 3 stubs
5. `UIRenderer` — 5 methods: renderStart, renderQuestion, renderGameOver, renderResults, updateTimer
6. `GameController` — init() + playAgain() implemented; Phase 2/3 stubs declared
7. Boot — `GameController.init()` called immediately

**Puzzle Generation Algorithm (all 3 guards):**
```javascript
// Guard 1: subtraction b <= a (no negative answers)
b = Math.floor(Math.random() * (a + 1));  // 0–a
// Guard 2: zero-answer filter
if (answer === 0) continue;
// Guard 3: dedup Set keyed on expression string
if (seen.has(key)) continue;
```

---

## Verification Results

All 18 automated checks passed ✅:
- `--color-bg: #FFF9C4` token ✅
- `min-height: 100dvh` ✅
- `min-height: 64px` numpad targets ✅
- `font-size: 16px` iOS prevention ✅
- `answer === 0` zero-answer filter ✅
- `seen.has(key)` dedup Set ✅
- `Math.random() * (a + 1)` subtraction constraint ✅
- `const PuzzleGenerator` module ✅
- All 5 UIRenderer methods (renderStart, renderQuestion, renderGameOver, renderResults, updateTimer) ✅
- No `<link>` external CSS ✅
- No `<script src=` external JS ✅
- No `@import url` external fonts ✅
- No `maximum-scale` accessibility violation ✅
- `GameController.init()` boot call ✅

---

## Deviations from Plan

None — plan executed exactly as written. All exact CSS values, JS patterns, and module contracts from the plan specification were used verbatim.

---

## Decisions Made

| Decision | Rationale |
|----------|-----------|
| Filtered zero-answer puzzles (`answer === 0` → `continue`) | Resolves the open question from STATE.md: young children may not recognize 0 as a valid answer; filtering prevents confusion |
| All exact CSS values from plan spec used verbatim | Plan specified precise hex colors, px values, and clamp() ranges — no substitutions needed |
| Single `<script>` block in dependency order | Ensures no forward-reference errors: CONFIG → PuzzleGenerator → GameState → TimerController → UIRenderer → GameController → Boot |

---

## Awaiting Human Verification (Task 2)

Task 2 is a `checkpoint:human-verify`. The following must be verified in a browser before Plan 02 proceeds:

1. Open `index.html` in Chrome — confirm warm-yellow background, white card, 🔢 emoji, orange ▶ Play! button
2. DevTools iPhone SE (375×667) emulation — no horizontal scroll, no clipped content
3. Console puzzle stress test (50 runs) — paste script from PLAN.md Task 2, confirm zero assertion errors
4. All 4 UIRenderer screens render from console
5. Numpad buttons ≥ 64px height in DevTools computed styles
6. Timer bar color test (`updateTimer(15)` → green, `updateTimer(8)` → amber, `updateTimer(3)` → red)

---

## Self-Check

### Created Files
- `index.html` — exists ✅ (366 lines)
- `.planning/phases/01-foundation/01-01-SUMMARY.md` — this file

### Commits
- `2ee2fa7` — feat(01-01): build complete index.html

## Self-Check: PASSED
