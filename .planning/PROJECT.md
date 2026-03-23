# Kids Math Game

## Overview

A fun, browser-based math game designed for young children. The game presents 10 random addition and subtraction puzzles (results always < 20) with a 15-second countdown timer per puzzle. Wrong answer or timeout = game over. Completing all 10 shows a score screen with a play-again button. Built with plain HTML + JavaScript only, hosted on GitHub Pages.

## Goals

- Give kids a fun, engaging way to practice basic math
- Keep it dead simple: no setup, no login, just open and play
- Deploy instantly via GitHub Pages (single `index.html`)

## Target User

Young children (ages 5–9), likely playing on a parent's phone or tablet.

## Core Value

A delightful, no-friction math game that kids can play instantly in any browser.

## Tech Stack

- Plain HTML5
- Vanilla JavaScript (no frameworks, no build tools)
- Inline CSS for styling
- Single `index.html` file — deploys directly to GitHub Pages

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Generate 10 random math puzzles per game (addition and subtraction only)
- [ ] All puzzle results are less than 20 (and operands are non-negative)
- [ ] Each puzzle has a 15-second countdown timer visible to the player
- [ ] Timer expiring = game over immediately
- [ ] Player enters a numeric answer and submits
- [ ] Correct answer advances to the next puzzle
- [ ] Wrong answer = game over immediately
- [ ] After all 10 puzzles solved, show final score + "Play Again" button
- [ ] Game over screen shows score + "Play Again" button
- [ ] Fun emojis / kid-friendly symbols used throughout UI
- [ ] Fully self-contained in a single `index.html` (no external dependencies)

### Out of Scope

- Multiple difficulty levels — keep it simple for v1
- User accounts / leaderboard — no backend
- Sound effects — avoids autoplay restrictions
- Multiplication / division — too advanced for target age

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Single index.html | GitHub Pages simplicity, zero build step | — Pending |
| Results < 20 | Age-appropriate math for 5–9 year olds | — Pending |
| Game over on wrong answer | Adds tension and fun; keeps sessions short | — Pending |
| 15-second timer | Long enough for young kids, short enough to feel exciting | — Pending |

---
*Last updated: 2026-03-23 after initialization*
