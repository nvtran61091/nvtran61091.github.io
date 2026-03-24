# Phase 1: Foundation — Research

**Researched:** 2026-03-23
**Domain:** Single-file vanilla HTML/CSS/JS browser game — puzzle generation, static UI prototype, GitHub Pages deployment
**Confidence:** HIGH — stack is fully constrained by the project spec; all findings draw from well-established, stable web platform features

---

## Summary

Phase 1 delivers a browsable `index.html` static prototype with three things that must be true before any game logic is wired: (1) valid puzzle generation verifiable in the browser console, (2) all four game screens rendered with kid-friendly visual design, and (3) a confirmed GitHub Pages deployment path. Nothing in this phase requires a timer, interactivity, or state transitions — those are Phase 2 and 3 concerns.

The stack is fixed by project constraints: plain HTML5, vanilla JS (ES2020+), inline CSS, single `index.html`, GitHub Pages. No library, framework, or build tool is appropriate or needed. The architecture is a plain-object module system inside one `<script>` block, rendering all screens into a single `<div id="app">` root via `innerHTML` swap. Phase 1 builds only the lowest two layers of this stack — `CONFIG` and `PuzzleGenerator` (pure logic, no DOM) plus `UIRenderer` static screens (hard-coded data, no interactivity) — giving a visual prototype that can be deployed and inspected before any game logic exists.

The dominant Phase 1 risk is getting the **puzzle generation constraints wrong from the start** (negative subtraction results, missing dedup, missing result bounds) — these are silent failures with no runtime errors that will propagate into Phases 2–4 if not verified in the console before any UI wiring happens. The dominant **UI risk** is building touch targets and font sizes that look fine on desktop DevTools but fail on a real phone — enforce 44×44px minimums and `font-size ≥ 16px` on inputs from first commit, not as a retrofit.

**Primary recommendation:** Build `CONFIG` + `PuzzleGenerator` first, verify all constraints in the browser console with 1000-iteration stress tests, then build static screen HTML/CSS. Deploy to GitHub Pages as soon as the file is non-trivial to validate the deployment path early.

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| PUZZLE-01 | Each puzzle is a random addition or subtraction expression | `PuzzleGenerator.generate()` pattern — random operator selection, `Math.random() < 0.5 ? '+' : '-'` |
| PUZZLE-02 | All puzzle results are non-negative integers less than 20 | Constrained generation: for `+` ensure `a+b < 20`; for `-` ensure `b ≤ a`. Validated by console stress-test (1000 runs) |
| PUZZLE-03 | Subtraction operands constrained so result is never negative (`b ≤ a`) | Pick `a` first from `[0,19]`, then `b` from `[0,a]` — guarantees `a−b ≥ 0` always |
| PUZZLE-04 | 10 puzzles generated fresh each game session | `while (puzzles.length < 10)` loop with dedup `Set` — always terminates (valid space ~381 combos) |
| PUZZLE-05 | Puzzle operands use only non-negative integers | `Math.floor(Math.random() * N)` always ≥ 0; subtraction formula keeps both operands ≥ 0 |
| UI-02 | Large, readable fonts (≥ 16px base, puzzle text much larger) | Base: `clamp(18px, 4vw, 28px)`. Puzzle equation: `clamp(48px, 12vw, 96px)`. Input `font-size` never below 16px (iOS zoom threshold) |
| UI-03 | Touch targets ≥ 44×44px for numpad buttons | `min-height: 56px; min-width: 56px` on all `<button>` elements. Phase 1 shows static numpad layout — sizing established here |
| UI-06 | Layout works on both mobile and desktop | `min-height: 100dvh`, `width: min(480px, 100%)`, mobile-first at 375px viewport. Test at 375px and 1440px |
| DEPLOY-01 | Entire game in a single `index.html` | Already the project constraint. All CSS in `<style>`, all JS in `<script>`, no `<link>` or `<script src>` |
| DEPLOY-02 | No external dependencies | No CDN links, no npm, no external fonts. System font stack only |
| DEPLOY-03 | Game runs correctly when served from GitHub Pages | Push `index.html` to `main` branch → GitHub Pages auto-serves from repo root. Verify within 60 seconds |
</phase_requirements>

---

## Standard Stack

### Core
| Technology | Version | Purpose | Why Standard |
|------------|---------|---------|--------------|
| HTML5 | Living Standard | Document structure, app shell, semantic elements | Only markup option; no alternative in a single-file browser game |
| CSS3 (inline `<style>`) | Living Standard | All visual styling, responsive layout, animations | Inline keeps deployment single-file; no HTTP request for external sheet |
| Vanilla JavaScript (inline `<script>`) | ES2020+ | Puzzle logic, DOM rendering, state, event handling | No build step = no toolchain; ES2020 is 97%+ browser coverage as of 2025 |
| GitHub Pages | — | Static hosting | Free, zero-config, auto-serves `index.html` from repo root on push to `main` |

### Supporting CSS Techniques
| Feature | Browser Support | Purpose | When to Use |
|---------|----------------|---------|-------------|
| `clamp()` | Chrome 79+, Safari 13.1+, FF 75+ | Fluid typography without media queries | Every font-size declaration |
| `100dvh` | Chrome 108+, Safari 15.4+, FF 109+ | Correct mobile viewport height (accounts for browser chrome) | `min-height` on `body` |
| CSS Custom Properties | Chrome 49+, Safari 9.1+, FF 31+ | Design token theming (colors, radii) | `:root` variable declarations |
| `@keyframes` | All modern browsers | Smooth animations on compositor thread (no JS animation library) | Correct/wrong feedback, timer pulse |
| CSS Grid | All modern browsers | Numpad button layout (3×4 grid) | Static numpad layout in Phase 1 |
| `min()` / `max()` | Chrome 79+, Safari 11.1+, FF 75+ | Responsive widths without breakpoints | Card width: `min(480px, 100%)` |

### What NOT to Use
| Technology | Why Excluded |
|------------|-------------|
| React / Vue / Svelte | 30–100KB runtime for ~250 lines of logic. Builds required. Zero ROI. |
| Vite / Parcel / Webpack | Breaks "just push `index.html`" deployment model |
| External CSS (Tailwind, Bootstrap) | Requires build step (Tailwind) or CDN dependency (Bootstrap). Both violate DEPLOY-02 |
| Canvas / WebGL | Game is form-based. DOM + CSS is simpler and more accessible |
| `<input type="number">` | Allows `e`, `+`, `-` chars per HTML spec. Use `type="text" inputmode="numeric"` |
| External fonts (Google Fonts `@import`) | Network dependency; breaks offline. System font stack is sufficient |
| `anime.js` / GSAP | CSS keyframes cover all Phase 1–4 animation needs |

**Installation:** None — no packages. Edit `index.html` directly.

---

## Architecture Patterns

### Recommended File Structure
```
index.html              ← entire application (~350–450 lines total in Phase 4)
  ├── <meta viewport>   ← width=device-width, initial-scale=1 (NO maximum-scale)
  ├── <style>           ← all CSS: variables, layout, typography, animations (~100–130 lines)
  ├── <body>
  │   └── <div id="app"> ← single render root; UIRenderer owns everything here
  └── <script>          ← modules in dependency order (~200–280 lines)
      ├── CONFIG         { PUZZLE_COUNT, TIMER_SECONDS, MAX_RESULT, OPERATORS }
      ├── PuzzleGenerator { generate(count) → Puzzle[] }
      ├── GameState      { phase, puzzles[], currentIndex, score, timeRemaining, timerID }
      ├── TimerController { start(onTick, onExpire), stop() }          ← Phase 3
      ├── UIRenderer     { renderStart(), renderQuestion(), renderGameOver(), renderResults(), updateTimer() }
      └── GameController { init(), startGame(), showQuestion(), checkAnswer(), onTimeout(), playAgain() }
```

### Phase 1 Build Scope (subset of full architecture)
```
index.html
  ├── <style>           ← full design system CSS (all phases will use this)
  ├── <div id="app">   ← pre-populated with start screen HTML (static)
  └── <script>
      ├── CONFIG         ← implement fully (used by PuzzleGenerator)
      ├── PuzzleGenerator ← implement fully (verifiable in console)
      ├── GameState      ← stub (empty object, populated in Phase 2)
      ├── TimerController ← stub only (empty object)
      ├── UIRenderer     ← static render methods only (hard-coded test data)
      └── GameController ← init() only (calls UIRenderer.renderStart())
```

### Pattern 1: Constrained Puzzle Generation (PUZZLE-01 through PUZZLE-05)
**What:** Pure function producing `count` unique, constraint-valid puzzle objects with no side effects.
**When to use:** Called once per game session by `GameController.startGame()`.

```javascript
// Source: PITFALLS.md + ARCHITECTURE.md (project research)
const PuzzleGenerator = {
  generate(count = CONFIG.PUZZLE_COUNT) {
    const seen = new Set();
    const puzzles = [];

    while (puzzles.length < count) {
      const op = Math.random() < 0.5 ? '+' : '-';
      let a, b;

      if (op === '+') {
        // Ensure a + b < MAX_RESULT (< 20)
        a = Math.floor(Math.random() * (CONFIG.MAX_RESULT - 1));  // 0–18
        b = Math.floor(Math.random() * (CONFIG.MAX_RESULT - a));  // ensures a+b ≤ 19
      } else {
        // Ensure b ≤ a so result is never negative (PUZZLE-03)
        a = Math.floor(Math.random() * CONFIG.MAX_RESULT);        // 0–19
        b = Math.floor(Math.random() * (a + 1));                  // 0–a
      }

      const answer = op === '+' ? a + b : a - b;

      // Guard: skip zero-answer puzzles (young kids may not know 0 is valid)
      if (answer === 0) continue;

      // Dedup: reject if same expression already in this set (PUZZLE-04)
      const key = `${a}${op}${b}`;
      if (seen.has(key)) continue;

      seen.add(key);
      puzzles.push({ operandA: a, operator: op, operandB: b, answer });
    }

    return puzzles;
  }
};
```

**Console verification:**
```javascript
// Paste into DevTools console after page loads:
const p = PuzzleGenerator.generate(10);
console.table(p);
// Verify: all answers 1–19, no negatives, no duplicates
console.assert(p.every(x => x.answer > 0 && x.answer < 20), 'FAIL: answer out of range');
console.assert(p.every(x => x.operandA >= 0 && x.operandB >= 0), 'FAIL: negative operand');
const keys = p.map(x => `${x.operandA}${x.operator}${x.operandB}`);
console.assert(new Set(keys).size === 10, 'FAIL: duplicates found');
console.log('All 10 puzzles valid ✓');
```

### Pattern 2: CSS Design System (UI-02, UI-03, UI-06)
**What:** All visual constants defined as CSS Custom Properties at `:root`. Typography uses `clamp()`. Touch targets hardcoded at ≥56px. Layout uses `dvh` and `min()`.

```css
/* Source: STACK.md (project research), MDN Web Docs */
:root {
  /* Color palette — warm, high-saturation, kid-friendly */
  --color-bg:       #FFF9C4;   /* warm yellow — feels sunny */
  --color-primary:  #FF6B35;   /* vibrant orange — CTAs */
  --color-correct:  #4CAF50;   /* green — correct feedback */
  --color-wrong:    #F44336;   /* red — wrong/game over */
  --color-text:     #2C2C2C;   /* near-black — readable on light bg */
  --color-card:     #FFFFFF;   /* white card surface */
  --color-accent:   #7C4DFF;   /* purple — secondary actions */

  /* Typography */
  --font-base:    clamp(18px, 4vw, 24px);
  --font-large:   clamp(28px, 6vw, 40px);
  --font-puzzle:  clamp(48px, 12vw, 80px);   /* equation display */
  --font-family:  'Comic Sans MS', 'Chalkboard SE', 'Arial Rounded MT Bold', cursive;

  /* Spacing & shape */
  --radius-card:  24px;
  --radius-btn:   16px;
  --gap:          16px;
}

/* Layout — mobile-first, single viewport */
body {
  font-family: var(--font-family);
  font-size: var(--font-base);
  background: var(--color-bg);
  min-height: 100dvh;          /* dynamic viewport height — handles mobile browser chrome */
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: var(--gap);
  box-sizing: border-box;
  margin: 0;
  color: var(--color-text);
}

/* Card container — max 480px wide, full-width on small screens */
.game-card {
  width: min(480px, 100%);
  background: var(--color-card);
  border-radius: var(--radius-card);
  padding: 32px 24px;
  box-shadow: 0 8px 32px rgba(0,0,0,0.12);
  text-align: center;
}

/* Touch targets — MUST be ≥ 44×44px; using 56px for safety (UI-03) */
button {
  min-height: 56px;
  min-width: 56px;
  font-size: var(--font-base);
  font-family: inherit;
  border-radius: var(--radius-btn);
  cursor: pointer;
  border: none;
  padding: 12px 24px;
  font-weight: 700;
}

/* iOS viewport zoom prevention — MUST be ≥ 16px on ALL inputs */
input {
  font-size: 16px;   /* never below 16px — iOS Safari auto-zooms below this */
  font-family: inherit;
}

/* Numpad grid for on-screen number pad */
.numpad {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
  margin-top: 20px;
}

.numpad-btn {
  min-height: 64px;   /* larger target for ages 5–9 fine motor imprecision */
  font-size: var(--font-large);
  font-weight: 900;
  background: var(--color-bg);
  border: 3px solid var(--color-primary);
  border-radius: 14px;
}

/* Puzzle equation display */
.puzzle-equation {
  font-size: var(--font-puzzle);
  font-weight: 900;
  color: var(--color-text);
  letter-spacing: 2px;
  margin: 24px 0;
}
```

### Pattern 3: Four Static Screen Templates (UI-02, UI-06)
**What:** Hard-coded HTML strings for each of the four game screens. In Phase 1, these are rendered once via direct `innerHTML` assignment or `GameController.init()`. No interactivity wired yet.

```html
<!-- START SCREEN -->
<div class="game-card screen-start">
  <div class="emoji-big">🔢</div>
  <h1>Math Adventure!</h1>
  <p>Can you solve <strong>10</strong> puzzles?</p>
  <button class="btn-primary" onclick="GameController.startGame()">
    ▶ Play!
  </button>
</div>

<!-- QUESTION SCREEN (static mockup — hard-coded values) -->
<div class="game-card screen-question">
  <div class="progress-row">
    <span class="progress-label">Question 1 / 10</span>
    <span class="score-label">⭐ 0</span>
  </div>
  <div class="timer-bar-track">
    <div class="timer-bar-fill" style="width: 80%"></div>
  </div>
  <div class="puzzle-equation">7 + 5 = ?</div>
  <div class="answer-display">__</div>
  <div class="numpad">
    <button class="numpad-btn">1</button>
    <button class="numpad-btn">2</button>
    <button class="numpad-btn">3</button>
    <button class="numpad-btn">4</button>
    <button class="numpad-btn">5</button>
    <button class="numpad-btn">6</button>
    <button class="numpad-btn">7</button>
    <button class="numpad-btn">8</button>
    <button class="numpad-btn">9</button>
    <button class="numpad-btn numpad-clear">⌫</button>
    <button class="numpad-btn">0</button>
    <button class="numpad-btn numpad-submit">✅</button>
  </div>
</div>

<!-- GAME OVER SCREEN -->
<div class="game-card screen-gameover">
  <div class="emoji-big">😢</div>
  <h1>Game Over!</h1>
  <p>You solved <strong>3</strong> puzzles!</p>
  <button class="btn-primary" onclick="GameController.playAgain()">
    🔄 Try Again
  </button>
</div>

<!-- RESULTS SCREEN -->
<div class="game-card screen-results">
  <div class="emoji-big">🎉</div>
  <h1>You Win!</h1>
  <p>All <strong>10</strong> puzzles solved! 🏆</p>
  <div class="stars">⭐⭐⭐⭐⭐</div>
  <button class="btn-primary" onclick="GameController.playAgain()">
    🔄 Play Again
  </button>
</div>
```

### Pattern 4: GitHub Pages Deployment (DEPLOY-01, DEPLOY-02, DEPLOY-03)
**What:** Push `index.html` to `main` branch root → GitHub Pages auto-serves it at `https://{username}.github.io`.
**When to use:** Phase 1 deploys the static prototype to validate the path before any game logic is wired.

```
Deployment checklist:
1. Repo: nvtran61091.github.io (username.github.io naming = auto-deploys)
2. Branch: main (default for username.github.io repos)
3. File location: /index.html (repo root)
4. Settings: GitHub → Settings → Pages → Source: "Deploy from branch: main / (root)"
5. Verify: navigate to https://nvtran61091.github.io — expect game start screen
6. No Jekyll, no _config.yml, no GitHub Actions needed for single .html file
```

**Key fact:** For `username.github.io` repos (this repo), GitHub Pages is automatically enabled. No configuration required — push `index.html` to main and it goes live within 30–60 seconds.

### Anti-Patterns to Avoid

- **Generating puzzles with rejection sampling on results only:** `if (result >= 0 && result < 20) { keep }` without operator-specific constraints is correct but biased — addition puzzles cluster near small numbers, subtraction puzzles will never produce answers near 19. Use the constrained formula in Pattern 1 for uniform distribution.
- **Storing timerID in a local function variable:** The ID gets lost after the function returns and can never be cleared. Always module-level.
- **Building CSS without testing at 375px width first:** What looks right at 1440px often overflows at 375px. Write mobile first, scale up.
- **Using `maximum-scale=1` in viewport meta:** Disables user zoom — an accessibility violation. Never include this. Instead prevent iOS zoom with `font-size: 16px` on inputs.
- **Calling `UIRenderer.renderQuestion()` on every timer tick:** Would reset the child's typed answer every second. `updateTimer()` must be a separate method targeting only the timer element.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Responsive typography | Media queries with breakpoint font sizes | `clamp(min, preferred, max)` | Single declaration handles all viewport sizes; no breakpoint math |
| Mobile-safe viewport height | `100vh` | `100dvh` | `100vh` includes browser chrome on iOS/Android causing overflow; `dvh` is dynamic and correct |
| Puzzle display font | Custom font via external URL | System `Comic Sans MS` / `Arial Rounded MT Bold` / `cursive` stack | No network request; no FOUT; offline-safe |
| Touch-accessible buttons | JavaScript touch event handlers | CSS `min-height: 56px` + `padding` | Pure CSS tap targets are more reliable and require zero JS |
| Color animations | `requestAnimationFrame` JS loops | CSS `transition` + `@keyframes` | Compositor thread; smoother on low-end phones; zero JS overhead |

**Key insight:** For a single-file game of this scope, the browser platform itself is the library. Every problem in Phase 1 has a CSS or HTML native solution. Resist the impulse to add JavaScript where CSS suffices.

---

## Common Pitfalls

### Pitfall 1: Subtraction Producing Negative Results (CRITICAL — PUZZLE-02, PUZZLE-03)
**What goes wrong:** `const b = Math.random() * 20; const result = a - b` silently produces `-11 = ?`. No runtime error. Kids get age-inappropriate puzzles.
**Why it happens:** Operator-agnostic random generation without per-operator constraints.
**How to avoid:** Use the constrained formula: pick `a` first, then `b` from `[0, a]` for subtraction. Pick `a` from `[0,18]` then `b` from `[0, 19-a]` for addition.
**Warning signs:** Run console stress test — check for any `answer < 0` across 500 generated puzzles.

### Pitfall 2: Duplicate Puzzles (PUZZLE-04)
**What goes wrong:** Same expression appears twice in one game. Kids say "I already did that."
**Why it happens:** Random generation from a small space (~381 valid combos) has meaningful collision probability.
**How to avoid:** `Set` keyed by `"${a}${op}${b}"` — reject and regenerate on collision. The space is always large enough for 10 unique puzzles.
**Warning signs:** `console.log(new Set(puzzles.map(p => p.operandA+p.operator+p.operandB)).size)` — must be 10.

### Pitfall 3: Touch Targets Too Small (UI-03)
**What goes wrong:** Numpad buttons look fine on desktop DevTools emulation but are impossible to tap reliably on a real phone for ages 5–9.
**Why it happens:** DevTools device emulation doesn't replicate actual finger-tip contact area.
**How to avoid:** Hard-code `min-height: 56px; min-width: 56px` on all buttons from the first CSS commit. Verify with DevTools computed styles tab on real element.
**Warning signs:** Open on a real phone — do the numpad buttons feel comfortable to tap with a thumb?

### Pitfall 4: iOS Viewport Zoom (UI-02, UI-06)
**What goes wrong:** Any `<input>` with `font-size < 16px` causes iOS Safari to auto-zoom the viewport on focus, breaking the game layout.
**Why it happens:** iOS Safari accessibility feature that can't be suppressed without violating WCAG (via `maximum-scale=1`).
**How to avoid:** `input { font-size: 16px; }` — never below this. Control visual size with `padding` and `width`, not `font-size`.
**Warning signs:** Test on real iOS device or BrowserStack — does the page zoom in when tapping the answer input?

### Pitfall 5: `100vh` Overflow on Mobile (UI-06)
**What goes wrong:** `min-height: 100vh` includes the browser address bar height, causing content to be clipped or create a scroll area on phones.
**Why it happens:** `vh` is relative to the full viewport including browser chrome. `dvh` is dynamic (updates when chrome hides/shows).
**How to avoid:** Use `min-height: 100dvh` on `body`. Browser support is 2022+, covers all 2025 target browsers.
**Warning signs:** On Chrome for Android, scrolling down shows the full game only after browser chrome auto-hides.

### Pitfall 6: GitHub Pages Not Serving Updated File (DEPLOY-03)
**What goes wrong:** Push a fix, hard-reload, still see old content. Think the push failed.
**Why it happens:** GitHub Pages CDN caches `index.html`. Propagation takes 3–10 minutes.
**How to avoid:** This is expected behavior — not fixable without paid CDN config. Always test locally with `file://` first. Wait ~5 minutes after push before verifying.
**Warning signs:** Check GitHub Actions tab for "pages build and deployment" completion before testing live URL.

---

## Code Examples

Verified patterns from project research and official sources:

### Puzzle Generator (Full Implementation)
```javascript
// Source: PITFALLS.md + ARCHITECTURE.md (project research)
const CONFIG = {
  PUZZLE_COUNT: 10,
  TIMER_SECONDS: 15,
  MAX_RESULT: 20,        // exclusive: answers are 1–19
  OPERATORS: ['+', '-']
};

const PuzzleGenerator = {
  generate(count = CONFIG.PUZZLE_COUNT) {
    const seen = new Set();
    const puzzles = [];

    while (puzzles.length < count) {
      const op = Math.random() < 0.5 ? '+' : '-';
      let a, b;

      if (op === '+') {
        a = Math.floor(Math.random() * (CONFIG.MAX_RESULT - 1));    // 0–18
        b = Math.floor(Math.random() * (CONFIG.MAX_RESULT - a));    // 0–(19-a)
      } else {
        a = Math.floor(Math.random() * CONFIG.MAX_RESULT);          // 0–19
        b = Math.floor(Math.random() * (a + 1));                    // 0–a
      }

      const answer = op === '+' ? a + b : a - b;

      // Skip zero-answer puzzles (young kids may not recognize 0 as a valid answer)
      if (answer === 0) continue;

      const key = `${a}${op}${b}`;
      if (seen.has(key)) continue;

      seen.add(key);
      puzzles.push({ operandA: a, operator: op, operandB: b, answer });
    }

    return puzzles;
  }
};
```

### CSS Fluid Typography (UI-02)
```css
/* Source: MDN Web Docs — CSS clamp() */
/* Base text — scales from 18px on small phones to 24px on large screens */
body    { font-size: clamp(18px, 4vw, 24px); }

/* Labels and UI chrome */
.label  { font-size: clamp(14px, 3vw, 18px); }

/* Puzzle equation — must be visually dominant */
.puzzle { font-size: clamp(48px, 12vw, 80px); font-weight: 900; }

/* Answer display and input */
.answer { font-size: clamp(32px, 8vw, 56px); }

/* Headings on start/gameover/results screens */
h1      { font-size: clamp(28px, 7vw, 48px); }
```

### Viewport Meta (UI-06)
```html
<!-- Source: MDN Web Docs — Viewport meta tag -->
<!-- NEVER add maximum-scale=1 — that disables user zoom (WCAG violation) -->
<meta name="viewport" content="width=device-width, initial-scale=1">
```

### Mobile-Safe Layout (UI-06)
```css
/* Source: MDN Web Docs — dvh, min() */
body {
  min-height: 100dvh;      /* dvh: dynamic viewport height — correct on mobile */
  overflow-x: hidden;      /* prevent horizontal scroll */
}

.game-card {
  width: min(480px, 100%); /* full-width on small screens, max 480px on desktop */
  box-sizing: border-box;  /* padding doesn't cause overflow */
}
```

### Static Screen Switcher (for Phase 1 testing)
```javascript
// Phase 1: Manual screen switcher for visual QA without full GameController
// Paste into DevTools console or add a temporary <select> to index.html

function previewScreen(name) {
  const screens = {
    start: UIRenderer.renderStart,
    question: () => UIRenderer.renderQuestion(
      PuzzleGenerator.generate(1)[0], 1, 15
    ),
    gameover: () => UIRenderer.renderGameOver(3),
    results:  () => UIRenderer.renderResults(10)
  };
  screens[name]?.call(UIRenderer);
}

// Usage:
// previewScreen('start')
// previewScreen('question')
// previewScreen('gameover')
// previewScreen('results')
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `100vh` for full-height layouts | `100dvh` | 2022 (iOS 15.4, Chrome 108) | Eliminates mobile overflow caused by browser toolbar |
| `calc(16px + 1vw)` for fluid type | `clamp(min, preferred, max)` | 2020 (broad support) | Single declaration; easier reasoning; no magic numbers |
| `type="number"` for numeric input | `type="text" inputmode="numeric" pattern="[0-9]*"` | Recognized pattern, ~2018 | Prevents `e`, `+`, `-` in input; consistent mobile keyboard |
| `@media (max-width: 480px)` breakpoints | Mobile-first + `min()` / `clamp()` | ~2021 | Fewer rules; no breakpoint calculations |
| Separate CSS files | Inline `<style>` (for single-file games) | Always valid; reinforced by offline/PWA patterns | Zero HTTP requests; offline-safe; single file to push |

**Deprecated/outdated for this project:**
- `100vh` on body: Use `100dvh` — `100vh` clips content on iOS/Android Safari
- `<input type="number">`: Use `type="text" inputmode="numeric"` — `type="number"` allows non-digit chars
- `maximum-scale=1` in viewport meta: Never use — disables user zoom, WCAG violation
- Comic Sans embedded via `@font-face`: Not needed — `Comic Sans MS` is in the Windows/Mac system font stack; `Chalkboard SE` is the macOS/iOS equivalent; `cursive` generic covers Android

---

## Open Questions

1. **Allow zero-answer puzzles?**
   - What we know: `5 − 5 = 0` and `0 + 0 = 0` are mathematically valid under the spec constraints
   - What's unclear: Whether ages 5–7 understand entering "0" as a valid answer, or whether they'll think the game is broken
   - Recommendation: **Filter zero-answer puzzles in Phase 1** (`if (answer === 0) continue`). It costs nothing, eliminates a confusion vector for the youngest players, and can be removed in v2 if desired. Document the decision in `CONFIG` as a comment.

2. **Screen switching mechanism for Phase 1 static prototype**
   - What we know: Phase 1 shows all 4 screens but has no game logic wired
   - What's unclear: Should the start screen be the default, with a dev-only dropdown to preview other screens? Or should all 4 screens be in the HTML as stacked divs?
   - Recommendation: **Default to start screen via `GameController.init()`**. Add a commented-out `previewScreen()` helper in the script for QA. Avoid stacked divs — they're harder to remove later.

3. **Timer bar CSS structure**
   - What we know: Timer bar needs to shrink (width %), change color, and animate in Phase 3
   - What's unclear: Should the timer bar HTML element exist in Phase 1's question screen mockup, or be stubbed with a placeholder?
   - Recommendation: **Build the full timer bar HTML + CSS in Phase 1** (with hard-coded `width: 100%`), even though the logic isn't wired. This establishes the CSS class names and element structure that Phase 3 will target, preventing HTML refactor later.

---

## Validation Architecture

> `nyquist_validation` is `true` in `.planning/config.json` — this section is required.

### Test Framework
| Property | Value |
|----------|-------|
| Framework | None — browser DevTools console only (no test runner; single-file vanilla JS constraint) |
| Config file | None |
| Quick run command | Open `index.html` in browser → paste console assertions |
| Full suite command | Open `index.html` → run full validation script below |

**Rationale:** This project has no build step, no Node.js, no `package.json`. Installing Jest/Vitest/Mocha would require a build toolchain that violates DEPLOY-01 and DEPLOY-02. All validation is in-browser console assertions — executable without any external tooling.

### Phase Requirements → Validation Map

| Req ID | Behavior to Verify | Validation Type | Command / Method |
|--------|-------------------|-----------------|-----------------|
| PUZZLE-01 | Puzzles are `+` and `-` only | Console assertion | `console.assert(puzzles.every(p => ['+','-'].includes(p.operator)))` |
| PUZZLE-02 | All answers are integers 1–19 | Console assertion | `console.assert(puzzles.every(p => Number.isInteger(p.answer) && p.answer > 0 && p.answer < 20))` |
| PUZZLE-03 | No negative answers from subtraction | Console stress test | 1000-iteration loop, check for any `answer < 0` |
| PUZZLE-04 | 10 fresh puzzles generated | Console assertion | `console.assert(PuzzleGenerator.generate(10).length === 10)` |
| PUZZLE-05 | All operands ≥ 0 | Console assertion | `console.assert(puzzles.every(p => p.operandA >= 0 && p.operandB >= 0))` |
| UI-02 | Base font ≥ 16px, puzzle ≥ 48px | DevTools Computed Styles | Inspect `.puzzle-equation` → computed `font-size`; inspect `body` → computed `font-size` |
| UI-03 | Touch targets ≥ 44×44px | DevTools Element Inspector | Inspect `.numpad-btn` → computed `height` and `width` ≥ 44px |
| UI-06 | Layout works on mobile + desktop | DevTools Responsive Mode | Toggle 375×667 (iPhone SE) ↔ 1440×900; no horizontal scroll, no clipped content |
| DEPLOY-01 | Single file, no external deps | File inspection | `grep -c "<link\|<script src" index.html` → must be 0 |
| DEPLOY-02 | No CDN links | File inspection | `grep "http" index.html` → must be 0 (no external URLs in `src` or `href`) |
| DEPLOY-03 | GitHub Pages serves correctly | Live URL check | Navigate to `https://nvtran61091.github.io` — game start screen loads, no 404, no console errors |

### Full Console Validation Script
Paste this entire block into DevTools console after `index.html` loads:

```javascript
// Phase 1 Full Validation Suite
// Run in browser DevTools console after loading index.html
(function validatePhase1() {
  console.group('🧪 Phase 1 Validation');

  // --- PUZZLE-01: Operators are + and - only ---
  const puzzles = PuzzleGenerator.generate(10);
  const validOps = puzzles.every(p => ['+', '-'].includes(p.operator));
  console.assert(validOps, '❌ PUZZLE-01: Invalid operator found');
  console.log(validOps ? '✅ PUZZLE-01: Operators valid' : '❌ PUZZLE-01 FAIL');

  // --- PUZZLE-02: Answers in range [1, 19] ---
  const validAnswers = puzzles.every(p =>
    Number.isInteger(p.answer) && p.answer >= 1 && p.answer < 20
  );
  console.assert(validAnswers, '❌ PUZZLE-02: Answer out of range');
  console.log(validAnswers ? '✅ PUZZLE-02: Answers in range' : '❌ PUZZLE-02 FAIL');

  // --- PUZZLE-03: No negative answers (stress test) ---
  let negFound = false;
  for (let i = 0; i < 1000; i++) {
    const batch = PuzzleGenerator.generate(10);
    if (batch.some(p => p.answer < 0)) { negFound = true; break; }
  }
  console.assert(!negFound, '❌ PUZZLE-03: Negative answer found in 1000-run stress test');
  console.log(!negFound ? '✅ PUZZLE-03: No negative answers (1000 runs)' : '❌ PUZZLE-03 FAIL');

  // --- PUZZLE-04: Always produces exactly 10 puzzles ---
  const count10 = PuzzleGenerator.generate(10).length === 10;
  console.assert(count10, '❌ PUZZLE-04: Did not produce 10 puzzles');
  console.log(count10 ? '✅ PUZZLE-04: 10 puzzles generated' : '❌ PUZZLE-04 FAIL');

  // --- PUZZLE-05: All operands non-negative ---
  const validOperands = puzzles.every(p => p.operandA >= 0 && p.operandB >= 0);
  console.assert(validOperands, '❌ PUZZLE-05: Negative operand found');
  console.log(validOperands ? '✅ PUZZLE-05: All operands non-negative' : '❌ PUZZLE-05 FAIL');

  // --- No duplicates check ---
  const keys = puzzles.map(p => `${p.operandA}${p.operator}${p.operandB}`);
  const noDupes = new Set(keys).size === 10;
  console.assert(noDupes, '❌ Duplicate puzzles found');
  console.log(noDupes ? '✅ No duplicate puzzles' : '❌ DUPLICATE FAIL');

  // --- Display all puzzles ---
  console.table(puzzles.map(p => ({
    puzzle: `${p.operandA} ${p.operator} ${p.operandB} = ?`,
    answer: p.answer,
    valid: p.answer >= 1 && p.answer < 20 ? '✅' : '❌'
  })));

  console.groupEnd();
})();
```

### DevTools Verification Procedures

**Verify touch target sizes (UI-03):**
1. Open DevTools → Elements tab
2. Inspect any `.numpad-btn`
3. Computed tab → check `height` and `width`
4. **Pass:** both ≥ 44px (should be 64px per implementation)
5. Alternatively: Elements → right-click → Inspect → hover element in viewport → tooltip shows dimensions

**Verify font sizes (UI-02):**
1. DevTools → Elements → inspect `.puzzle-equation`
2. Computed tab → `font-size` must be ≥ 48px at 375px viewport
3. Inspect `body` → computed `font-size` must be ≥ 16px
4. Inspect any `input` → computed `font-size` must be exactly 16px (iOS zoom threshold)

**Verify mobile layout (UI-06):**
1. DevTools → Toggle Device Toolbar (Ctrl+Shift+M / Cmd+Shift+M)
2. Set dimensions to **375 × 667** (iPhone SE) — smallest common target
3. Verify: no horizontal scrollbar, no clipped content, all 4 screens readable
4. Set dimensions to **1440 × 900** (desktop) — verify card width caps at ~480px, centered
5. Set dimensions to **414 × 896** (iPhone XR) — verify no layout breaks

**Verify no external dependencies (DEPLOY-01, DEPLOY-02):**
1. DevTools → Network tab → reload page
2. Verify: only ONE request — `index.html` (or the GitHub Pages URL)
3. No requests for `.css`, `.js`, fonts, or CDN resources
4. Alternatively, use Sources tab — only `index.html` should appear in the file tree

**Verify GitHub Pages deployment (DEPLOY-03):**
1. Push `index.html` to `main` branch on `nvtran61091.github.io`
2. GitHub → Actions tab (or Settings → Pages) → wait for "pages build and deployment" to show ✅
3. Navigate to `https://nvtran61091.github.io`
4. DevTools → Console: zero errors
5. DevTools → Network: only `index.html` loaded (no 404s)
6. Hard-reload (Ctrl+Shift+R) if cached version appears

### Wave 0 Gaps
- None — Phase 1 requires no test framework installation. All validation is in-browser console assertions that run against the same `index.html` being built. No Wave 0 test file creation needed.

---

## Sources

### Primary (HIGH confidence)
- **Project STACK.md** — CSS techniques, browser compatibility table, touch target sizing, typography patterns (project research, 2026-03-23)
- **Project ARCHITECTURE.md** — module structure, `PuzzleGenerator` API shape, `UIRenderer` render methods, build order (project research, 2025-01-30)
- **Project PITFALLS.md** — subtraction constraint formula, dedup `Set` pattern, `font-size: 16px` iOS threshold, stale interval pattern (project research, 2025-01-30)
- **Project FEATURES.md** — touch target minimums, typography sizing, anti-features (project research, 2025-01-31)
- **Project SUMMARY.md** — phase 1 scope, architecture rules, open questions (project research, 2026-03-23)
- **MDN Web Docs** — `clamp()`, `dvh`, CSS Custom Properties, `inputmode`, `setInterval`/`clearInterval` (official spec documentation — HIGH)
- **WCAG 2.5.5** — minimum touch target 44×44px (official W3C standard — HIGH)
- **Apple Human Interface Guidelines** — minimum tap target 44pt, iOS zoom trigger at `font-size < 16px` (official Apple docs — HIGH)

### Secondary (MEDIUM confidence)
- GitHub Pages documentation — auto-deploy from `main` for `username.github.io` repos; CDN caching behavior (verified behavior, not formally documented latency numbers)
- Can I Use (caniuse.com) — `dvh`, `clamp()`, CSS Grid, ES2020 browser support (authoritative compatibility data, verified 2025)

### Tertiary (LOW confidence — none for Phase 1)
- All Phase 1 findings are HIGH or MEDIUM confidence based on verified official sources.

---

## Metadata

**Confidence breakdown:**
- Standard stack: **HIGH** — Fully constrained by project spec; confirmed by MDN, WCAG, Apple HIG, caniuse
- Architecture: **HIGH** — Vanilla JS module-object pattern is established; no external dependencies to validate; derived from project's own ARCHITECTURE.md
- Puzzle generation: **HIGH** — Pure arithmetic constraints; verified by runnable stress tests; no external library involved
- UI patterns (CSS): **HIGH** — Stable CSS features with 2025 universal browser support; sizes from WCAG + Apple HIG standards
- GitHub Pages deployment: **HIGH** — Auto-deploy from `username.github.io` repo is documented GitHub behavior; CDN latency is MEDIUM (observed, not spec'd)
- Pitfalls: **HIGH** — All are well-documented JavaScript / mobile browser failure modes with preventable, verified solutions

**Research date:** 2026-03-23
**Valid until:** 2026-06-23 (stable — CSS/HTML/GitHub Pages specs change slowly; browser compatibility data valid ~3 months)
