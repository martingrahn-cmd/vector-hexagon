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

## Controls

| Input | Action |
|---|---|
| `←` / `→` or `A` / `D` | rotate the cursor around the core |
| **Tap & hold** left / right half (mobile, mouse) | rotate that way |
| `SPACE` / `Enter` / tap | begin · retry |
| `Esc` | back to menu |
| `M` | toggle sound |

## Feel

- **Announcer** — a spoken "BEGIN" / "GAME OVER" (Web Speech) plus on-screen
  callouts, and survival-milestone shouts as you last longer: LINE (10s),
  TRIANGLE, SQUARE, PENTAGON, HEXAGON, PERFECT (60s).
- **Close-call feedback** — thread a wall by a hair and the cursor flares, the
  screen flashes and a high tick fires (with a subtle phone buzz on razor-thin
  passes). The closer the pass, the stronger the cue.
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

Built with Claude Code.
