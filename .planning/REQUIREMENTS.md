# v1 Requirements — Kids Math Game

## v1 Requirements

### Game Flow

- [ ] **GAME-01**: User can start a new game from a start screen
- [ ] **GAME-02**: Game presents 10 puzzles sequentially, one at a time
- [ ] **GAME-03**: Answering correctly advances to the next puzzle
- [ ] **GAME-04**: Answering incorrectly ends the game immediately (game over)
- [ ] **GAME-05**: Timer expiring ends the game immediately (game over)
- [ ] **GAME-06**: Completing all 10 puzzles shows a results screen with score and Play Again button
- [ ] **GAME-07**: Game over screen shows score achieved and a Play Again button
- [ ] **GAME-08**: Play Again fully resets all game state and starts a fresh game

### Puzzle Generation

- [x] **PUZZLE-01**: Each puzzle is a random addition or subtraction expression
- [x] **PUZZLE-02**: All puzzle results are non-negative integers less than 20
- [x] **PUZZLE-03**: Subtraction operands are constrained so the result is never negative (b ≤ a)
- [x] **PUZZLE-04**: 10 puzzles are generated fresh each game session
- [x] **PUZZLE-05**: Puzzle operands use only non-negative integers

### Input

- [ ] **INPUT-01**: Kid enters answer via on-screen numpad buttons (digits 0–9)
- [ ] **INPUT-02**: Current entered digits are displayed as the kid taps
- [ ] **INPUT-03**: A Submit button confirms the answer
- [ ] **INPUT-04**: A backspace/clear button lets kid correct a mis-tap
- [ ] **INPUT-05**: Submitting an empty answer is ignored (no accidental game over)

### Timer

- [ ] **TIMER-01**: Each puzzle has a 15-second countdown timer
- [ ] **TIMER-02**: Timer is displayed as a shrinking progress bar
- [ ] **TIMER-03**: Progress bar shifts color: green → yellow → red as time runs out
- [ ] **TIMER-04**: Timer resets to 15 seconds for each new puzzle
- [ ] **TIMER-05**: Timer stops immediately when an answer is submitted

### UI & Visual Design

- [ ] **UI-01**: Fun, kid-friendly emoji and symbols used throughout (e.g., ⭐ 🎉 😊 🔢)
- [x] **UI-02**: Large, readable fonts suitable for young children (≥ 16px base, puzzle text much larger)
- [x] **UI-03**: Touch targets (numpad buttons) are large enough for small fingers (≥ 44px)
- [ ] **UI-04**: Correct answer triggers a visible positive feedback animation/flash
- [ ] **UI-05**: Results screen shows score in a celebratory way (e.g., star rating or big emoji)
- [x] **UI-06**: Layout works on both mobile phones and desktop browsers

### Deployment

- [x] **DEPLOY-01**: Entire game is self-contained in a single `index.html` file
- [x] **DEPLOY-02**: No external dependencies (no CDN links, no npm packages)
- [ ] **DEPLOY-03**: Game runs correctly when served from GitHub Pages

---

## v2 Requirements (Deferred)

- Personal best score saved in localStorage — easy win, creates replay motivation
- Tab visibility pause (pause timer when child switches apps)
- Sound effects — blocked by browser autoplay policy; needs user gesture
- Multiple difficulty levels
- Score history / leaderboard

---

## Out of Scope

- User accounts — no backend
- Multiplication / division — too advanced for target age (5–9)
- Tutorial / instructions screen — game must be self-explanatory
- External fonts or icon libraries — keep single-file constraint
- Timer pause on tab switch — defer to v2

---

## Traceability

| REQ-ID | Phase | Status |
|--------|-------|--------|
| GAME-01 | Phase 2: Core Game Loop | Pending |
| GAME-02 | Phase 2: Core Game Loop | Pending |
| GAME-03 | Phase 2: Core Game Loop | Pending |
| GAME-04 | Phase 2: Core Game Loop | Pending |
| GAME-05 | Phase 3: Timer Subsystem | Pending |
| GAME-06 | Phase 2: Core Game Loop | Pending |
| GAME-07 | Phase 2: Core Game Loop | Pending |
| GAME-08 | Phase 2: Core Game Loop | Pending |
| PUZZLE-01 | Phase 1: Foundation | Complete |
| PUZZLE-02 | Phase 1: Foundation | Complete |
| PUZZLE-03 | Phase 1: Foundation | Complete |
| PUZZLE-04 | Phase 1: Foundation | Complete |
| PUZZLE-05 | Phase 1: Foundation | Complete |
| INPUT-01 | Phase 2: Core Game Loop | Pending |
| INPUT-02 | Phase 2: Core Game Loop | Pending |
| INPUT-03 | Phase 2: Core Game Loop | Pending |
| INPUT-04 | Phase 2: Core Game Loop | Pending |
| INPUT-05 | Phase 2: Core Game Loop | Pending |
| TIMER-01 | Phase 3: Timer Subsystem | Pending |
| TIMER-02 | Phase 3: Timer Subsystem | Pending |
| TIMER-03 | Phase 3: Timer Subsystem | Pending |
| TIMER-04 | Phase 3: Timer Subsystem | Pending |
| TIMER-05 | Phase 3: Timer Subsystem | Pending |
| UI-01 | Phase 2: Core Game Loop | Pending |
| UI-02 | Phase 1: Foundation | Complete |
| UI-03 | Phase 1: Foundation | Complete |
| UI-04 | Phase 4: Polish & Delight | Pending |
| UI-05 | Phase 4: Polish & Delight | Pending |
| UI-06 | Phase 1: Foundation | Complete |
| DEPLOY-01 | Phase 1: Foundation | Complete |
| DEPLOY-02 | Phase 1: Foundation | Complete |
| DEPLOY-03 | Phase 1: Foundation | Pending |

---
*Last updated: 2026-03-23 after requirements definition*
