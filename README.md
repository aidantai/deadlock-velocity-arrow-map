# Deadlock Mod: Velocity Direction Arrow

A [Deadlock](https://playdeadlock.com/) mod that draws real-time debug arrows on your hero showing horizontal velocity, acceleration, and strafe error — visual feedback for air-strafing, so you can see your movement vectors instead of guessing them.

## What's in this repo

| File | What it is |
|---|---|
| `velocity_arrow.vpulse` | The Pulse graph (visual-scripting) source. Edit in [vpulse-editor](https://github.com/LionDoge/vpulse-editor). |
| `velocity_test.vmap` | The Hammer map source — a sealed test room with a `point_pulse` entity running the graph. |
| `velocityarrow.vpk` | The packaged addon. Drop this straight into your `addons` folder to install. |

## How it works

A [Pulse](https://github.com/LionDoge/vpulse-editor) graph on a `point_pulse` entity runs server-side every tick and draws three arrows:

- **Velocity** — your current horizontal velocity, read directly.
- **Acceleration** — velocity minus last tick's velocity, showing how much and which way you're accelerating.
- **Error** — how far your current acceleration direction is from the mathematically optimal 90°-from-velocity strafe angle, rotated into a left/right "turn speed" gauge (too fast one way, too slow the other).

No code injection — it's a normal content addon, same mechanism as any other custom Deadlock map/mod.

## Installing

1. Rename `velocityarrow.vpk` to a `pakNN_dir.vpk` slot number not already used in your `addons` folder (check what's there first — Deadlock Mod Manager uses several by default).
2. Copy it into `<Deadlock>/game/citadel/addons/`.
3. Launch and load the map:
   ```
   sv_cheats 1
   developer 1
   citadel_hero_testing_enabled true
   map velocity_test nomapvalidation=true
   ```

## Building from source

1. [CSDK 12](https://deadlockmodding.pages.dev/modding-tools/csdk-12) installed, with Full Game Files set up.
2. Place `velocity_arrow.vpulse` and `velocity_test.vmap` under `content/citadel_addons/velocityarrow/` (map goes in the `maps/` subfolder).
3. Compile the Pulse graph in Pulse Editor or Asset Browser.
4. Compile the map with `GUIMapCompiler/CS2MapCompiler.exe`, Custom Path set to a real `cs2.exe` install.
5. Package via Asset Browser → CS2 Workshop Manager → New → Submit (the submission itself fails, but generates `velocityarrow.vpk` as a side effect).
