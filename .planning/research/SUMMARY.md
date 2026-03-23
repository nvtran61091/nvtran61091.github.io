# Project Research Summary

**Project:** Kids Math Game (`nvtran61091.github.io`)
**Domain:** Single-file browser-based educational math game for ages 5–9
**Researched:** 2026-03-23
**Confidence:** HIGH — stack is fully constrained by project spec; all four research areas drew from well-established patterns

---

## Executive Summary

This is a single-file, no-backend, no-build-step browser game targeting children ages 5–9 on mobile devices. The entire product lives in one `index.html` deployed to GitHub Pages. The right approach is **Vanilla JS + inline CSS + HTML5 only** — any framework (React, Vue) or build tool (Vite, Parcel) would add complexity that dwarfs the ~250 lines of actual game logic. The architecture follows a classic state-machine pattern with 6 plain object modules: `CONFIG`, `PuzzleGenerator`, `GameState`, `TimerController`, `UIRenderer`, and `GameController`. This is a well-understood pattern for this class of project and carries no technology risk.

The dominant UX decision with the most impact on build order is the **answer input mechanism**: research strongly recommends an **on-screen number pad (0–9 + Submit + Backspace)** instead of a free-form `<input>` field. Ages 5–7 cannot reliably type on mobile virtual keyboards, and iOS/Android keyboards introduce autocorrect, wrong keyboard types, and viewport-covering behavior. This is a medium-complexity feature that should be treated as table stakes, not an enhancement.

The primary technical risks are all in the **timer subsystem**: stale `setInterval` accumulation causing ghost timeouts, a race condition between timer expiry and answer submission, and interval drift on low-end devices. All three are preventable with a `gameActive` guard flag, a wall-clock deadline (not tick counting), and always calling `clearInterval()` before `setInterval()`. None of these are exotic problems — they are documented pitfalls with known solutions.

---

## Key Findings

### Recommended Stack

The project constraints (single `index.html`, GitHub Pages, no build tools) make the stack essentially fixed. Research confirms these constraints align with 2025 best practices for this scope.

**Core technologies:**
- **HTML5** — app shell and input markup; `inputmode="numeric"` triggers mobile numeric keypad natively
- **Vanilla JavaScript (ES2020+)** — game logic, state machine, DOM rendering; no transpile step needed; 97%+ browser support
- **CSS3 (inline `<style>`)** — all visual styling and animations; CSS keyframes run on the compositor thread (smoother than JS animation); inline keeps the file self-contained
- **GitHub Pages** — zero-config static hosting; push `index.html` to `main` → live in ~30 seconds

**What NOT to use:**
- No React/Vue/Svelte — framework runtime dwarfs the application code (~30–100KB overhead for ~250 lines of logic)
- No Vite/Parcel/webpack — build tools break the "just push `index.html`" deployment model
- No external CSS (Tailwind, Bootstrap) — inline custom CSS is faster, smaller, and offline-safe
- No CDN dependencies — CDN failures show kids a spinner instead of a game
- No `<input type="number">` — allows `e`, `+`, `-` characters; use `type="text" inputmode="numeric"` instead

**Key CSS techniques:** CSS custom properties for theming, `clamp()` for responsive typography without media queries, `100dvh` for correct mobile viewport height, CSS `@keyframes` for bounce/shake/pulse animations (no JS animation library needed).

---

### Expected Features

**Must have (table stakes) — v1 blockers:**
- **On-screen number pad (0–9 + Submit + Backspace)** — critical for mobile; free-form input fails for ages 5–7
- **Large equation display** — minimum 48–72px; kids are still learning to read digits
- **Visual countdown timer as progress bar** — depleting bar (not just digits) with green→yellow→red color shift
- **Immediate correct/wrong visual feedback** — green flash + ✅ emoji / red flash + ❌ emoji; must be < 300ms; pair color with emoji for colorblind users (~8% of boys)
- **Progress indicator** ("Question 3 of 10") — without it, kids don't know how close they are to winning
- **Game over screen** — show score so far + empathetic messaging + "Try Again" button
- **Win/results screen** — star rating + celebration emoji (🎉) + "Play Again" button
- **Touch targets ≥ 56×56px** — ages 5–9 have imprecise fine motor control; numpad buttons should be 64–72px
- **Instant game start** — first puzzle appears in one tap from landing screen; no tutorial required

**Should have (differentiators — low cost, ship in v1):**
- Timer pulse/shake animation when < 5 seconds left (pure CSS, trivial)
- Encouragement messages on correct answer ("Awesome! 🌟", "You're on fire! 🔥") — 5 lines of JS
- Personal best via `localStorage` ("New high score! 🏆") — 5–8 lines of JS
- Smooth question fade transition (3 lines of CSS)
- Star rating on score screen (map 10 correct answers to 5 stars)

**Defer to v2+:**
- Streak indicator ("3 in a row! 🔥") — nice, but not essential
- Themed emoji per question (operands as emoji counts — e.g., 🍕+🍕=?) — medium complexity
- Difficulty levels — adds pre-game decision friction; bad for this age group in v1
- Mascot character

**Hard anti-features (never build):**
- Sound effects — autoplay browser restrictions cause silent failures
- Login / leaderboard / accounts — requires backend; destroys open-and-play value proposition
- Skip button — removes game tension
- Negative number answers — cognitively inappropriate for ages 5–9

---

### Architecture Approach

The architecture is a **module-object state machine** inside a single `<script>` block. All screens render into one `<div id="app">` by swapping `innerHTML` — the "poor man's React." Six plain object literals cover all concerns, each with a single responsibility, arranged in dependency order so the file reads top-to-bottom.

**State machine (4 phases):**
```
START ──[Play]──► QUESTION ──[correct × 10]──► RESULTS
                      │                            │
                 [wrong/timeout]                   │
                      ▼                            │
                  GAME_OVER ◄────────────────────────
                      │
                   [Play Again] → START
```

**Major components:**
1. **`CONFIG`** — single source of truth for all magic numbers (`PUZZLE_COUNT=10`, `TIMER_SECONDS=15`, `MAX_RESULT=20`)
2. **`PuzzleGenerator`** — pure function; generates 10 unique, non-negative puzzles; enforces answer constraints; no DOM access
3. **`GameState`** — shared mutable state object (`phase`, `puzzles[]`, `currentIndex`, `score`, `timeRemaining`, `timerID`)
4. **`TimerController`** — owns `setInterval` lifecycle; uses wall-clock deadline (not tick counting); fires `onTick` and `onExpire` callbacks
5. **`UIRenderer`** — all DOM writes; receives data as arguments (never reads `GameState` directly); `updateTimer()` is separate from `renderQuestion()` to avoid resetting the child's typed answer on every tick
6. **`GameController`** — the only module that mutates `GameState`; drives all state transitions; event handlers in HTML call only `GameController.*` methods

**Critical architecture rules:**
- Only `GameController` writes to `GameState`
- `UIRenderer` receives all data as function arguments — no hidden coupling to `GameState`
- `TimerController.stop()` must always be called before `TimerController.start()` — the most common bug source in this type of game
- `UIRenderer.updateTimer()` (not `renderQuestion()`) handles every timer tick — prevents input field reset while child is typing

---

### Critical Pitfalls

1. **Stale interval / double-timer accumulation** (CRITICAL) — Store `timerID` at module level; always call `clearInterval(timerID)` before every `setInterval()` call, including on Play Again. Failing to do this causes game over to trigger 3–8 seconds early after multiple rounds. Detection: does game over fire with visible time remaining on the clock?

2. **Race condition: timer expiry vs. correct answer submission** (CRITICAL) — Introduce a `gameActive` boolean flag; set it to `false` as the first action in `gameOver()`, and guard every handler with `if (!gameActive) return`. Without this, a correct answer submitted on the exact tick of expiry increments score after the game-over screen renders.

3. **Subtraction generating negative answers** (CRITICAL) — When generating subtraction puzzles, pick `a` first then `b` from `[0, a]` so `a - b >= 0` is guaranteed. Simple range-based generation (`Math.random() * 20` for both operands) silently produces age-inappropriate puzzles like `-11 = ?` with no runtime error.

4. **Play Again leaves stale state** (HIGH) — Implement a single canonical `resetGame()` function that resets every piece of state: timer ID, score, puzzle index, `gameActive` flag, fresh puzzle generation, input value, and input `disabled` state. Play Again must call only this one function. Scattered initialization creates bugs where the input is greyed out on game restart.

5. **`type="number"` input allows `e`, `-`, `+`** (HIGH) — Use `type="text" inputmode="numeric" pattern="[0-9]*"` instead. Strip non-digits in the submit handler with `parseInt(val.replace(/[^0-9]/g, ''), 10)`. The HTML spec permits scientific notation in `type="number"`, allowing a child to inadvertently submit `1e2 = 100`.

6. **Enter key double-submission** (HIGH) — Choose one event binding strategy and use it exclusively. Recommended: listen only on `form.addEventListener('submit', e => { e.preventDefault(); handleAnswer(); })`. Never also attach `onclick` to the submit button — mixing both causes double puzzle advancement.

7. **iOS viewport zoom on input focus** (MODERATE) — Set `font-size: 16px` minimum on all `<input>` elements. iOS Safari auto-zooms the viewport when a focused input has `font-size < 16px`, breaking the entire game layout for young kids who can't manually zoom out.

8. **Duplicate puzzles in same game** (MODERATE) — Track generated puzzles in a `Set` keyed by `"${a}${op}${b}"` and reject collisions. The valid puzzle space (~380 combinations) is large enough that 10 unique puzzles never cause infinite loops.

---

## Implications for Roadmap

Research supports a **4-phase build** that incrementally adds runnable layers. Each phase produces something testable before the next phase begins — critical for catching timer and state bugs early.

---

### Phase 1: Foundation — Config, Puzzle Generation & Static UI

**Rationale:** Zero-dependency modules first. `CONFIG`, `PuzzleGenerator`, and `GameState` have no external dependencies and are testable in the browser console before any game logic exists. Static `UIRenderer` screens give a visible prototype immediately without wiring any logic.

**Delivers:** A browsable prototype showing all four screens with hard-coded data. Puzzle generation can be validated in the console.

**Features addressed:**
- All puzzle constraints (non-negative, results < 20, no duplicates, no zero-answer puzzles)
- Large equation display typography
- Kid-friendly visual design (colors, emojis, touch targets)
- All four screen layouts (start, question, game over, results)
- On-screen number pad UI (layout, not logic yet)

**Pitfalls to prevent:**
- P3: Implement constrained subtraction formula from day one
- P9: Add dedup `Set` to `PuzzleGenerator`
- P8: Set `font-size: 16px` on inputs immediately
- P11: Build 64px touch targets into CSS from the start — retrofitting is harder

**Research flag:** Standard patterns — no additional research needed.

---

### Phase 2: Core Game Loop — Controller + Screen Wiring

**Rationale:** Wire `GameController` to drive state transitions without the timer. A complete playable game (Start → Question → Game Over / Results → Play Again) validates the state machine and DOM rendering before the highest-risk component (timer) is introduced.

**Delivers:** A fully playable game with no time pressure — answer questions, see correct/wrong feedback, reach results screen, play again.

**Features addressed:**
- State machine transitions (START → QUESTION → GAME_OVER / RESULTS → START)
- Immediate correct/wrong visual feedback (green/red flash + emoji)
- Progress indicator ("Question X of 10")
- Score tracking and display on results/game-over screens
- On-screen numpad logic (digit accumulation + backspace + submit)
- Play Again full state reset (`resetGame()` canonical function)
- Enter key single-submission via `form.addEventListener('submit')`

**Pitfalls to prevent:**
- P5: Implement the canonical `resetGame()` function now — don't scatter initialization
- P6: Use form submit event exclusively; no onclick on submit button
- P7: Use `type="text" inputmode="numeric"` from the start, not `type="number"`
- P4 (architecture): `UIRenderer.updateTimer()` separate from `renderQuestion()` — build this boundary now even though timer isn't wired yet

**Research flag:** Standard patterns — no additional research needed.

---

### Phase 3: Timer Subsystem

**Rationale:** Timer is isolated to its own phase because it is the highest source of bugs in this type of game. With the game loop already working in Phase 2, the timer can be introduced, tested, and validated independently. All three critical timer pitfalls (stale interval, race condition, tick drift) are addressed here.

**Delivers:** A complete, timed game. Countdown bar animates, game ends on timeout, timer resets correctly on each new question and on Play Again.

**Features addressed:**
- Visual countdown timer as depleting progress bar
- Green→yellow→red color shift as time runs low
- Timer expiry = game over immediately
- Per-puzzle timer reset on correct answer
- Wall-clock deadline math (not tick counting) to prevent drift on low-end devices

**Pitfalls to prevent:**
- P1 (CRITICAL): Module-level `timerID`; `clearInterval()` before every `setInterval()`
- P2 (CRITICAL): `gameActive` guard flag in every handler
- P4: `deadline = Date.now() + 15000`; poll every 200ms instead of counting ticks
- P10: Add `visibilitychange` pause/resume if time permits; document as known limitation if deferred

**Research flag:** Timer patterns are well-documented — no research needed. But this phase warrants extra manual testing on real mobile devices.

---

### Phase 4: Polish & Differentiators

**Rationale:** With a complete, bug-free game loop in place, low-cost delighters can be layered on without risk of destabilizing core logic.

**Delivers:** A polished, delightful experience with celebration animations, encouragement messaging, personal best tracking, and smooth transitions.

**Features addressed:**
- Celebration micro-animation on correct answer (CSS bounce-in keyframe)
- Encouragement messages between questions ("Awesome! 🌟", "You're on fire! 🔥")
- Timer pulse/shake animation when < 5 seconds (pure CSS)
- Personal best via `localStorage` with "New high score! 🏆" celebration
- Smooth fade transition between questions (CSS opacity)
- Star rating on results screen (map score to 1–5 stars)
- Final mobile layout review on real device (not DevTools emulation)

**Pitfalls to prevent:**
- P12: Ensure all feedback pairs color with emoji (not color-only) — verify throughout
- P14: Verify `input.focus()` is called synchronously inside user event handlers on iOS
- P13: Awareness only — hard-reload after push to verify CDN propagation

**Research flag:** Standard patterns — no additional research needed.

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | **HIGH** | Fully constrained by project spec; confirmed by MDN, caniuse, Apple HIG, WCAG |
| Features | **HIGH** | Table stakes grounded in children's HCI principles + spec; differentiators are low-risk additions |
| Architecture | **HIGH** | Vanilla JS state machine is a well-established pattern; no external dependencies to verify |
| Pitfalls | **HIGH** | All pitfalls are documented JavaScript / mobile browser failure modes with verified prevention strategies |

**Overall: HIGH confidence.** This is a well-scoped project with no novel technical territory. Every risk is known and preventable.

### Gaps to address during build

- **On-screen numpad UX** — the exact layout (grid vs. row, backspace placement, display area for typed digits) is not prototyped. Worth sketching on a real phone before committing to HTML structure.
- **Puzzle answer zero (P15)** — decide early whether to filter zero-answer puzzles or allow them. Simple decision but it affects `PuzzleGenerator`.
- **Tab visibility / timer pause (P10)** — decide in Phase 3 whether to implement `visibilitychange` handling or document as a known limitation. Not a correctness bug, but notable UX issue on phones.

---

## Sources

**From STACK.md:**
- MDN Web Docs — `inputmode`, `clamp()`, `dvh`, CSS Custom Properties
- WCAG 2.5.5 — Minimum touch target size
- Apple Human Interface Guidelines — minimum tap target 44pt
- Can I Use (caniuse.com) — browser support data, verified 2025
- PROJECT.md — technology constraints

**From FEATURES.md:**
- Children's HCI/UX literature — principles for ages 5–9
- Pattern analysis: Khan Academy Kids, Prodigy, Math Playground, SplashLearn
- COPPA — children's privacy implications for anti-features
- WCAG 2.1 — touch targets and contrast

**From ARCHITECTURE.md:**
- Vanilla JS game loop patterns — MDN + community patterns
- `setInterval` pitfalls — MDN + common browser game examples

**From PITFALLS.md:**
- MDN Web Docs — `setInterval`/`clearInterval`, Page Visibility API
- Apple WebKit — programmatic `focus()` restrictions on iOS
- Google Material Design — 48dp minimum touch target
- HTML Living Standard — `<input type="number">` permitted characters
- WCAG 1.4.1 — Use of Color
