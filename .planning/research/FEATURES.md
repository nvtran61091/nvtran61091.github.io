# Feature Landscape — Kids Browser Math Game

**Domain:** Browser-based educational math game for ages 5–9
**Researched:** 2025-01-31
**Confidence:** MEDIUM — drawn from established UX patterns for children's educational software, mobile game design principles, and the project spec constraints. No Brave/web search available; based on domain expertise.

---

## Context

This is a **no-backend, single-file** game on GitHub Pages. Target user is a 5–9 year old, typically on a parent's phone or tablet. The constraint "wrong answer = game over" defines the core game loop tension. Every feature decision must pass this filter:

> *"Would a 6-year-old understand this in 3 seconds with no instructions?"*

---

## Table Stakes

Features where absence causes immediate disengagement or confusion. If any of these are missing, kids won't play more than once.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| **Large, readable question text** | Kids 5–7 struggle with small text; squinting kills immersion instantly | Low | Min 48–64px font for equation display; use bold, high-contrast |
| **Visible countdown timer** | Kids need to feel the time pressure to stay engaged; a number alone is hard for young children to internalize urgency | Low–Med | A depleting progress bar (not just digits) is far more readable. Color shift green→yellow→red as time runs low |
| **Immediate correct/wrong visual feedback** | Children expect instant acknowledgement of every action; delay = confusion | Low | Flash green ✅ for correct, red ❌ for wrong before advancing or ending. Must happen < 300ms |
| **Progress indicator ("Question X of 10")** | Without it, kids don't know how close they are to winning — leads to quitting mid-game | Low | Simple "3 / 10" or star dots work. Helps older kids (7–9) calibrate effort |
| **Score screen at game end** | Defines the "reward" moment the whole session builds toward | Low | Show stars (⭐⭐⭐) or big number. Must be celebratory, not clinical |
| **Play Again button** | Re-play is how kids build mastery; if missing, session ends at loss with no recovery path | Low | Must be prominent, single tap. No confirmation dialog |
| **Game over screen (loss path)** | Without a clear "game over" state, kids think the app broke. Needs empathy, not punishment | Low | Show score so far, not just "you lost". Encourage re-try |
| **Kid-friendly visual language (emojis, colors)** | Clinical UI (white background, plain text) reads as "homework" not "game" | Low | Bright colors, large emojis (🎉 ⭐ 💪 ⏰ 😢) throughout. This is spec-required |
| **Touch-friendly answer input** | 60–70% of target users are on phones/tablets. Keyboard-reliant input fails on mobile for young kids who can't type reliably | Med | **Number pad buttons (0–9 + Submit)** instead of free-form text input. Large tap targets (min 56×56px). See note below |
| **Non-negative, age-appropriate math** | Results < 20, no borrowing confusion, no negatives. Violating this destroys confidence | Low | Already in spec. A result like "3 − 7" confusing kids is a trust-killer |
| **Instant game start (no tutorial, no login)** | Kids have zero tolerance for setup screens. First puzzle must appear within 1 tap | Low | Home/start screen → immediate first question. No "how to play" modal required if UI is self-explanatory |

### Critical Note: Answer Input Mechanism

**Use on-screen number buttons (0–9) + a Submit/Check button, NOT a free-form text `<input>` field.**

Rationale:
- Ages 5–7 often cannot type reliably on a phone keyboard
- Mobile virtual keyboards are unpredictable (autocorrect, wrong keyboard type, covering UI)
- Large tappable buttons guarantee consistent touch experience across iOS/Android
- Answers are always 1–2 digits (max 19), so a small numpad is sufficient
- Complexity: Medium (managing display + backspace logic) but essential for the platform

---

## Differentiators

Features that go beyond baseline expectations. Kids won't miss them if absent, but they create delight, re-play motivation, and a polished feel.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| **Celebration micro-animation on correct answer** | Emoji burst, bounce, or brief confetti flash rewards the child emotionally | Low | CSS keyframe animation. A "🎉 bouncing" for 600ms before next question. No library needed |
| **Encouragement messages between questions** | "Awesome! 🌟" / "Keep going! 💪" / "You're on fire! 🔥" makes the game feel alive | Low | Pick randomly from 4–5 phrases. Shown briefly on correct answer screen |
| **Timer color shift (visual urgency)** | Bar/number turning red in the last 5 seconds creates exciting pressure without sound | Low | Pure CSS transition: green (15–6s) → yellow (5–3s) → red (2–0s). Pairs with slight shake animation |
| **Streak indicator** | "3 in a row! 🔥" motivates the 7–9 age group to keep focus | Low | Track consecutive correct answers. Reset on game over. No storage needed |
| **Star rating on score screen** | "You got ⭐⭐⭐⭐ (8/10)!" is more emotionally rewarding than "Score: 80%" | Low | 5-star or 10-star display. Map score to stars with thresholds |
| **Personal best (localStorage)** | "New high score! 🏆" creates re-play motivation without any backend | Low | `localStorage.getItem('bestScore')`. Simple compare + update. ~5 lines of code |
| **Smooth question transitions** | Fade or slide-in between questions feels polished vs. instant snap | Low | CSS opacity/transform transition. ~3 lines CSS, 1 JS class toggle |
| **Timer "pulse" animation at low time** | Progress bar or number pulses/shakes when < 5 seconds left — visceral urgency without sound | Low | CSS `@keyframes shake` on the timer element when `timeLeft <= 5` |
| **Themed emoji per question** | Each question has a random contextual emoji (🍕 + 🍕 = ?) that makes the math visual | Med | Requires mapping operands to emoji counts. More suited to ages 5–6. Optional if pure number format is cleaner |

---

## Anti-Features

Things to deliberately **not** build. Each has a clear reason — avoiding scope creep and preserving the project's core strength (zero-friction simplicity).

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| **Sound effects** | Autoplay browser restrictions make sounds unreliable; parents hate unexpected noise; adds no correctness signal | Use visual feedback (color, emoji, animation) exclusively |
| **Tutorial / How-to-Play screen** | Every extra screen before the game is friction. Kids abandon before playing. | Make the first screen the game itself. UI should be self-explanatory |
| **Login / user accounts** | Requires backend, destroys the "open and play" value prop | localStorage for best score only — no identity |
| **Leaderboard** | Requires backend + identity. Also inappropriate for this age (competitive anxiety) | Personal best via localStorage is sufficient motivation |
| **Multiple difficulty levels (v1)** | Adds decision paralysis before the game. Already scoped out in PROJECT.md | One well-tuned difficulty. Revisit if v2 is planned |
| **Skip button** | Letting kids skip removes the tension that makes the game interesting | Timer expiry is the only "escape" — that's already handled as game over |
| **Lives system** | Adds complexity; already decided: wrong answer = game over. Partial-life tracking is confusing for young kids | Game over is clean and final. Score shows progress made |
| **Multiplication / division** | Cognitively inappropriate for 5–9 range (especially 5–7). Already scoped out | Stay with + and − only |
| **External dependencies (CDN, fonts, libraries)** | Single `index.html` requirement means CDN failures break the game | Inline everything. System fonts are fine (or embed 1 Google Font via `@import` if absolutely needed) |
| **Ads or analytics scripts** | Children's privacy law (COPPA) concerns; breaks single-file constraint; parents will distrust the app | No third-party scripts. Zero tracking |
| **Complex SVG/canvas animations** | Overkill for this scope. Risk of performance issues on older phones | CSS animations only. Emoji are sufficient visual reward |
| **Negative number answers** | Confusing and discouraging for 5–9 year olds. Already spec'd out | Ensure `a − b` is only generated when `a >= b` |

---

## Feature Dependencies

```
On-screen numpad buttons
  └── requires: answer display area (shows digits as tapped)
  └── requires: backspace/clear logic
  └── enables: touch-first UX on mobile

Countdown timer (visual)
  └── requires: timer logic (setInterval / requestAnimationFrame)
  └── enables: color-shift differentiator
  └── enables: pulse/shake animation differentiator

Progress indicator ("Q 3 of 10")
  └── requires: question index tracking
  └── enables: star rating on score screen (need total count)

Score tracking
  └── requires: correct answer detection
  └── enables: score screen display
  └── enables: star rating calculation
  └── enables: personal best (localStorage) differentiator

Score screen
  └── requires: final score value
  └── enables: personal best comparison
  └── enables: play again (resets all state)

Encouragement messages (differentiator)
  └── requires: correct answer event
  └── requires: brief display timing before next question

Streak indicator (differentiator)
  └── requires: consecutive correct answer tracking
  └── independent of score tracking (resets differently)

Personal best (differentiator)
  └── requires: score screen
  └── requires: localStorage read/write
```

---

## MVP Recommendation

Given the spec (10 questions, 15s timer, game over on wrong/timeout, single index.html), the following is the right MVP scope:

### Must Ship in v1

1. **Large equation display** with kid-friendly typography
2. **On-screen number pad (0–9 + Submit + Backspace)** — critical for mobile
3. **Visual countdown timer as progress bar + digits** with green→yellow→red shift
4. **Immediate visual feedback** (green flash / red flash) on answer
5. **Progress indicator** (Question X of 10)
6. **Game over screen** (with score, emoji, Play Again button)
7. **Win/score screen** (with star rating, celebration emoji, Play Again button)
8. **Emojis throughout** (already spec-required)

### Add After Core Works (still v1, low cost)

- Timer pulse/shake when < 5 seconds (pure CSS, trivial to add)
- Encouragement messages on correct answer (5 lines of JS)
- Personal best via localStorage (5–8 lines of JS)
- Smooth question fade transition (3 lines of CSS)

### Defer to v2

- Themed emoji per question (operands as emoji counts)
- Streak indicator
- Difficulty levels
- Mascot character

---

## UX Notes for Ages 5–9

These are implementation guidelines that emerge from feature decisions:

| Concern | Guideline |
|---------|-----------|
| **Tap target size** | Minimum 56×56px for all interactive elements. Number pad buttons should be 64–72px |
| **Color contrast** | WCAG AA minimum (4.5:1). Bright colors are fine — just ensure text is readable on them |
| **Font size** | Equation display: 48–72px. Instructions/labels: 20–24px. Never below 18px for any text |
| **Answer confirmation** | Require explicit "Submit/Check" tap — no auto-submit on last digit. Kids mis-tap constantly |
| **Error recovery** | Backspace button on numpad. Kids need to fix mistakes before submitting |
| **Viewport** | Design mobile-first at 375px width. Most parents' phones are iPhone SE / mid-range Android |
| **No horizontal scroll** | Everything must fit in a single viewport height. No scrolling during gameplay |

---

## Sources

- PROJECT.md specification (validated requirements)
- Children's HCI/UX literature: established principles for ages 5–9 (large targets, immediate feedback, minimal text)
- Common patterns in Khan Academy Kids, Prodigy, Math Playground, SplashLearn (domain knowledge)
- COPPA (Children's Online Privacy Protection Act) — relevant for anti-features
- WCAG 2.1 accessibility guidelines for touch targets and contrast

**Confidence:**
| Area | Level | Reason |
|------|-------|--------|
| Table stakes features | HIGH | Grounded in universal children's UX principles + project spec |
| Differentiators | MEDIUM | Based on domain pattern recognition; individual items easy to validate during build |
| Anti-features | HIGH | Clear rationale from spec decisions + legal/technical constraints |
| UX guidelines (sizes, colors) | HIGH | WCAG + children's HCI standards are well-established |
