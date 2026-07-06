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
