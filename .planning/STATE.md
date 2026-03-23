# STATE — Kids Math Game

## Project Reference

**Core value:** A delightful, no-friction math game kids can play instantly in any browser.
**Tech stack:** Plain HTML5 + Vanilla JS + inline CSS — single `index.html`, GitHub Pages
**Roadmap:** 4 phases — Foundation → Core Game Loop → Timer Subsystem → Polish & Delight

---

## Current Position

| Field | Value |
|-------|-------|
| **Active phase** | Phase 1: Foundation |
| **Active plan** | None (not yet planned) |
| **Status** | Not started |
| **Last action** | Roadmap created |

```
Progress: [░░░░░░░░░░░░░░░░░░░░] 0%

Phase 1: Foundation          [ ] Not started
Phase 2: Core Game Loop      [ ] Not started
Phase 3: Timer Subsystem     [ ] Not started
Phase 4: Polish & Delight    [ ] Not started
```

---

## Performance Metrics

| Metric | Value |
|--------|-------|
| Phases total | 4 |
| Phases complete | 0 |
| Plans complete | 0 |
| Requirements mapped | 32/32 |

---

## Accumulated Context

### Key Decisions

| Decision | Rationale |
|----------|-----------|
| On-screen numpad (not `<input>`) | Mobile virtual keyboards are unusable for ages 5–7; avoids iOS/Android autocorrect and viewport issues |
| `type="text" inputmode="numeric"` | Prevents `type="number"` allowing `e`, `+`, `-` characters |
| `clearInterval()` before every `setInterval()` | Prevents stale timer accumulation causing premature game-over across rounds |
| `gameActive` guard flag | Prevents race condition between timer expiry and answer submission on the same tick |
| Wall-clock deadline (`Date.now() + 15000`) | Prevents interval drift on low-end devices; poll every 200ms not tick-count |
| `UIRenderer.updateTimer()` separate from `renderQuestion()` | Prevents resetting child's typed answer on every timer tick |
| Single canonical `resetGame()` function | Scattered initialization causes bugs (greyed input, stale score) on Play Again |
| Form submit event only (no onclick on button) | Prevents double puzzle advancement from mixed event binding |

### Architecture Notes

- State machine: `START → QUESTION → GAME_OVER / RESULTS → START`
- Modules: `CONFIG`, `PuzzleGenerator`, `GameState`, `TimerController`, `UIRenderer`, `GameController`
- Only `GameController` writes to `GameState`; `UIRenderer` receives data as arguments
- All screens render into one `<div id="app">` via `innerHTML` swap

### Critical Pitfalls (from research)

1. **Stale interval** — always `clearInterval(timerID)` before `setInterval()`
2. **Race condition** — `gameActive` flag; guard every handler with `if (!gameActive) return`
3. **Negative subtraction results** — pick `a` first, then `b` from `[0, a]`
4. **Play Again leaves stale state** — single `resetGame()` resets everything
5. **`type="number"` allows `e`, `-`, `+`** — use `type="text" inputmode="numeric"`
6. **Enter key double-submission** — `form.addEventListener('submit')` only; no onclick
7. **iOS viewport zoom** — `font-size: 16px` minimum on all `<input>` elements
8. **Duplicate puzzles** — track generated puzzles in a `Set` keyed by `"${a}${op}${b}"`

### Open Questions

- [ ] Filter zero-answer puzzles or allow them? (affects `PuzzleGenerator` — decide in Phase 1)
- [ ] Implement `visibilitychange` timer pause in Phase 3 or document as known limitation?

---

## Session Continuity

**To resume work:**
1. Check which phase is active above
2. Run `/gsd-plan-phase <N>` to create the execution plan for that phase
3. Use `/gsd-work` to begin executing plans

**File locations:**
- Roadmap: `.planning/ROADMAP.md`
- Requirements: `.planning/REQUIREMENTS.md`
- Research: `.planning/research/SUMMARY.md`
- Game file: `index.html` (root)

---

*Last updated: 2026-03-23 after roadmap creation*
