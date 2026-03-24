# Project Retrospective

*A living document updated after each milestone. Lessons feed forward into future planning.*

## Milestone: v1.0 — MVP

**Shipped:** 2026-03-24
**Phases:** 1 (partial, known gaps) | **Plans:** 2 (1 executed) | **Sessions:** 1

### What Was Built
- Complete `index.html` (~590 lines): CSS design system, PuzzleGenerator, all 4 screens, GameController, SoundManager
- Web Audio API sound layer: background pentatonic melody + 4 contextual sound effects (no external files)
- Full game loop: 10-puzzle sequence, timer countdown, correct/wrong handling, game-over with answer reveal, win screen with all answers
- Vietnamese UI throughout
- Live on GitHub Pages: https://nvtran61091.github.io

### What Worked
- Single-file constraint forced clean module separation (CONFIG → PuzzleGenerator → GameState → UIRenderer → GameController)
- Implementing Phase 2/3 game logic early (ahead of their planned phases) made the v1.0 demo immediately playable for verification
- Web Audio API via `setTimeout` loop (not `setInterval`) avoided background music timing bugs
- Lazy `AudioContext` init on first user gesture avoided browser autoplay warnings

### What Was Inefficient
- Milestone marked complete with only 1/4 phases formally executed — Phases 2–4 were built informally as part of Phase 1 debugging/feedback rather than through the GSD planning workflow
- No SUMMARY.md for Plan 01-02 (GitHub Pages deploy) — deploy was done manually outside GSD executor
- Requirements in REQUIREMENTS.md reflect the 4-phase waterfall plan; actual delivery leapfrogged to full playable game in Phase 1

### Patterns Established
- `UIRenderer` receives all data as arguments — never reads `GameState` directly (good separation)
- Timer: always `clearInterval` before `setInterval`; `checkAnswer` stops timer before processing
- Touch numpad via `data-value` on a `<div>` (not `<input>`) avoids iOS keyboard popup

### Key Lessons
1. For small single-file projects, formal phase gates slow things down — better to build iteratively and use GSD phases for tracking, not gating
2. Browser autoplay policy: initialize `AudioContext` lazily on first tap, not on page load
3. `MAX_RESULT` should be defined once in `CONFIG` — making it a constant made the user's "change to 9" fix a 1-line change

### Cost Observations
- Model mix: ~60% Sonnet (execution), ~30% Opus (planning), ~10% Haiku (exploration)
- Sessions: 1 extended session
- Notable: Most of the v1 game was shipped in direct conversation rather than through formal GSD executor phases

---

## Cross-Milestone Trends

### Process Evolution

| Milestone | Sessions | Phases | Key Change |
|-----------|----------|--------|------------|
| v1.0 | 1 | 1 formal + 3 informal | Established single-file architecture |

### Top Lessons (Verified Across Milestones)

1. Small projects benefit from lighter GSD workflow — plan phases but execute conversationally
2. Define all magic numbers in `CONFIG` object from day 1 — makes later adjustments trivial
