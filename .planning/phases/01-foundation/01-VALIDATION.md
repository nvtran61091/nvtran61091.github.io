---
phase: 1
slug: foundation
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-23
---

# Phase 1 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Browser DevTools console (no test runner — single HTML file project) |
| **Config file** | none |
| **Quick run command** | Open `index.html` in browser, paste puzzle-validator snippet in DevTools console |
| **Full suite command** | Paste full validation script in DevTools console (provided in RESEARCH.md) |
| **Estimated runtime** | ~5 seconds |

---

## Sampling Rate

- **After every task commit:** Open `index.html` in browser and visually verify the screen renders
- **After every plan wave:** Run full DevTools console validation script
- **Before `/gsd-verify-work`:** Full script must pass + all 4 screens verified on mobile viewport
- **Max feedback latency:** ~30 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | Status |
|---------|------|------|-------------|-----------|-------------------|--------|
| 1-01-01 | 01 | 1 | PUZZLE-01–05 | console | Paste puzzle stress test in DevTools | ⬜ pending |
| 1-01-02 | 01 | 1 | UI-02/03/06 | manual | Open in browser + DevTools mobile emulator | ⬜ pending |
| 1-01-03 | 01 | 1 | DEPLOY-01–03 | manual | Push to main, verify GitHub Pages URL loads | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

None — no test runner or framework needed. All validation is via browser DevTools console or visual inspection.

*Existing infrastructure covers all phase requirements.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| All 4 screens render on mobile | UI-06 | No automated DOM size test without runner | Open in Chrome DevTools → toggle device toolbar → iPhone SE viewport |
| Numpad buttons ≥ 44×44px | UI-03 | Pixel measurement requires DevTools | Inspect a numpad button → check computed size in DevTools |
| Fonts ≥ 16px base | UI-02 | Visual + DevTools computed styles | Inspect body font-size + puzzle text size |
| GitHub Pages serves with no errors | DEPLOY-03 | Requires live deployment | Push to main → open `https://nvtran61091.github.io` → check console for errors |

---

## Validation Sign-Off

- [ ] All tasks have manual verify instructions or console script
- [ ] Wave 0: N/A (no test runner)
- [ ] No watch-mode flags
- [ ] Feedback latency < 30s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
