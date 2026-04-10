# Goal Crushed Celebration & GIF Export — Design

**Date:** 2026-04-09
**Status:** Draft, pending approval

## Problem

The team hit their monthly goal for the first time. The current dashboard caps visually at 100% and treats "goal reached" the same as "pace is fine" — no recognition of the accomplishment. Additionally, the team wants a way to share the dashboard in Microsoft Teams that is more engaging than a screenshot.

## Goals

1. When `totalSold >= goal`, visibly celebrate the accomplishment without hiding or obscuring any existing data.
2. Provide a one-click button that exports an animated GIF of the dashboard suitable for posting in Teams chat.

## Non-goals

- Persisting celebration state across reloads (CSV data is reloaded each session per existing architecture).
- Multiple animation themes or user-configurable celebration styles.
- GIF export of arbitrary historical reports — only the currently-loaded dataset.

## Part 1 — Celebration UI (triggered when `totalSold >= goal`)

### Hero area

- Subtitle `Remaining to Goal` → `🏆 GOAL CRUSHED — BONUS`
- Big number switches from `$remaining` (gold) to `+$overage` in bright celebration green (`#4ade80`)
- New badge under the sold/goal line: `107% — $5,234 over` (pill, green background)
- Percentage circle: arc completes a full ring, then a second gold→green overlay ring renders the surplus portion. Gentle pulsing glow via `box-shadow` keyframe.

### Progress bar — overflow treatment

- Primary bar still fills to 100%.
- A vertical gold tick marks the 100% line.
- A second "overflow" segment extends past the track (track widens to accommodate up to ~150% visually; surplus beyond that caps at the edge but the numeric badge shows the real percentage).
- Overflow segment: animated shimmering gold→green gradient via CSS keyframe background-position sweep.
- Soft green glow via `box-shadow` on the whole bar.

### Confetti

- ~40 small divs spawned via JS, positioned absolutely over the dashboard card, animated with CSS keyframes (fall + rotate + fade).
- Colors drawn from a palette: gold, green, red, blue, white.
- Fires **once per CSV load** (tracked via a module-level `confettiFiredForCurrentData` flag, reset in the CSV load handler). Does NOT re-fire when filters change.
- Total runtime ~3 seconds; elements removed from DOM after animation ends.

### Pace indicator

- New `exceeded` class (green, 🎉 icon).
- Text: `Goal exceeded by $5,234 (107%) — keep going!`
- Takes precedence over `ahead`/`on-track`/`behind` classes.

### Ambient touches

- Dashboard card gets a slow gold↔green animated border glow when in exceeded state (`@keyframes` on `box-shadow`).
- Title gets a ✨ prefix.

### Data visibility guarantee

Every existing number (sales count, total sold, avg premium, daily chart, deadline, days left) remains in place and fully legible. The celebration is additive decoration on top of the normal layout.

## Part 2 — GIF export

### User flow

1. Button `📥 Share as GIF` appears next to the CSV loader once data is loaded.
2. User clicks → full-screen overlay appears: "Rendering GIF…" with a progress bar.
3. Frames are captured off-screen; user sees the overlay but the live dashboard is untouched.
4. When encoding finishes, browser downloads `sales-report-YYYY-MM-DD.gif`.
5. Overlay disappears, dashboard returns to its normal state.

### Animation sequence (~6 seconds, loops)

| Phase | Duration | Content |
|---|---|---|
| Intro | 0.5s | Empty dashboard, title visible, all counters at 0 |
| Count-up | 1.5s | Sales count / sold $ / avg premium roll from 0 to final values (ease-out); progress bar sweeps from 0% to final %; percentage arc draws in sync |
| Transition | 0.3s | Brief flash at 100% line if goal was exceeded; otherwise straight to hold |
| Celebration (conditional) | 1.5s | Only if `totalSold >= goal`: confetti burst, "GOAL CRUSHED" reveal, overflow segment extends past 100%, glow pulses |
| Hold | 2s | Everything frozen on final state — readable numbers |
| Loop | — | GIF loops infinitely (standard GIF behavior) |

**Pre-goal case:** intro + count-up + hold (skips transition flash and celebration). Total ~4 seconds.

### Technical approach

- **`html2canvas`** (CDN, `https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js`) captures DOM elements as canvas frames.
- **`gif.js`** (CDN, `https://cdn.jsdelivr.net/npm/gif.js@0.2.0/dist/gif.js` + worker URL) encodes frames into a GIF blob.
- Frame rate: **15 fps** → ~90 frames total (6 sec). Trade-off: smooth enough for counters, keeps file size reasonable (target <3 MB).
- Capture target: a dedicated off-screen clone of the `.dashboard` element, sized to a fixed width (720px) for consistent output regardless of viewport. Clone lives in a hidden container positioned off-screen (`left: -9999px`). The live dashboard is NOT disturbed.
- For each frame, JS sets the clone's DOM state (counters, bar width, confetti opacity, etc.) synchronously, then `html2canvas` captures it, then `gif.addFrame(canvas)` stores it.
- Encoding runs in a Web Worker (provided by `gif.js`) so the UI doesn't freeze. Progress events update the overlay progress bar.

### Library inclusion

Per the updated CLAUDE.md rule (see changes below), CDN script tags are acceptable. Both libraries are loaded via `<script>` tags in the `<head>`. No npm, no build step, no bundling. Still a single `index.html` served directly.

### Failure modes

- **Network failure loading CDN libraries:** Button is disabled with a tooltip "GIF export unavailable — libraries failed to load" if either library is undefined when the user clicks.
- **Encoding failure (gif.js worker error):** Error caught, overlay replaced with a "Something went wrong rendering the GIF — try again?" message with a dismiss button.
- **Very large GIF (>10 MB):** Not prevented but logged to console. 720px width + 15fps + 6s should stay well under this.

## Part 3 — CLAUDE.md update

Change the "Architecture Rules" section to allow CDN-loaded JS libraries:

**Before:**
> - **No frameworks or libraries** — vanilla HTML/JS/CSS only. No React, no jQuery, no bundlers.

**After:**
> - **No frameworks, no bundlers** — vanilla HTML/JS/CSS. Small utility libraries loaded via CDN `<script>` tags are allowed when they make a feature meaningfully better (e.g., `html2canvas`, `gif.js`). No React, no jQuery, no npm, no build step.

## Out of scope / deferred

- Customizing GIF duration or fps via UI.
- Exporting as MP4/WebM (would require different encoding path).
- Sharing directly to Teams API (user copies the GIF manually).
- Sound effects.

## Open questions

None — all clarified in brainstorming:
- Confetti fires once per CSV load, not per filter change. ✅
- Celebration triggers as soon as `totalSold >= goal`, regardless of deadline. ✅
- Pre-goal GIF still shows count-up and progress bar fill, ending on current pace state. ✅
