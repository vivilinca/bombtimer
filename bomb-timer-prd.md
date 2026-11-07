# Bomb Timer — Product Requirements Document

**Version:** 1.1 (matches shipped build `bomb-timer.html`)
**Date:** 7 November 2026
**Owner:** Rose
**Status:** Live prototype, single-file web app
**Repo:** https://github.com/vivilinca/bombtimer
**Live:** https://vivilinca.github.io/bombtimer/

---

## 1. Summary

Bomb Timer is a countdown timer for a young child, dressed up as a chase. A pink cartoon bomb with a smiley face runs along a dashed route around the clock while it counts down. When time runs out, the bomb reaches the clock and explodes. The child sets the time herself and presses Start.

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
2. **Running** — Setup card disappears. Clock sits centre-screen showing MM:SS. A randomly chosen shape appears around it as a dashed route, and the bomb starts running along it. On longer timers it moves on to a different shape after each lap. Sound and Stop buttons appear top-right.
3. **Countdown pressure** — At 10 s the clock jitters and a tick plays each second. At 3 s the tick pitch rises, the clock shakes harder and its digits turn yellow. The bomb looks anxious for the last 6 s and scared for the last 2.2 s (less on short timers, see 5.3), and finishes the route with a fast final charge.
4. **Explosion** — At 00:00 the bomb reaches the end of the route and lands on the clock. Starburst, screen shake, "BOOM!", flying hearts and stars, bomb face becomes X X, clock tilts and its 00:00 is struck through. Sound and Stop buttons disappear.
5. **Done** — "Time's up!" card with Play again. Play again returns to Setup with the previous duration preserved.
6. **Stop** (any time while running) — returns to Setup immediately, no explosion.

## 5. Functional requirements

### 5.1 Time setting
- Default duration 05:00.
- Minutes 0–99 via ±1; seconds in steps of 5, wrapping 55 → 0 → 55.
- Minimum duration 5 s; the UI silently enforces it. At 0 minutes the seconds stop at 5 going down instead of wrapping.
- Presets set both fields and highlight when the current value matches one.
- Last-used duration persists for the session so Play again is one tap.

### 5.2 Bomb behaviour
**Route**
- A route is a chain of legs, each one lap of a shape: circle, triangle, rectangle, maze, bunny, frog, horse or unicorn.
- The bomb must always be on a fresh line. It never repeats a lap of the same shape back to back: all eight shapes are shuffled and used before any repeats, and the same shape never comes twice in a row.
- Each leg gets its own size (85–100%), and shapes also vary by rotation (circle, triangle), mirroring (animals), row count (maze, 4 or 6) and lap direction, so a repeated shape takes a different line.
- The number of legs scales with the timer, aiming for an average of 175 px/s at any duration. Very short timers run a single leg, so the bomb averages faster than the target.
- Each leg joins on from where the last one ended: closed shapes start at their nearest point, the maze at its nearest end.
- Legs are centred on the clock and sized from the viewport. Resizing mid-run rebuilds them to fit. When the clock fills the short side of the screen (phones), circle, triangle and animal legs stretch up to 1.6× along the long side so they stay recognisable.
- The route keeps 36 px clear of a rounded box around the clock. Points that land inside are pushed out to it, so the route wraps around the clock instead of covering it, and smoothing can bring it slightly closer.
- All corners are rounded, including animal outlines, so the line always looks soft and never faceted.
- The last leg ends with a straight run in to the clock's keep-out edge.
- Only the current leg is drawn, ahead of the bomb, as a soft pink track with an animated dashed "rope" on top. The part the bomb has passed disappears, and the next leg appears when the bomb reaches it.

**Pace**
- Progress along the route follows a randomised schedule of beats (about one per 8 s of timer, at least 2), plus a final fast charge. Beats stay short on long timers, so the bomb never stalls for long.
- Each beat is fast (55%), slow (27%) or pause (18%). Timers of 12 s or less have no pauses, so slow takes the 45%.
- The schedule never moves the bomb backwards, and it is pinned at both ends. The ending is deterministic: the bomb reaches the end of the route at exactly 00:00, never early, never late, then lands on the clock centre as it explodes.

**Animation**
- Only turns to face the other direction when the route clearly reverses (hysteresis) so it never flickers left‑right.
- Leg stride and bob scale with speed and fade out when the bomb pauses.
- A sparkling aura surrounds the bomb: radial glow, tilted ring, 7 orbiting stars that pass in front of and behind it, and 6 twinkles. A comet-glow streak trails it when moving fast.

### 5.3 Bomb expressions
The face follows the bomb's movement until the final seconds, when time remaining takes over.

| When | Face | Notes |
|---|---|---|
| Start of run, slow beat, or below ~150 px/s | Happy (^_^ eyes, grin, tongue) | |
| Fast beat at higher speed | Hurry (determined brows, open mouth) | |
| Pause beat, or below ~23 px/s | Doubt (one eye open, one squint, raised brow, wavy mouth) | Takes priority over Happy and Hurry |
| Anxious threshold | Anxious (worried brows, small open mouth, sweat drop) | Spark and aura flare larger |
| Scared threshold | Scared (wide eyes, O mouth, raised brows) | Spark and aura flare largest |
| After explosion | X X eyes, O mouth | |

Thresholds adapt to the timer's length:
- **Scared:** the smaller of 2.2 s and 14% of the total.
- **Anxious:** the smaller of 6 s and 33% of the total, and always at least 0.5 s before Scared.

### 5.4 Clock
- Pink alarm-clock design: two bells, dark screen, pink tabular digits, heart row, feet.
- States: normal → `hurry` (≤10 s, small jitter) → `panic` (≤3 s, fast jitter, yellow digits) → `dead` (tilted, struck-through 00:00).
- Always rendered beneath the bomb and the explosion.

### 5.5 Explosion
- ~1.6 s sequence: expanding 12‑point starburst (orange → yellow → cream core), pink puffs, 70 particles (hearts, stars, dots) under gravity.
- Screen shake decays over ~1.2 s.
- The bomb stays visible on the clock for the first 250 ms, then the explosion replaces it.
- "BOOM!" text pops at 250 ms.
- Boom sound: filtered noise burst.
- Done card appears after 2.3 s so the explosion is never interrupted.

### 5.6 Sound
- Tick each second from 10 s (880 Hz), higher (1320 Hz) from 3 s.
- Sound toggle on the running screen; default on. It also silences the boom.
- The audio context is created on the first animation frame after Start is pressed.

### 5.7 Controls
- Tap targets: stepper arrows are 56 × 38 px, presets are about 40 px tall, Sound and Stop about 44 px tall, and Start and Play again are large full-width buttons.
- Visible keyboard focus ring.
- Stop is always available while running.

## 6. Visual and audio design

- **Style reference:** the uploaded character sheet — flat cartoon, thick dark outlines, pink palette, starburst effects.
- **Palette:** pink `#F48FB1`, dark pink `#D95A8A`, blush `#FDE1EC`, ink `#2B2035`, yellow `#FFE14A`, orange `#FF8A3D`, cream `#FFF6F1`.
- **Type:** Fredoka (rounded, single family). Numbers use tabular figures so the clock doesn't shift width.
- **Background:** cream with a faint pink dot grid.
- **Motion principle:** everything that moves answers the countdown. The bomb is the one bold element; the UI chrome stays quiet.

## 7. Technical requirements

- Single HTML file, no build step, no dependencies beyond Google Fonts (degrades to system font offline).
- Canvas 2D for bomb, route, explosion; DOM for clock and cards (crisp text, easy styling).
- Animal routes are stored as SVG path data and sampled once at load through a hidden SVG element.
- Timing derived from `performance.now()` against the start timestamp — never from frame counts — so background tabs and dropped frames don't drift the clock.
- Canvas scaled for devicePixelRatio up to 2×; responsive to any viewport; handles resize mid-run.
- Respects `prefers-reduced-motion`: clock jitter disabled, screen shake and aura pulse reduced.
- Layering: clock (z1) < canvas (z2) < BOOM text (z4) < cards and top bar (z5).

## 8. Success criteria

- Child can set a timer and start it without adult help on first use.
- Bomb explodes only at 00:00 (0 early explosions in 20 runs across phone, tablet, laptop).
- No visible direction flicker along the route.
- On any timer length, the bomb never runs the same shape twice in a row and route lines never pile up into one.
- Route lines look smooth on every shape, with no faceted corners.
- Animal routes stay recognisable on phone, tablet and laptop.
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
