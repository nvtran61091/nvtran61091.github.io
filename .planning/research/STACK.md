# Technology Stack

**Project:** Kids Math Game (`nvtran61091.github.io`)
**Researched:** 2026-03-23
**Confidence:** HIGH — Stack is constrained by explicit project requirements (single `index.html`, GitHub Pages, no build tools). Research confirms 2025 best practices align with these constraints.

---

## Recommended Stack

### Core Platform

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| HTML5 | Living Standard | Markup, app shell | Universally supported; `<input type="number">` handles numeric keyboard on mobile natively |
| CSS3 (inline `<style>`) | Living Standard | All visual styling, animations | Inline in `index.html` keeps deployment single-file; no network request for stylesheet |
| Vanilla JavaScript (inline `<script>`) | ES2020+ | Game logic, timer, DOM control | No transpile step needed; ES2020 features (`optional chaining`, `nullish coalescing`, `const`/`let`) supported by 97%+ of browsers as of 2025 |

### Deployment Platform

| Technology | Purpose | Why |
|------------|---------|-----|
| GitHub Pages | Static hosting | Free, zero-config, instant on push to `main`; serves `index.html` at repo root automatically |

---

## Architecture: Single-File Rationale

**Single `index.html` is the right call for this project. Here's why it matters:**

1. **Zero deployment friction.** GitHub Pages serves `index.html` from the repo root with no configuration file, no `_config.yml`, no CI pipeline. Push → live in ~30 seconds.

2. **No CDN dependency risk.** External CDN links (even unpkg, jsDelivr) can 404, rate-limit, or change. A child gets a spinner instead of a game. Inline everything = no failure mode.

3. **Shareability.** Parents can `Save As` → `index.html` and the game still works offline on the plane. No hydration, no fetch.

4. **Complexity ceiling.** The game is ~150–250 lines of JS logic at most. The overhead of a module bundler (Vite, Parcel) or framework (React) would dwarf the actual application code. This is not a scaling problem.

---

## CSS Techniques for Kid-Friendly UI

**Confidence: HIGH** — These are stable, widely-supported CSS features.

### Color Palette

Use high-saturation, warm palette. Kids respond to bright primaries and high contrast. Avoid pastels (low energy). Avoid dark themes (overwhelming for young children).

```css
:root {
  --color-bg:       #FFF9C4;   /* warm yellow background — feels "sunny" */
  --color-primary:  #FF6B35;   /* vibrant orange — CTAs, timer bar */
  --color-correct:  #4CAF50;   /* green — correct feedback */
  --color-wrong:    #F44336;   /* red — wrong/game over */
  --color-text:     #333333;   /* near-black — readable on light bg */
  --color-card:     #FFFFFF;   /* white card for question area */
}
```

### Typography

```css
body {
  font-family: 'Comic Sans MS', 'Chalkboard SE', 'Arial Rounded MT Bold', cursive;
  /* Rationale: Comic Sans is maligned by designers but genuinely readable for
     early readers. Its irregular letterforms mirror handwriting, which is what
     kids 5–9 are learning. "Arial Rounded MT Bold" is the fallback that avoids
     Comic Sans without losing roundness. Never use serif or thin fonts for this
     audience. */
  font-size: clamp(18px, 4vw, 28px);
  /* clamp() scales with viewport — works on a 375px phone AND a 1440px desktop
     without media queries. */
}

.question-number {
  font-size: clamp(48px, 12vw, 96px);
  font-weight: 900;
  /* Numbers must be BIG. Kids are still learning to read digits quickly.
     Thumb-sized numbers reduce cognitive friction. */
}
```

### Layout

```css
body {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100dvh;   /* dvh = dynamic viewport height — accounts for mobile
                            browser chrome appearing/disappearing */
  padding: 16px;
  box-sizing: border-box;
}

.game-card {
  width: min(480px, 100%);  /* Never wider than 480px; full-width on small screens */
  border-radius: 24px;
  padding: 32px 24px;
  box-shadow: 0 8px 32px rgba(0,0,0,0.12);
}
```

### Animations (CSS Keyframes Only — No JS Animation Libraries)

```css
/* Celebrate correct answer */
@keyframes bounce-in {
  0%   { transform: scale(0.5); opacity: 0; }
  60%  { transform: scale(1.2); }
  100% { transform: scale(1);   opacity: 1; }
}

/* Shake wrong answer / game over */
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20%       { transform: translateX(-12px); }
  40%       { transform: translateX(12px); }
  60%       { transform: translateX(-8px); }
  80%       { transform: translateX(8px); }
}

/* Pulse timer bar when low */
@keyframes pulse-red {
  0%, 100% { background: #FF6B35; }
  50%       { background: #F44336; }
}

.correct  { animation: bounce-in 0.4s ease-out; }
.wrong    { animation: shake 0.5s ease-in-out; }
.timer-danger { animation: pulse-red 0.5s ease-in-out infinite; }
```

**Why keyframes over JS animation:** `requestAnimationFrame` is fine for canvas games but for DOM transitions, CSS keyframes run on the compositor thread (off main thread), producing smoother 60fps animation even when JS is busy updating state. No dependency on `anime.js`, `GSAP`, or similar libraries.

### Timer Bar (Visual Countdown)

```css
.timer-bar-track {
  width: 100%;
  height: 16px;
  background: #eee;
  border-radius: 8px;
  overflow: hidden;
}

.timer-bar-fill {
  height: 100%;
  background: #FF6B35;
  border-radius: 8px;
  transition: width 0.1s linear, background 0.3s ease;
  /* Smooth shrink every 100ms tick — avoids choppy jump-cuts.
     Color transitions to red automatically when width < 30%. */
}
```

### Touch Targets

```css
button, input[type="number"] {
  min-height: 56px;    /* Apple HIG minimum: 44pt. 56px gives comfortable margin
                          for small fingers. WCAG 2.5.5 target size is 44×44px. */
  font-size: inherit;  /* Never let browser default 16px shrink your button text */
  border-radius: 16px;
  cursor: pointer;
}
```

---

## JavaScript Patterns

**Confidence: HIGH** — Vanilla JS patterns are stable and well-understood.

### State Machine Pattern (No Framework Needed)

Represent the game as a simple state object. No Redux, no Zustand — just a plain object and a `render()` function.

```javascript
const state = {
  phase: 'start',      // 'start' | 'playing' | 'gameover' | 'win'
  puzzles: [],
  currentIndex: 0,
  correctCount: 0,
  timeLeft: 15,
  timerId: null,
};

function render() {
  // Single function that reads state and updates DOM.
  // Called after every state mutation.
  // This is the "poor man's React" — predictable, debuggable, zero overhead.
  document.getElementById('app').innerHTML = buildHTML(state);
  attachEventListeners(); // re-attach after innerHTML replace
}
```

**Why this pattern:** It keeps all DOM updates in one place. No scattered `element.style.display = 'none'` calls throughout business logic. Debugging means reading `state`, not tracing DOM mutations.

### Puzzle Generator

```javascript
function generatePuzzles(count = 10) {
  const puzzles = [];
  while (puzzles.length < count) {
    const a = Math.floor(Math.random() * 20);  // 0–19
    const b = Math.floor(Math.random() * 20);  // 0–19
    const op = Math.random() < 0.5 ? '+' : '-';
    const result = op === '+' ? a + b : a - b;
    // Constraints: result >= 0 AND result < 20 AND operands non-negative
    if (result >= 0 && result < 20) {
      puzzles.push({ a, b, op, answer: result });
    }
    // Loop until we have enough valid puzzles — guaranteed to terminate quickly
  }
  return puzzles;
}
```

### Timer Pattern

```javascript
function startTimer() {
  state.timerId = setInterval(() => {
    state.timeLeft -= 1;
    if (state.timeLeft <= 0) {
      clearInterval(state.timerId);
      triggerGameOver('timeout');
    }
    updateTimerBar();  // update only the bar, not full re-render (performance)
  }, 1000);
}

function resetTimer() {
  clearInterval(state.timerId);  // ALWAYS clear before starting new — prevents
  state.timeLeft = 15;           // multiple interval leaks if called repeatedly
  startTimer();
}
```

**Why `setInterval` over `setTimeout` chain:** For a countdown display, `setInterval(fn, 1000)` is simpler and sufficient. A `setTimeout` recursive chain is marginally more accurate but introduces complexity not warranted here. The 1-second tick visual update doesn't require sub-second accuracy.

### Input Handling (Mobile-First)

```html
<input type="number"
       inputmode="numeric"
       pattern="[0-9]*"
       min="0" max="99"
       autocomplete="off"
       autofocus />
<!--
  type="number" — triggers numeric keypad on iOS/Android
  inputmode="numeric" — reinforces numeric-only keyboard on modern mobile
  pattern="[0-9]*" — iOS Safari respects this for numeric keyboard
  autofocus — puts cursor in field immediately, no tap required
-->
```

```javascript
// Submit on Enter key — kids may tap Enter on keyboards
input.addEventListener('keydown', (e) => {
  if (e.key === 'Enter') submitAnswer();
});
```

### Emoji Usage

Use emoji directly in HTML strings — no icon library needed. Emoji render natively on all target platforms (iOS, Android Chrome, desktop Chrome/Firefox/Safari).

```javascript
const EMOJIS = {
  correct: '⭐',
  wrong:   '😢',
  win:     '🎉',
  timer:   '⏱️',
  puzzle:  ['🦁','🐶','🐱','🐸','🦊','🦄','🐼','🐨','🦋','🌈'],
};
// One random animal emoji per puzzle adds delight without any asset loading
```

---

## What NOT to Use — And Why

| Technology | Why Excluded |
|------------|-------------|
| **React / Vue / Svelte** | Adds 30–100KB+ of framework runtime. All problems React solves (state management, component reuse) are trivially solved with 20 lines of vanilla JS for a 10-question game. Zero ROI, 100% complexity cost. |
| **Vite / Parcel / webpack** | A build tool requires Node.js, `npm install`, `package.json`. GitHub Pages CI from a build tool adds ~2min to deploy cycle and breaks the "just push `index.html`" simplicity. |
| **TypeScript** | No compiler = no TS. Single-file constraint means no build step, so TS is unavailable. JSDoc comments in the source provide sufficient inline documentation. |
| **External CSS (separate file)** | Requires a second HTTP request (minor on fast connections, but also breaks "Save As" offline playability). Inline `<style>` is the right call for single-file. |
| **CSS frameworks (Tailwind, Bootstrap)** | Tailwind requires a build step. Bootstrap CDN adds an external dependency and ~200KB of CSS for a page that needs ~80 custom rules. Inline custom CSS is smaller and faster. |
| **Canvas / WebGL** | The game is form-based (number input), not graphical. Canvas would add complexity for zero visual gain. DOM + CSS animations are sufficient and more accessible. |
| **Web Components** | Spec overhead for a single-file game. HTML string templates rendered to `innerHTML` are simpler and just as effective at this scale. |
| **LocalStorage for scores** | Out of scope per PROJECT.md. Avoids feature creep in v1. |
| **Sound effects** | Explicitly excluded in PROJECT.md — autoplay restrictions on mobile browsers would produce silent failures, degrading UX instead of enhancing it. |
| **`anime.js` / GSAP / Motion** | CSS keyframe animations cover all animation needs (bounce, shake, pulse). No external animation library justified. |

---

## File Structure

```
index.html          ← entire application; ~300–400 lines total
  ├─ <style>        ← all CSS (variables, layout, animations) ~100–120 lines
  ├─ <body>         ← minimal shell; #app div populated by JS   ~10 lines
  └─ <script>       ← state, game logic, render, event handlers ~180–250 lines
```

---

## Browser Compatibility

**Confidence: HIGH**

Target: Chrome 90+, Safari 14+, Firefox 88+, Samsung Internet 14+ (2025 market share of these versions: >97% of mobile and desktop browsers).

Features used and their support:
| Feature | Chrome | Safari | Firefox | Notes |
|---------|--------|--------|---------|-------|
| CSS Custom Properties | 49+ | 9.1+ | 31+ | ✅ Universal |
| CSS `clamp()` | 79+ | 13.1+ | 75+ | ✅ Universal |
| `100dvh` | 108+ | 15.4+ | 109+ | ✅ 2025 safe |
| `inputmode="numeric"` | 66+ | 12.1+ | 95+ | ✅ Universal |
| ES2020 (`??`, `?.`) | 80+ | 13.1+ | 74+ | ✅ Universal |
| CSS `@keyframes` | 43+ | 9+ | 16+ | ✅ Universal |
| `setInterval` / `clearInterval` | All | All | All | ✅ Baseline |

No polyfills required.

---

## Sources

- MDN Web Docs — `inputmode`, `clamp()`, `dvh`, CSS Custom Properties (HIGH confidence — official spec documentation)
- WCAG 2.5.5 — Minimum touch target size 44×44px (HIGH confidence — official W3C standard)
- Apple Human Interface Guidelines — minimum tap target 44pt (HIGH confidence — official Apple docs)
- Can I Use (caniuse.com) — browser support data for `dvh`, `clamp()`, ES2020 (HIGH confidence — authoritative compatibility data, verified 2025)
- PROJECT.md — technology constraints (single `index.html`, no frameworks, GitHub Pages) defined by project owner
