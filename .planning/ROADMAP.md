# Roadmap — Kids Math Game

## Project

A fun, browser-based math game for young children. 10 random addition/subtraction puzzles, 15-second countdown timer per puzzle, single `index.html`, deployed on GitHub Pages.

**Core value:** A delightful, no-friction math game kids can play instantly in any browser.

---

## Phases

- [ ] **Phase 1: Foundation** — Puzzle logic, static UI prototype, deployment wiring
- [ ] **Phase 2: Core Game Loop** — Fully playable game from start to finish (no timer yet)
- [ ] **Phase 3: Timer Subsystem** — Live countdown, color bar, timeout game-over
- [ ] **Phase 4: Polish & Delight** — Answer feedback animations, celebratory results screen

---

## Phase Details

### Phase 1: Foundation
**Goal**: A browsable static prototype with valid puzzle generation, kid-friendly visual design, and a verified deployment path to GitHub Pages.
**Depends on**: Nothing (first phase)
**Requirements**: PUZZLE-01, PUZZLE-02, PUZZLE-03, PUZZLE-04, PUZZLE-05, UI-02, UI-03, UI-06, DEPLOY-01, DEPLOY-02, DEPLOY-03
**Success Criteria** (what must be TRUE):
  1. Opening `index.html` in a browser shows a game start screen with large, readable fonts, kid-friendly colors, and emoji
  2. Puzzle generator produces 10 valid expressions verifiable in the browser console — all results non-negative integers < 20, no duplicates, no negative operands
  3. All four game screens (start, question, game-over, results) render with correct layout on both a mobile phone and a desktop browser
  4. Numpad buttons measure at least 44 × 44px in DevTools and are visually obvious on a phone-sized viewport
  5. Pushing `index.html` to GitHub Pages serves the game with no errors in under 60 seconds
**Plans**: TBD

---

### Phase 2: Core Game Loop
**Goal**: A fully playable game — start, answer 10 puzzles with an on-screen numpad, see correct/wrong outcomes, reach the results screen, and play again — all without time pressure.
**Depends on**: Phase 1
**Requirements**: GAME-01, GAME-02, GAME-03, GAME-04, GAME-06, GAME-07, GAME-08, INPUT-01, INPUT-02, INPUT-03, INPUT-04, INPUT-05, UI-01
**Success Criteria** (what must be TRUE):
  1. Tapping Start immediately shows the first puzzle with a progress indicator (e.g., "Question 1 of 10") and emoji/kid-friendly chrome
  2. Tapping numpad digits shows them accumulating on screen; backspace removes the last digit; submitting while empty does nothing
  3. Correct answer advances to the next puzzle and increments the progress counter
  4. Submitting a wrong answer ends the game and shows the game-over screen with the score achieved and a Play Again button
  5. Completing all 10 puzzles shows the results screen with the final score and a Play Again button; tapping Play Again fully resets all state and starts a fresh game with new puzzles
**Plans**: TBD

---

### Phase 3: Timer Subsystem
**Goal**: Every puzzle has a live 15-second countdown that visually depletes and immediately ends the game on expiry.
**Depends on**: Phase 2
**Requirements**: TIMER-01, TIMER-02, TIMER-03, TIMER-04, TIMER-05, GAME-05
**Success Criteria** (what must be TRUE):
  1. A progress bar visibly depletes from full to empty during the 15 seconds of each puzzle
  2. The bar shifts color — green while plenty of time remains, yellow at mid-range, red when nearly out of time
  3. Letting the timer reach zero immediately ends the game and shows the game-over screen (no extra keypress needed)
  4. Submitting a correct answer resets the bar to full green for the next puzzle; submitting any answer stops the countdown immediately
  5. Playing through multiple rounds via Play Again never triggers a premature game-over caused by a stale timer from a previous round
**Plans**: TBD

---

### Phase 4: Polish & Delight
**Goal**: Correct answers feel rewarding and completing the game feels celebratory.
**Depends on**: Phase 3
**Requirements**: UI-04, UI-05
**Success Criteria** (what must be TRUE):
  1. A correct answer triggers a visible green flash or animation (< 300 ms) before the next puzzle appears — distinguishable from no feedback at all
  2. The results screen displays the score with clear celebration — stars, a large emoji, or equivalent — so a child knows they did well
**Plans**: TBD

---

## Progress

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation | 0/? | Not started | - |
| 2. Core Game Loop | 0/? | Not started | - |
| 3. Timer Subsystem | 0/? | Not started | - |
| 4. Polish & Delight | 0/? | Not started | - |

---

## Requirement Coverage

| REQ-ID | Phase |
|--------|-------|
| PUZZLE-01 | Phase 1 |
| PUZZLE-02 | Phase 1 |
| PUZZLE-03 | Phase 1 |
| PUZZLE-04 | Phase 1 |
| PUZZLE-05 | Phase 1 |
| UI-02 | Phase 1 |
| UI-03 | Phase 1 |
| UI-06 | Phase 1 |
| DEPLOY-01 | Phase 1 |
| DEPLOY-02 | Phase 1 |
| DEPLOY-03 | Phase 1 |
| GAME-01 | Phase 2 |
| GAME-02 | Phase 2 |
| GAME-03 | Phase 2 |
| GAME-04 | Phase 2 |
| GAME-06 | Phase 2 |
| GAME-07 | Phase 2 |
| GAME-08 | Phase 2 |
| INPUT-01 | Phase 2 |
| INPUT-02 | Phase 2 |
| INPUT-03 | Phase 2 |
| INPUT-04 | Phase 2 |
| INPUT-05 | Phase 2 |
| UI-01 | Phase 2 |
| TIMER-01 | Phase 3 |
| TIMER-02 | Phase 3 |
| TIMER-03 | Phase 3 |
| TIMER-04 | Phase 3 |
| TIMER-05 | Phase 3 |
| GAME-05 | Phase 3 |
| UI-04 | Phase 4 |
| UI-05 | Phase 4 |

**Coverage: 32/32 ✓**

---

*Last updated: 2026-03-23 after roadmap creation*
