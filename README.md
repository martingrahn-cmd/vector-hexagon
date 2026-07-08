# VECTOR HEXAGON

A twitch survival game in the spirit of **Super Hexagon**: one hexagonal core,
six lanes, and walls closing in from the edge of the screen. Slip your cursor
into the gap before each wall reaches the centre. Miss, and you start over.

One self-contained `index.html` — HTML canvas, no build step, no dependencies.
Plays on desktop and mobile.

## Play

Open `index.html` in a browser, or serve the folder:

```sh
python3 -m http.server 8791
# open http://localhost:8791
```

Published via GitHub Pages once enabled (Settings → Pages → deploy from
`main`): `https://martingrahn-cmd.github.io/vector-hexagon/`.

## Install as an app (PWA)

Over HTTPS (i.e. from GitHub Pages, not a plain `file://`), the game is an
installable Progressive Web App: it runs full-screen, has its own home-screen
icon, and **plays fully offline** — a service worker (`sw.js`) caches the shell,
icons and the soundtrack on first visit.

- **Desktop / Android (Chrome/Edge):** an **INSTALL APP** button appears on the
  title screen (browser install icon works too).
- **iOS Safari:** tap **Share → Add to Home Screen** (the title screen shows a
  hint). Launches standalone, no browser chrome.

`manifest.webmanifest` uses relative paths, so it installs correctly from the
project subpath. Bump `CACHE` in `sw.js` when cached files change to push an
update to installed clients.

## Controls

Fully keyboard-navigable. In-game you steer; on the menu the arrows move a focus
ring. **Escape is deliberately not used** for back (fullscreen / the portal
iframe hijack it) — back is **Backspace**.

| Input | Action |
|---|---|
| `←` / `→` or `A` / `D` | rotate the cursor around the core (in game) · change the focused menu value |
| `↑` / `↓` or `W` / `S` | move the menu focus (menu) · scroll a screen |
| **Tap & hold** left / right half (mobile, mouse) | rotate that way |
| `Enter` / `SPACE` / tap | select · begin · retry |
| `Backspace` | back to menu / close a screen |
| `M` | toggle sound |

The title screen has **HIGH SCORES** and **TROPHIES** (the 31-trophy grid).
High scores has two tabs: **LOCAL** (your best time per mode+tier) and
**GLOBAL** — the live Supabase leaderboard (when embedded in GameVolt) with a
board picker and TOP / AROUND ME / paging / ±1000 controls, so you can jump to
the leader, your own neighbourhood, or any rank (e.g. #10000). Standalone, the
GLOBAL tab prompts you to sign in on GameVolt.

## Modes

Pick on the title screen:

- **CLIMB** — the tier-clear campaign below: survive, auto-advance, win by
  clearing Hexagonest.
- **ENDLESS** — no tier clears; the difficulty ramps forever and you chase a
  survival high score (tracked separately per starting tier).

## Shifting shape (4 / 5 / 6 lanes)

Every ~12–20s the field **morphs between a square, a pentagon and a hexagon** —
it collapses to a point, swaps the lane count and reblooms in the new shape,
which changes the whole dodging rhythm. Morphs only happen while the screen is
clear of walls, so the geometry is never ambiguous. Wall patterns are generated
against the current shape (tunnel · spiral · zig-zag · spike · double-gap ·
C-shape · 180-reversal · closing spiral · double-back fake-out · stutter), with
the nastier ones weighted in on higher tiers and deeper into an endless run.

## Clearing a tier (60s) and climbing

Survive **60 seconds** on a tier and the run **auto-advances into the next one,
seamlessly, in the same run** — the field flips its spin, snaps to the tier's
colour and calls the new level, while the timer keeps running. A **ring meter**
at the edge of the arena fills toward each clear so you can see it coming. Start
on any tier; clearing the last one (**HEXAGONEST**) is a **YOU WIN**, and each
tier you clear from earns a ★ on the menu (persisted). The HUD shows the tier
you're currently in.

## Trophies

31 trophies — 15 bronze, 10 silver, 5 gold and 1 platinum that unlocks once all
the others are earned (survive-time thresholds, clearing tiers, winning, seeing
each shape, close-call runs, endless survival, run counts…). They unlock from
play, persist to `localStorage`, and pop a toast when earned. Browse them on the
**🏆 TROPHIES** screen from the title. Difficulty note: the easiest tier
(**HEXAGON**) has a gentle base speed and a shallow ramp so it's an approachable
way in.

## Feel

- **Announcer** — a spoken "BEGIN" / "GAME OVER" (Web Speech) plus on-screen
  callouts, and survival-milestone shouts as you last longer: LINE (10s),
  TRIANGLE, SQUARE, PENTAGON, HEXAGON, PERFECT (60s).
- **Close-call feedback** — thread a wall by a hair and the cursor flares, the
  screen flashes and a high tick fires (with a subtle phone buzz on razor-thin
  passes). The closer the pass, the stronger the cue.
- **Crash** — hitting a wall fires a proper impact: a filtered noise burst, a
  low thud and a metallic ring-out.
- **Haptics** — mobile devices vibrate on a crash (`navigator.vibrate`).

## Music (Suno-ready)

The field pulses, flashes and flips its spin **in time with whatever is
playing**, because both soundtrack sources feed one beat engine:

- **SYNTH** (default) — a small procedural WebAudio groove, always available.
- **Your own MP3** — click **＋ LOAD MP3** and pick a track (e.g. one you made
  in [Suno](https://suno.com)). The game runs an `AnalyserNode` over it and
  detects beats live from the bass energy, so the visuals dance to *any* song —
  no need to enter a BPM.

To ship a soundtrack with the game, commit it as **`assets/track.mp3`**; it is
picked up automatically on the first run. Missing file → the synth groove plays,
no error.

## Modes

Three difficulties, each with its own best time saved to `localStorage`:

- **HEXAGON** — approachable pace, calmer patterns
- **HEXAGONER** — faster walls, tighter spin
- **HEXAGONEST** — brutal

## How it works

The player is an angle orbiting the core at a fixed radius; walls are hexagonal
rings that shrink inward, each blocking a subset of the six lanes and leaving at
least one gap. Collision is a pure polar-domain check — is the player's centre
lane blocked while the ring crosses the orbit radius — so there are no colliders
and no tunnelling. Walls come from a difficulty-weighted pattern spawner
(tunnel · spiral · zig-zag · spike · C-shape) with a short grace window at the
start of each run. The field spins, pulses on the beat and drifts through hues;
the soundtrack is a small procedural WebAudio groove.

## Portal / host integration

The game is local-first (bests + trophies in `localStorage`), but it emits its
results so a host site can collect scores with **no backend required**:

- **Embedded in an iframe** — every run end and achievement unlock is
  `postMessage`'d to the parent window. Listen for it:
  ```js
  addEventListener('message', e => {
    if (e.data?.type === 'vhex:score')       { /* {mode,tier,time,won,personalBest,isNewBest,player,ts} */ }
    if (e.data?.type === 'vhex:achievement') { /* {id,name,desc} */ }
  });
  ```
- **Auto-POST to an API** — set a config before the game script runs and every
  event is `fetch`-POSTed there as JSON:
  ```html
  <script>window.VECTOR_HEXAGON_CONFIG = { apiUrl: '/api/scores', playerId: 'abc', targetOrigin: 'https://your.portal' };</script>
  ```
  You can also send it later (e.g. after auth) via
  `iframe.contentWindow.postMessage({ type:'vhex:config', config:{ apiUrl, playerId } }, '*')`.
- **Read API** — `window.VectorHexagon` exposes `version`, `getBests()`,
  `getAchievements()`, `getRunsPlayed()`, and `on('score'|'achievement', cb)`.

Payloads include `game`, `version`, and `player` so the portal can attribute
and version them. Nothing is sent anywhere unless a parent frame or `apiUrl`
is present — standalone play stays fully local and offline.

Built with Claude Code.
