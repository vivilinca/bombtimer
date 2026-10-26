# Bomb Timer — Product Requirements Document

**Version:** 1.0 (matches shipped build `bomb-timer.html`)
**Date:** 13 September 2026
**Owner:** Rose
**Status:** Live prototype, single-file web app
**Repo:** https://github.com/vivilinca/bombtimer
**Live:** https://vivilinca.github.io/bombtimer/

---

## 1. Summary

Bomb Timer is a countdown timer for a young child, dressed up as a chase. A pink cartoon bomb with a smiley face runs around the screen while the clock counts down. When time runs out, the bomb reaches the clock and explodes. The child sets the time herself and presses Start.

The product exists to make time limits (homework, screen time, tidy-up, reading) something the child watches and owns rather than something a parent enforces.

## 2. Goals and non-goals

**Goals**
- Make a fixed time limit visible, fun, and self-set by the child.
- Give a clear, unmissable "time is up" moment that ends the activity without a parent having to intervene.
- Zero setup: open one file in any browser, no account, no install.

**Non-goals (v1)**
- Not a task manager, reward system, or habit tracker.
- No multiple timers, no pause/resume, no scheduling.
- No progress data or parental dashboard.

## 3. Users

| User | Role | Needs |
|---|---|---|
| Child (primary, ~4–8 years) | Sets the timer, watches, reacts | Big tap targets, no reading required to operate, delight, a fair and predictable ending |
| Parent (secondary) | Suggests the duration, opens the app | Fast to launch, reliable ending signal, sound can be turned off |

## 4. User flow

1. **Setup** — Card shows minutes and seconds with up/down arrows, a row of presets (30 s, 1, 3, 5, 10, 15 min), and a large Start button.
2. **Running** — Setup card disappears. Clock sits centre-screen showing MM:SS. Bomb appears in a random corner and starts running. Sound and Stop buttons appear top-right.
3. **Countdown pressure** — At 10 s the clock jitters, a tick plays each second, and the bomb's face turns worried. At 3 s the tick pitch rises, the clock shakes harder, the bomb goes wide-eyed and charges the clock.
4. **Explosion** — At 00:00 the bomb hits the clock. Starburst, screen shake, "BOOM!", flying hearts and stars, bomb face becomes X X, clock cracks.
5. **Done** — "Time's up!" card with Play again. Play again returns to Setup with the previous duration preserved.
6. **Stop** (any time while running) — returns to Setup immediately, no explosion.

## 5. Functional requirements

### 5.1 Time setting
- Minutes 0–99 via ±1; seconds in steps of 5, wrapping 55 → 0 → 55.
- Minimum duration 5 s; the UI silently enforces it.
- Presets set both fields and highlight when the current value matches one.
- Last-used duration persists for the session so Play again is one tap.

### 5.2 Bomb behaviour
- Runs at 170–330 px/s, speeding up as time elapses.
- Picks random destinations at least 110 px inside the screen edge and away from the clock; re-picks on arrival, on hitting an edge, or at random (~every 3 s) so the path never looks scripted.
- Steers away from screen edges and from the clock's exclusion zone while wandering — it must never touch the clock early.
- Only turns to face the other direction when clearly reversing (hysteresis) so it never flickers left‑right.
- Leaves a dashed pink line behind it (the "rope" it runs along), plus speed lines when moving fast.
- **Final 4 s:** heads straight for the clock at exactly the speed needed to arrive at 00:00. The ending must be deterministic — the bomb always makes contact at zero, never early, never late.

### 5.3 Bomb expressions
| Time remaining | Face | Notes |
|---|---|---|
| > 10 s | Smile; occasional wink | Random wink while ticking |
| ≤ 10 s | Worried (frown) | Signals "hurry up" |
| ≤ 4 s | Scared (wide eyes, O mouth) | Spark flares larger |
| After explosion | X X eyes | |

### 5.4 Clock
- Pink alarm-clock design: two bells, dark screen, pink tabular digits, heart row, feet.
- States: normal → `hurry` (≤10 s, small jitter) → `panic` (≤3 s, fast jitter, yellow digits) → `dead` (tilted, struck-through 00:00).
- Always rendered beneath the bomb and the explosion.

### 5.5 Explosion
- ~1.6 s sequence: expanding 12‑point starburst (orange → yellow → cream core), pink puffs, 70 particles (hearts, stars, dots) under gravity.
- Screen shake decays over ~1.2 s.
- "BOOM!" text pops at 250 ms.
- Boom sound: filtered noise burst.
- Done card appears after 2.3 s so the explosion is never interrupted.

### 5.6 Sound
- Tick each second from 10 s (880 Hz), higher (1320 Hz) from 3 s.
- Sound toggle on the running screen; default on. Audio context is created on Start (user gesture) so it works on iOS.

### 5.7 Controls
- All tap targets ≥ 44 px. Visible keyboard focus ring.
- Stop is always available while running.

## 6. Visual and audio design

- **Style reference:** the uploaded character sheet — flat cartoon, thick dark outlines, pink palette, starburst effects.
- **Palette:** pink `#F48FB1`, dark pink `#D95A8A`, blush `#FDE1EC`, ink `#2B2035`, yellow `#FFE14A`, orange `#FF8A3D`, cream `#FFF6F1`.
- **Type:** Fredoka (rounded, single family). Numbers use tabular figures so the clock doesn't shift width.
- **Background:** cream with a faint pink dot grid.
- **Motion principle:** everything that moves answers the countdown. The bomb is the one bold element; the UI chrome stays quiet.

## 7. Technical requirements

- Single HTML file, no build step, no dependencies beyond Google Fonts (degrades to system font offline).
- Canvas 2D for bomb, trail, explosion; DOM for clock and cards (crisp text, easy styling).
- Timing derived from `performance.now()` against the start timestamp — never from frame counts — so background tabs and dropped frames don't drift the clock.
- Canvas scaled for devicePixelRatio up to 2×; responsive to any viewport; handles resize mid-run.
- Respects `prefers-reduced-motion` (clock jitter disabled).
- Layering: clock (z1) < canvas (z2) < BOOM text (z4) < cards and top bar (z5).

## 8. Success criteria

- Child can set a timer and start it without adult help on first use.
- Bomb never contacts the clock before 00:00 (0 early explosions in 20 runs across phone, tablet, laptop).
- No visible direction flicker at screen edges.
- Explosion is recognisable as "time's up" from across the room, with sound off.
- Play again returns to a clean setup screen every time.

## 9. Open questions / v2 candidates

- Pause/resume, or is Stop-and-restart enough?
- Named activities ("Reading", "Tidy up") shown on the clock.
- Choice of bomb expression sets or a second character (the sheet includes sunglasses, hearts, laughing).
- Let the child draw the bomb's path before Start.
- Optional "warning" chime at the halfway mark.
- Save preferred durations across sessions (needs local storage — not available in the current hosted preview, fine as a downloaded file).

## 10. Hosting and deployment

- **Repository:** https://github.com/vivilinca/bombtimer (public, branch `main`). First pushed 26 October 2026.
- **Live URL:** https://vivilinca.github.io/bombtimer/ — served by GitHub Pages from the root of `main`.
- **Files in the repo:**
  - `bomb-timer.html` — the app.
  - `index.html` — redirects the site root to `bomb-timer.html`.
  - `README.md` — short description and play link.
  - `bomb-timer-prd.md` — this document.
- **Updating:** commit and push to `main`; GitHub Pages redeploys automatically within a minute or two. No build step.
