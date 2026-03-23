# Domain Pitfalls

**Domain:** Single-file browser kids math game (plain HTML + Vanilla JS, GitHub Pages)
**Researched:** 2025-01-30
**Confidence:** HIGH — these are well-established JavaScript timer, input, and mobile browser failure modes

---

## Critical Pitfalls

Mistakes that cause incorrect game behavior, silent data corruption, or rewrites.

---

### Pitfall 1: Stale Interval — Double-Timer Accumulation

**What goes wrong:**
`setInterval()` returns an ID. If `clearInterval(id)` is not called before starting the next
puzzle's timer, the old interval keeps firing silently in the background. After 3 puzzles, 3
simultaneous countdown loops are running. One of them fires `gameOver()` at a seemingly random
moment mid-puzzle.

**Why it happens:**
Developers store `timerId` in a local variable inside a function instead of a module-level
variable, so the ID is lost after the function returns and can never be cleared.

**Consequences:**
- Game ends without the timer visually hitting zero
- Score shows mid-game values on the game-over screen
- Impossible to reproduce reliably (timing-dependent)

**Prevention:**
```javascript
// ONE module-level variable — never declare inside a function
let timerId = null;

function startTimer() {
  clearInterval(timerId);           // always clear before starting
  timerId = setInterval(tick, 1000);
}

function gameOver() {
  clearInterval(timerId);           // always clear on game end
  timerId = null;
  // ...
}

function startGame() {
  clearInterval(timerId);           // also clear at game start (Play Again path)
  // ...
}
```

**Detection (warning signs):**
- Game over triggers with 3–8 seconds still on the visible countdown
- Happens more often the longer a session runs (more accumulated intervals)

**Phase:** Core gameplay loop (puzzle + timer implementation)

---

### Pitfall 2: Race Condition — Timer Expiry vs. Correct Answer Submission

**What goes wrong:**
The player types the correct answer and presses Enter in the same 1000ms tick that the timer
callback fires. Both the answer-submit handler and `gameOver()` execute in the same event loop
turn. `gameOver()` fires, showing the game-over screen — then the answer handler fires,
increments the score and advances the puzzle on a screen the player is no longer on.

**Why it happens:**
`setInterval` callbacks and DOM event listeners both queue in the JavaScript event loop. With no
guard, both fire.

**Consequences:**
- Score is incremented on the game-over screen (wrong final score shown)
- Puzzle index advances past 10, potentially hitting `undefined` puzzle data

**Prevention:**
Use a single `gameActive` boolean as a gate in every handler:

```javascript
let gameActive = false;

function gameOver() {
  gameActive = false;      // ← set FIRST, before any DOM changes
  clearInterval(timerId);
  // show game over screen
}

function handleSubmit() {
  if (!gameActive) return; // ← guard at top — exits if game is over
  // check answer...
}

// setInterval callback
function tick() {
  if (!gameActive) return; // ← same guard
  secondsLeft--;
  if (secondsLeft <= 0) gameOver();
}
```

**Detection:**
- Answer at exactly second 0 — observe whether score is correct on game-over screen
- Add `console.log` to both handlers during testing

**Phase:** Core gameplay loop

---

### Pitfall 3: Subtraction Generating Negative Results

**What goes wrong:**
```javascript
// BUG — can produce negative answers (e.g., 3 - 14 = -11)
const a = Math.floor(Math.random() * 20);
const b = Math.floor(Math.random() * 20);
const result = a - b; // violates "results always < 20, operands non-negative"
```
Kids encounter `-11 = ?` which is confusing and outside the stated requirement.

**Why it happens:**
Developers apply the `< 20` constraint to the operands but forget the result constraint.

**Consequences:**
- Age-inappropriate puzzles (negative arithmetic is not in scope for 5–9 year olds)
- Breaks the promise of "non-negative results"
- No runtime error — silently produces wrong puzzles

**Prevention:**
For subtraction: pick `a` first in the full range, then pick `b` from `[0, a]` to guarantee
`a - b >= 0`. For addition: constrain so `a + b < 20`.

```javascript
function generatePuzzle() {
  const op = Math.random() < 0.5 ? '+' : '-';
  let a, b;
  if (op === '+') {
    a = Math.floor(Math.random() * 19);          // 0–18
    b = Math.floor(Math.random() * (19 - a));    // ensures a+b <= 18
  } else {
    a = Math.floor(Math.random() * 20);          // 0–19
    b = Math.floor(Math.random() * (a + 1));     // ensures b <= a, result >= 0
  }
  return { a, b, op, answer: op === '+' ? a + b : a - b };
}
```

**Detection:**
- Log all 10 generated puzzles to console and scan for negatives
- Run generation 1000 times in a loop and check for any negative `answer`

**Phase:** Puzzle generation

---

### Pitfall 4: `setInterval` Tick Drift on Cheap/Busy Devices

**What goes wrong:**
`setInterval(fn, 1000)` fires approximately every 1000ms — not exactly. On a loaded mobile
browser, a tick may fire at 950ms or 1200ms. After 15 ticks, the visible countdown reaches "0"
but the interval has only fired 13-14 times (or vice versa). The timer "feels off" and can
mismatch the visual display.

**Why it happens:**
`setInterval` is not a real-time clock — it queues a callback after approximately the given
delay. Browser tab throttling (especially on mobile) makes this worse.

**Consequences:**
- Timer expires earlier or later than the visual "0" suggests
- Feels unfair to kids ("but there was still time on the clock!")

**Prevention:**
Record wall-clock start time and derive remaining seconds from `Date.now()` on each tick,
rather than counting ticks:

```javascript
let deadline = null;

function startTimer() {
  clearInterval(timerId);
  deadline = Date.now() + 15000; // 15 seconds from now
  timerId = setInterval(() => {
    const remaining = Math.ceil((deadline - Date.now()) / 1000);
    updateDisplay(remaining);
    if (remaining <= 0) gameOver();
  }, 200); // poll every 200ms for smooth updates
}
```

**Detection:**
- Run on a low-end Android device or throttle CPU in DevTools
- Count visual seconds vs. actual elapsed time

**Phase:** Timer implementation

---

### Pitfall 5: Play Again Leaves Stale State

**What goes wrong:**
The "Play Again" button resets the puzzle index and score but forgets to reset one or more of:
- Timer display (still shows "0" from the last game)
- Input field value (previous answer still visible)
- Input `disabled` state (disabled on game over, not re-enabled)
- `gameActive` flag (still `false` from game over)
- Puzzle array (same 10 puzzles served again instead of freshly generated)

**Why it happens:**
State is scattered — initialized in multiple functions, with Play Again only calling a subset.

**Consequences:**
- Input is greyed out on the new game start — child can't type
- Timer immediately shows "0" before it should
- Score counter starts from previous game value

**Prevention:**
Single `resetGame()` function is the canonical source for all state initialization.
**Every** piece of state is set here, and Play Again calls only this function:

```javascript
function resetGame() {
  clearInterval(timerId);
  timerId = null;
  score = 0;
  puzzleIndex = 0;
  gameActive = false;           // set to true only when first puzzle shows
  puzzles = generatePuzzles();  // fresh set
  inputEl.value = '';
  inputEl.disabled = false;
  timerDisplay.textContent = '15';
}
```

**Detection:**
- Play the game to game-over, then hit Play Again. Is the input disabled?
- Check timer display at the moment of re-start

**Phase:** Game flow / state management

---

### Pitfall 6: Enter Key Double-Submission

**What goes wrong:**
If the answer `<input>` is inside a `<form>` **and** a button has an `onclick` handler,
pressing Enter fires both the `form.submit` event AND the button's `onclick`. The answer gets
processed twice: puzzle advances, then advances again (skipping a puzzle) or score increments
twice.

**Why it happens:**
Mixing two event binding approaches (form submit + button click) is common when code evolves
from a button-click prototype to a keyboard-friendly version.

**Consequences:**
- Puzzles skipped (puzzle index advances by 2)
- Score off by 1
- Game ends on puzzle 9 instead of puzzle 10 (one skip)

**Prevention:**
Choose ONE event binding strategy and use it consistently:

```javascript
// RECOMMENDED: Listen on the form's submit event only
form.addEventListener('submit', (e) => {
  e.preventDefault();   // prevents page reload
  handleAnswer();
});
// Do NOT also put onclick on the submit button
```

**Detection:**
- Press Enter rapidly 10 times from puzzle 1 — does game end correctly on puzzle 10?

**Phase:** Input handling

---

## Moderate Pitfalls

---

### Pitfall 7: `type="number"` Input Allows Invalid Characters

**What goes wrong:**
`<input type="number">` on mobile shows the numeric keypad (good) but the HTML spec permits
`e`, `+`, and `-` in number inputs (scientific notation). A child pressing the wrong key
submits `1e2` (= 100) or `-5` as an answer, causing unexpected behavior.

**Prevention:**
Use `type="text"` with `inputmode="numeric"` and `pattern="[0-9]*"` — this gives the numeric
keypad on mobile while rejecting non-digit characters:

```html
<input type="text" inputmode="numeric" pattern="[0-9]*" autocomplete="off">
```
Strip or reject non-numeric input in the submit handler:
```javascript
const val = parseInt(inputEl.value.replace(/[^0-9]/g, ''), 10);
if (isNaN(val)) return; // empty/invalid — don't process
```

**Phase:** Input handling

---

### Pitfall 8: iOS Viewport Zoom on Input Focus

**What goes wrong:**
iOS Safari automatically zooms the viewport when a text input receives focus if the input's
`font-size` is less than 16px. For a fixed-layout game, this zoom breaks the entire visual
design and is difficult for kids to undo.

**Prevention:**
Set `font-size: 16px` (minimum) on all `<input>` elements. Do NOT use
`maximum-scale=1` in the viewport meta tag — it disables user zoom (accessibility violation).

```css
input {
  font-size: 16px; /* iOS magic threshold — never go below this */
  /* Visual size controlled with padding/width, not font-size */
}
```

**Phase:** UI/styling

---

### Pitfall 9: Duplicate Puzzles in the Same Game

**What goes wrong:**
Purely random generation from a small space (results 0–19) produces duplicates. A child sees
`5 + 3 = ?` twice in the same game. It feels broken — "I already did that one!"

**Prevention:**
Track generated puzzle strings in a `Set` and regenerate on collision:

```javascript
function generatePuzzles(count = 10) {
  const seen = new Set();
  const puzzles = [];
  while (puzzles.length < count) {
    const p = generatePuzzle();
    const key = `${p.a}${p.op}${p.b}`;
    if (!seen.has(key)) {
      seen.add(key);
      puzzles.push(p);
    }
  }
  return puzzles;
}
```

The valid puzzle space (addition: ~171 combinations, subtraction: ~210) is large enough that
10 unique puzzles are always achievable without infinite looping.

**Phase:** Puzzle generation

---

### Pitfall 10: Tab Visibility / Device Lock Freezes Timer

**What goes wrong:**
When the browser tab is hidden (child switches apps, device locks, parent notification), Chrome
and Safari throttle or suspend `setInterval`. On return, the callback fires rapidly to "catch
up." A 15-second timer can appear to expire instantly on return, even if only 2 seconds elapsed
before the tab was hidden.

**Why it happens:**
Browser power-saving and background tab throttling — standard browser behavior since ~2012,
aggressively applied on mobile.

**Consequences:**
- Timer unfairly expires while child is on a different app
- Creates confusion and frustration

**Prevention (v1 — simple):**
Pause the timer when the tab is hidden, resume when visible:

```javascript
document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    clearInterval(timerId);
  } else if (gameActive) {
    // Recalculate deadline to give back the paused time
    deadline = Date.now() + remainingMs;
    startTimer();
  }
});
```

If the pause/resume complexity is undesirable for v1, document the known limitation and
defer — it is a minor UX issue, not a correctness bug.

**Phase:** Timer implementation (can defer to polish phase)

---

### Pitfall 11: Touch Targets Too Small for Young Children

**What goes wrong:**
Ages 5–9 have imprecise fine motor control. Buttons and inputs sized for adults (< 40px tall)
are difficult to hit precisely, causing frustration and mis-taps.

**Prevention:**
- Minimum **48×48px** touch target for all interactive elements (Google's recommendation)
- Large font sizes throughout: `1.5rem` minimum for puzzle numbers, `2rem`+ for the answer input
- Generous padding on submit button (not just large font — actual tappable area)
- Test on a real phone, not browser DevTools device emulation

```css
button {
  min-height: 48px;
  min-width: 48px;
  padding: 12px 24px;
  font-size: 1.25rem;
}
```

**Phase:** UI/styling

---

## Minor Pitfalls

---

### Pitfall 12: Color-Only Feedback for Correct/Wrong

**What goes wrong:**
Flashing the background red or green to indicate wrong/correct is invisible to red-green
colorblind children (~8% of boys). The feedback is lost entirely.

**Prevention:**
Pair every color signal with a semantic emoji or text:
- Correct: ✅ green flash + "✅ Correct!"
- Wrong: ❌ red flash + "❌ Wrong!"
- Timer low: ⏰ + yellow flash

The project already plans emoji usage — use them to carry meaning, not just decoration.

**Phase:** UI/feedback

---

### Pitfall 13: GitHub Pages CDN Caching After Push

**What goes wrong:**
GitHub Pages CDN caches `index.html` aggressively. After pushing a bug fix, the live site
may serve the old version for 5–10 minutes. Developers think their fix didn't work.

**Prevention:**
This is not fixable without a paid CDN configuration (Cloudflare, etc.). Understand it:
- Hard-reload (`Ctrl+Shift+R` / `Cmd+Shift+R`) bypasses local cache
- Wait 10 minutes after push for CDN propagation
- Confirm the fix locally with `file://` before pushing

**Phase:** Deployment (awareness only — no action needed)

---

### Pitfall 14: `input.focus()` Unreliable on iOS After Timeout

**What goes wrong:**
Calling `inputEl.focus()` inside a `setTimeout(..., 100)` — to "wait for the UI to update"
before focusing the input — is blocked by iOS Safari. iOS only allows programmatic `focus()`
calls that are directly triggered by a user gesture (tap/keypress). A delayed focus does nothing,
and the keyboard never appears.

**Consequences:**
Kids on iPhones have to manually tap the input field before every puzzle to bring up the keyboard.

**Prevention:**
Call `focus()` synchronously inside the user event handler, not deferred:

```javascript
function showPuzzle(puzzle) {
  // render puzzle DOM...
  inputEl.value = '';
  inputEl.focus(); // ← synchronous, inside the flow triggered by user action
}
```
If transitioning with CSS animations, trigger `focus()` before the animation starts, not after.

**Phase:** Input handling / mobile UX

---

### Pitfall 15: Puzzle Answer of Zero Feels Wrong to Children

**What goes wrong:**
`5 - 5 = 0` or `0 + 0 = 0` is a valid puzzle under the constraints, but very young children
may not know that entering `0` is a valid answer. They may think the game is broken when
the answer is zero.

**Prevention:**
Optionally filter out zero-answer puzzles, or ensure at least the first 2–3 puzzles have
answers ≥ 1. Simple filter in `generatePuzzle()`:

```javascript
// Skip zero-answer puzzles (optional — reduces frustration for youngest players)
if (p.answer === 0) continue;
```

**Phase:** Puzzle generation (polish)

---

## Phase-Specific Warnings

| Phase Topic | Likely Pitfall | Mitigation |
|-------------|---------------|------------|
| Timer implementation | Stale interval accumulation (P1), tick drift (P4), race condition (P2) | Module-level `timerId`, wall-clock math, `gameActive` guard |
| Puzzle generation | Negative results (P3), duplicates (P9), zero answers (P15) | Constrained subtraction formula, dedup `Set` |
| Input handling | Double-submission on Enter (P6), `type="number"` allows `e`/`-` (P7) | Single event binding, `inputmode="numeric"` |
| Game state / Play Again | Stale state on reset (P5) | Single canonical `resetGame()` function |
| Mobile / UI | iOS zoom (P8), touch targets (P11), focus delay (P14) | `font-size: 16px`, 48px targets, synchronous `focus()` |
| Feedback / A11y | Color-only signals (P12) | Emoji + color always paired |
| Deployment | CDN cache confusion (P13) | Awareness; hard-reload to verify |
| Tab switching | Timer throttling on hidden tab (P10) | `visibilitychange` pause/resume (can defer to v2) |

---

## Sources

- MDN Web Docs — `setInterval` / `clearInterval` (HIGH confidence)
- MDN Web Docs — Page Visibility API (HIGH confidence)
- Apple WebKit Bug Tracker — programmatic `focus()` restrictions on iOS (HIGH confidence, widely documented)
- Google Material Design — Touch target guidelines, 48dp minimum (HIGH confidence)
- HTML Living Standard — `<input type="number">` permitted characters including `e`, `+`, `-` (HIGH confidence)
- Web Content Accessibility Guidelines (WCAG) 1.4.1 — Use of Color (HIGH confidence)
- GitHub Pages documentation — CDN caching behavior (MEDIUM confidence — behavior observed, not formally documented)
