# Kids Math Game

## What This Is

A Vietnamese-language, browser-based math game for young children (ages 5–9). Players answer 10 addition/subtraction puzzles using an on-screen numpad. A 15-second countdown timer adds excitement. Wrong answer or timeout ends the game immediately. Built as a single `index.html` deployed on GitHub Pages — no install, no login, open and play.

## Core Value

A delightful, no-friction math game that kids can play instantly in any browser.

## Current State

**Shipped: v1.0 MVP** (2026-03-24)
- `index.html` (~590 lines): full game playable end-to-end
- Web Audio API sounds: background music, correct chime, wrong buzz, win fanfare
- Vietnamese UI throughout (`lang="vi"`)
- Live at: https://nvtran61091.github.io
- Puzzle results capped at 9 (not 20 as originally planned)
- Countdown timer implemented and working

## Tech Stack

- Plain HTML5 + Vanilla JavaScript (ES2020)
- Inline CSS with CSS custom properties
- Web Audio API (no external files)
- Single `index.html` — GitHub Pages, zero build step

## Target User

Young children (ages 5–9), playing on a parent's phone or tablet.

## Requirements

### Validated — v1.0

- ✓ Single `index.html`, no external dependencies — v1.0
- ✓ Puzzle generation: addition + subtraction, non-negative, results 1–9 — v1.0
- ✓ Subtraction constraint (b ≤ a, never negative answer) — v1.0
- ✓ 10 unique puzzles per session (dedup Set) — v1.0
- ✓ Large readable fonts (clamp-based fluid typography) — v1.0
- ✓ Touch targets ≥ 64px (numpad buttons) — v1.0
- ✓ Responsive layout: mobile + desktop (min(480px,100%)) — v1.0
- ✓ On-screen numpad input (no iOS keyboard popup) — v1.0
- ✓ Backspace / empty-submit guard — v1.0
- ✓ 15-second countdown timer with color bar — v1.0
- ✓ Timeout → game over with correct answer reveal — v1.0
- ✓ Wrong answer → game over — v1.0
- ✓ Correct answer progression through 10 puzzles — v1.0
- ✓ Results screen with stars and all 10 answers — v1.0
- ✓ Play Again fully resets state — v1.0
- ✓ Sound effects + background music (Web Audio API) — v1.0
- ✓ Vietnamese labels throughout — v1.0

### Active — v1.1

- [ ] Visible positive feedback animation on correct answer (green flash < 300ms)
- [ ] Celebratory results screen enhancement (star rating)

### Out of Scope

- Multiple difficulty levels — keep simple
- User accounts / leaderboard — no backend
- Multiplication / division — too advanced for ages 5–9
- Timer pause on tab switch — defer to v2
- External fonts / icon libraries — single-file constraint

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Single index.html | GitHub Pages simplicity, zero build step | ✓ Works well |
| Results < 10 (not 20) | User adjusted: simpler for young kids | ✓ Better fit |
| Game over on wrong answer | Adds tension; keeps sessions short | ✓ Good |
| 15-second timer | Long enough for kids, short enough to feel exciting | ✓ Good |
| Web Audio API (no files) | No external assets, no autoplay restriction | ✓ Works on all browsers |
| Vietnamese UI | User's language preference | ✓ Shipped |
| On-screen numpad | Avoids iOS keyboard popup; touch-friendly | ✓ Works well |
| Zero-answer filter | Young kids may not recognize 0 as valid answer | ✓ Good UX |

---
*Last updated: 2026-03-24 after v1.0 milestone*
