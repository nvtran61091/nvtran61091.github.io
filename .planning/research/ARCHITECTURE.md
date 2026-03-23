# Architecture Patterns

**Domain:** Single-file browser kids math game
**Researched:** 2025-01-30
**Confidence:** HIGH (vanilla JS game loops are well-established patterns; no external dependencies to verify)

---

## Recommended Architecture

A **module-object pattern** inside a single `<script>` block. Each concern is a plain JavaScript object literal with a clear, named API. No classes, no frameworks, no build step. All screens are rendered into one `<div id="app">` root by swapping innerHTML templates.

```
index.html
├── <style>        ← all CSS inline
├── <div id="app"> ← single render target; all screens live here
└── <script>
    ├── CONFIG           { PUZZLE_COUNT, TIMER_SECONDS, MAX_RESULT }
    ├── PuzzleGenerator  { generate() }
    ├── GameState        { phase, puzzles[], currentIndex, score, timeRemaining, timerID }
    ├── TimerController  { start(), stop(), reset() }
    ├── UIRenderer       { renderStart(), renderQuestion(), renderGameOver(), renderResults() }
    └── GameController   { init(), startGame(), checkAnswer(), onTimeout(), playAgain() }
```

---

## State Machine

Five phases. Each phase has exactly one screen owner in UIRenderer.

```
         ┌─────────────────────────────┐
         │                             │
         ▼                             │ playAgain()
┌──────────────┐  startGame()  ┌──────────────────┐
│    START     │ ────────────► │    QUESTION       │
│  (welcome)   │               │  (active puzzle)  │
└──────────────┘               └──────────────────┘
                                    │         │
                       correct +    │         │  wrong answer
                       last puzzle  │         │  OR timer expires
                                    ▼         ▼
                           ┌─────────┐   ┌──────────┐
                           │ RESULTS │   │ GAME_OVER│
                           │(win!)   │   │(lose)    │
                           └─────────┘   └──────────┘
                                │              │
                                └──────────────┘
                                    playAgain()
                                        │
                                        ▼
                                     START
```

### State Transition Table

| From        | Event                            | To          | Side-effects                                    |
|-------------|----------------------------------|-------------|-------------------------------------------------|
| START       | Player clicks "Play"             | QUESTION    | generate puzzles, reset score, render Q #1      |
| QUESTION    | Correct answer, puzzle < 10      | QUESTION    | increment index, stop timer, render next Q      |
| QUESTION    | Correct answer, puzzle == 10     | RESULTS     | stop timer, render results screen               |
| QUESTION    | Wrong answer submitted           | GAME_OVER   | stop timer, render game over screen             |
| QUESTION    | Timer reaches 0                  | GAME_OVER   | stop timer, render game over screen             |
| GAME_OVER   | Player clicks "Play Again"       | START       | render start screen                             |
| RESULTS     | Player clicks "Play Again"       | START       | render start screen                             |

---

## Component Boundaries

### 1. `CONFIG` — Constants Only
```javascript
const CONFIG = {
  PUZZLE_COUNT: 10,
  TIMER_SECONDS: 15,
  MAX_RESULT: 20,       // exclusive upper bound on answers
  OPERATORS: ['+', '-']
};
```
**Responsibility:** Single source of truth for all magic numbers.  
**Communicates with:** PuzzleGenerator, TimerController, UIRenderer (reads values, never writes).

---

### 2. `PuzzleGenerator` — Pure Function
```javascript
const PuzzleGenerator = {
  generate(count) { ... }  // returns Puzzle[]
};

// Puzzle shape:
// { operandA: 7, operator: '+', operandB: 5, answer: 12 }
```
**Responsibility:** Produce an array of `count` valid puzzle objects.  
**Rules enforced here (not elsewhere):**
- `answer = operandA ± operandB`
- `answer >= 0` and `answer < CONFIG.MAX_RESULT`
- `operandA >= 0`, `operandB >= 0`

**Communicates with:** Nothing. Called once by `GameController.startGame()`.  
**No DOM access. No side-effects.** Easily testable in the browser console.

---

### 3. `GameState` — Shared Mutable State
```javascript
const GameState = {
  phase: 'START',           // 'START' | 'QUESTION' | 'GAME_OVER' | 'RESULTS'
  puzzles: [],              // Puzzle[] — populated at startGame()
  currentIndex: 0,          // 0–9, which puzzle is active
  score: 0,                 // incremented on each correct answer
  timeRemaining: 0,         // current countdown value (seconds)
  timerID: null             // setInterval handle, so TimerController can clear it
};
```
**Responsibility:** The single source of truth for all runtime data.  
**Communicates with:** Read by UIRenderer and TimerController. Written by GameController.  
**Rule:** Only `GameController` mutates GameState. UIRenderer and TimerController only read.

---

### 4. `TimerController` — Timer Lifecycle
```javascript
const TimerController = {
  start(onTick, onExpire) { ... },  // begins setInterval; stores ID in GameState.timerID
  stop()                 { ... },   // clears the interval
};
```
**Responsibility:** Own `setInterval` lifecycle. Decrement `GameState.timeRemaining` on each tick. Fire `onExpire` when it hits zero.  
**Communicates with:**
- Reads `GameState.timerID` to clear old timer
- Writes `GameState.timeRemaining`
- Calls `onTick` callback → UIRenderer updates countdown display
- Calls `onExpire` callback → GameController.onTimeout()

**Critical rule:** Always call `TimerController.stop()` before starting a new timer or transitioning state. Forgetting this causes two timers racing simultaneously — a classic single-file game bug.

---

### 5. `UIRenderer` — DOM Writes Only
```javascript
const UIRenderer = {
  renderStart()               { ... },  // welcome/splash screen
  renderQuestion(puzzle, idx, timeRemaining) { ... },  // active puzzle
  updateTimer(timeRemaining)  { ... },  // only updates countdown element (no full re-render)
  renderGameOver(score)       { ... },  // wrong answer or timeout screen
  renderResults(score)        { ... },  // all 10 solved screen
};
```
**Responsibility:** All innerHTML writes. Zero business logic. Receives data as arguments — never reads GameState directly.  
**Communicates with:** Reads `document.getElementById('app')` as root. Writes innerHTML. Attaches event listeners for buttons and answer input.  
**Rule:** Event listeners wired in render functions must delegate to `GameController` methods — not contain logic themselves. Example: `onclick="GameController.checkAnswer()"`.

**Why `updateTimer` is separate from `renderQuestion`:** Avoids re-rendering the entire puzzle + answer input on every second tick (which would reset what the child typed mid-answer).

---

### 6. `GameController` — Orchestrator
```javascript
const GameController = {
  init()          { ... },  // called on page load; renders start screen
  startGame()     { ... },  // generates puzzles, resets state, shows first question
  showQuestion()  { ... },  // renders current puzzle, starts timer
  checkAnswer()   { ... },  // reads input, validates, transitions state
  onTimeout()     { ... },  // timer expired handler
  playAgain()     { ... },  // resets to start screen
};
```
**Responsibility:** Drive state transitions. Coordinate the other modules.  
**Communicates with:** All other modules. This is the only place state transitions live.

---

## Data Flow

```
                         Page Load
                             │
                             ▼
                    GameController.init()
                             │
                             ▼
                    UIRenderer.renderStart()
                             │
                    [child clicks "Play"]
                             │
                             ▼
                    GameController.startGame()
                       │
                       ├─► PuzzleGenerator.generate(10) ──► GameState.puzzles[]
                       ├─► GameState reset (index=0, score=0)
                       └─► GameController.showQuestion()
                                  │
                                  ├─► UIRenderer.renderQuestion(puzzle, idx, 15)
                                  └─► TimerController.start(onTick, onExpire)
                                              │
                                    ┌─────────┴──────────┐
                              each second            at 0 seconds
                                    │                    │
                                    ▼                    ▼
                           GameState.timeRemaining--   GameController.onTimeout()
                           UIRenderer.updateTimer()         │
                                                            ▼
                                                   GameState.phase = 'GAME_OVER'
                                                   TimerController.stop()
                                                   UIRenderer.renderGameOver(score)

                    [child submits answer]
                             │
                             ▼
                    GameController.checkAnswer()
                       │
                       ├─ WRONG: GameState.phase='GAME_OVER'
                       │         TimerController.stop()
                       │         UIRenderer.renderGameOver(score)
                       │
                       └─ CORRECT:
                             GameState.score++
                             GameState.currentIndex++
                             TimerController.stop()
                             │
                             ├─ if index < 10: GameController.showQuestion()  (loop)
                             └─ if index == 10: UIRenderer.renderResults(score)
```

---

## HTML Structure

Only one element in `<body>` that JavaScript touches:

```html
<body>
  <div id="app">
    <!-- UIRenderer owns all content here; screens swap via innerHTML -->
  </div>
</body>
```

Each `render*()` function replaces `app.innerHTML` completely — except `updateTimer()` which targets only `#timer` within the active question screen.

### Screen HTML Sketches

**Start screen** (`renderStart`)
```html
<div class="screen screen-start">
  <h1>🔢 Math Adventure!</h1>
  <p>Can you solve 10 puzzles? ⏱️</p>
  <button onclick="GameController.startGame()">▶ Play!</button>
</div>
```

**Question screen** (`renderQuestion`)
```html
<div class="screen screen-question">
  <div class="progress">Question 3 / 10 ⭐</div>
  <div id="timer" class="timer">⏱️ 15</div>
  <div class="puzzle">7 + 5 = ?</div>
  <input id="answer-input" type="number" autofocus />
  <button onclick="GameController.checkAnswer()">✅ Submit</button>
</div>
```

**Game over screen** (`renderGameOver`)
```html
<div class="screen screen-gameover">
  <h1>😢 Game Over!</h1>
  <p>You solved <strong>3</strong> puzzles! 🌟</p>
  <button onclick="GameController.playAgain()">🔄 Try Again</button>
</div>
```

**Results screen** (`renderResults`)
```html
<div class="screen screen-results">
  <h1>🎉 You Win!</h1>
  <p>All 10 puzzles solved! 🏆</p>
  <button onclick="GameController.playAgain()">🔄 Play Again</button>
</div>
```

---

## Script Organization Within `<script>` Tag

Sections should appear in dependency order (each section only depends on what's above it):

```javascript
// ── 1. CONFIG ─────────────────────────────────────────────────
const CONFIG = { ... };

// ── 2. PUZZLE GENERATOR ───────────────────────────────────────
// Depends on: CONFIG
const PuzzleGenerator = { ... };

// ── 3. GAME STATE ─────────────────────────────────────────────
// Depends on: nothing (pure data)
const GameState = { ... };

// ── 4. TIMER CONTROLLER ───────────────────────────────────────
// Depends on: GameState, CONFIG
const TimerController = { ... };

// ── 5. UI RENDERER ────────────────────────────────────────────
// Depends on: CONFIG (for display values)
// NOTE: Does NOT depend on GameState directly — data passed as args
const UIRenderer = { ... };

// ── 6. GAME CONTROLLER ────────────────────────────────────────
// Depends on: all of the above
const GameController = { ... };

// ── 7. BOOT ───────────────────────────────────────────────────
GameController.init();
```

---

## Build Order (Recommended)

Build in this sequence — each step produces something runnable/testable:

| Step | What to Build | Why First | Testable How |
|------|---------------|-----------|--------------|
| 1 | `CONFIG` + `PuzzleGenerator` | Zero dependencies; pure logic | `PuzzleGenerator.generate(10)` in browser console |
| 2 | `GameState` skeleton | Data structure all other modules depend on | Inspect object in console |
| 3 | `UIRenderer` static screens | See the UI immediately; no logic needed yet | Hard-code calls like `UIRenderer.renderQuestion(puzzle, 1, 15)` |
| 4 | `GameController.init()` + `startGame()` | Wire start → question flow | Click "Play", see first puzzle |
| 5 | `TimerController` | Adds countdown; most isolated timer risk | Watch timer count down to game over |
| 6 | `GameController.checkAnswer()` + `onTimeout()` | Complete the game loop | Play a full game end-to-end |
| 7 | CSS polish + emoji + mobile layout | Non-blocking; layered on top | Visual review on phone |

**Rationale for this order:** Steps 1-3 give a visible, interactive prototype before any game logic is wired. Timer (Step 5) is isolated after the core question/answer flow works — this is intentional because timer bugs (double-firing, lingering intervals) are the most common source of defects in this type of game.

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: Global Variables Scattered Everywhere
**What goes wrong:** `let currentIndex = 0; let score = 0; let timerID;` declared at top level, mutated anywhere.  
**Why bad:** State mutations become untraceable. Adding any feature requires reading the whole file.  
**Instead:** All mutable state lives in `GameState`. Only `GameController` writes to it.

### Anti-Pattern 2: Timer Never Cleared Before Re-Starting
**What goes wrong:** Moving to next question starts a new `setInterval` without clearing the old one. Two timers fire simultaneously. Game over triggers twice.  
**Why bad:** Causes double-render, score corruption, impossible-to-debug behavior.  
**Instead:** `TimerController.stop()` is always called before `TimerController.start()`. Enforced in `GameController.showQuestion()`.

### Anti-Pattern 3: Full Screen Re-Render Every Timer Tick
**What goes wrong:** Calling `UIRenderer.renderQuestion()` every second (inside the timer tick) re-generates the entire question HTML — which resets the `<input>` field, clearing what the child is typing mid-answer.  
**Why bad:** Extremely frustrating UX for young kids.  
**Instead:** `UIRenderer.updateTimer(n)` updates only `#timer` innerHTML. The question and input are never touched by the timer tick.

### Anti-Pattern 4: Business Logic in onclick Attributes
**What goes wrong:** `<button onclick="GameState.score++; UIRenderer.renderResults()">` — logic leaks into HTML strings.  
**Why bad:** Untestable, unreadable, breaks on rename.  
**Instead:** onclick attributes call only `GameController.*` methods. All logic lives in the script.

### Anti-Pattern 5: UIRenderer Reading GameState Directly
**What goes wrong:** `UIRenderer.renderQuestion()` reads `GameState.currentIndex` and `GameState.timeRemaining` internally.  
**Why bad:** Creates hidden coupling. Renderer can't be tested or called without correct GameState.  
**Instead:** Pass all display data as arguments: `renderQuestion(puzzle, questionNumber, timeRemaining)`.

---

## Scalability Considerations

This is deliberately a single-session toy; no scalability concerns. However:

| Concern | Current approach | If ever needed |
|---------|-----------------|----------------|
| More question types | Add to `CONFIG.OPERATORS` + update `PuzzleGenerator` | Already extensible |
| Difficulty levels | Add `CONFIG.DIFFICULTY` presets | One config object change |
| High score persistence | `localStorage.setItem('highScore', score)` in `renderResults` | No architecture change needed |
| Sound effects | `new Audio(dataURI).play()` inline | Add to UIRenderer; no structure change |
| Multiple pages | Doesn't apply — single-file design is intentional | Would require a build step |

---

## Sources

- Vanilla JS game loop patterns: well-established, HIGH confidence from training data + MDN
- `setInterval` pitfalls (double-firing, stale closures): HIGH confidence, MDN + community wisdom
- Single-file HTML game structure: MEDIUM confidence (no formal spec; derived from common browser game examples and GitHub Pages constraints)
- UX note on input-reset-on-re-render: HIGH confidence — direct consequence of how `innerHTML` assignment works in the DOM
