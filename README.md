# Lunar-Lander-Culebra

[aayala4/Lunar-Lander-Python](https://github.com/aayala4/Lunar-Lander-Python)
ported to [Culebra](https://github.com/yhirose/culebra).

The original is a Lunar Lander written for a Fundamentals of Computing course,
running on pgzero (Pygame Zero). This port keeps its 1400x800 playfield, its
physics constants, its midpoint-displacement terrain and its scoring, and
replaces pgzero with Culebra's `Canvas`, so it opens a real window on macOS,
Linux and Windows.

## Running

Culebra 0.4.0 or later, built with the Canvas window backend (the default on
macOS).

```bash
culebra lunar_lander.cul       # play
culebra --jit lunar_lander.cul # the same, JIT-compiled
```

P starts a round. Left and Right rotate the hull, Up and Down work the
throttle, Q returns to the title screen, Esc quits. The hull spawns pointing
sideways, so the first thing to do is rotate it upright.

Put both feet down under 12 units of vertical speed and 25 of horizontal for a
good landing, worth 50 points and 50 units of fuel back. The flattened pads
marked `2x` through `5x` multiply the score, and the narrower the pad the more
it pays. Come in faster and it is a hard landing or a crash, and a crash costs
100 fuel. When the tank empties, the game is over.

Headless autopilot, for a regression check or a machine with no display. It
flies a fixed seed and writes `shot.png` of its last frame:

```bash
CULEBRA_CANVAS_HEADLESS=1 culebra lunar_lander.cul demo 1200
# => score 50, fuel 723 after 1200 frames
```

Tests:

```bash
CULEBRA_CANVAS_HEADLESS=1 culebra test
```

## Layout

| File | Contents |
| --- | --- |
| `lunar_lander.cul` | the game: states, physics, HUD, frame loop, demo autopilot |
| `ship.cul` | the lander: hull, legs, throttle, fuel, hitbox, collision verdict |
| `terrain.cul` | midpoint-displacement ground, landing pads and their multipliers |
| `test_lander.cul` | checks on terrain generation, the pads, and each collision verdict |
| `assets/` | the original's two sound effects |

## What changed

| The original | Here |
| --- | --- |
| pgzero's `draw()` / `update(dt)` pair | one `Canvas.run` tick, with the real frame time replaced by a fixed 1/60 s |
| `clock.schedule(...)` for the pauses between lives | a frame deadline, since the loop runs at vsync |
| `keyboard.left` and friends | `Canvas.key("left")`, polled the same way |
| the `dylova` TTF, which the player had to download separately | Culebra's built-in 8x8 bitmap font, so there is no asset to fetch |
| `sounds.rocket_thrust.play(-1)` | the looping `Canvas.music` slot, put in step with the throttle once a frame |
| scanning every terrain point for every hitbox point | the same test against a topmost-ground-per-column lookup |

That last row is the only one that changes the shape of the code rather than
the API it calls. The original compares each of the ship's 38 hitbox points
against all ~1400 terrain points every frame. Only the highest terrain point in
a column can ever decide the test, so `terrain.cul` keeps one array of those
and `ship.cul` reads three columns per point. The verdict is identical, the
frame cost is not: a frame costs about 0.6 ms on the bytecode VM against a
16.7 ms budget, measured as the difference between headless `demo 2400` and
`demo 1200` wall clock (best of eight each, so interpreter startup cancels).

Two quirks of the original are kept deliberately, because they are visible in
play: the overlap check in `place_spots` indexes a pad's length by the number
of pads placed so far rather than by that pad's own length, and the HUD's
"vertical speed" reads as a rate of climb, so a descent shows negative.

Sound is the explosion as a one-shot `Canvas.Sound` and the rocket thrust
looping through the single `Canvas.music` slot. Both are silent in a headless
run.

## License

GPL-3.0, the same as the original, and it has to be. Culebra itself is MIT, but
that does not enter into it: this is a derivative work of GPL-3.0 sources, and
it ships the original's two `.ogg` files, so the port as a whole carries the
same license. `LICENSE` is the GPLv3 text taken from the original repository.

The original game is Copyright (C) aayala4.
