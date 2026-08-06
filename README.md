# Deadlock Mod: Velocity Direction Arrow

A Deadlock (Valve, Source 2) mod that draws a 3D arrow on your hero pointing
in the direction of your horizontal velocity, updated every server tick.
Built to make air-strafe angles visible instead of something you have to
mentally estimate.

## What's in this repo

| File | What it is |
|---|---|
| `velocity_arrow.vpulse` | The Pulse graph (visual-scripting) source. Edit this in [vpulse-editor](https://github.com/LionDoge/vpulse-editor). |
| `velocity_test.vmap` | The Hammer map source. Has a sealed room, an `info_team_spawn` per team (with **Initial player spawn** checked), and a `point_pulse` entity with its `graph_def` bound to the compiled `.vpulse_c`. |
| `velocityarrow.vpk` | The final packaged addon — this is what actually gets installed. |

## Architecture

No code injection, no Metamod, no LuaUnlocker. The whole mod is a **Pulse
graph attached to a `point_pulse` entity** in a normal custom map, compiled
through CSDK 12 like any other addon. This is the same mechanism community
maps like `jump_control` use.

The graph's logic, roughly:
- On `Think` (rescheduled every server tick, ~64Hz): read the player's
  current origin, diff it against the origin saved on the *previous* tick
  (`Save Variable`/`Load Variable`) to get a per-tick displacement vector —
  this stood in for `Get Entity Velocity`, which does not return usable data
  in this CSDK build.
- Draw `Debug World Arrow` from current origin to current-origin-plus-scaled-delta.
- Everything runs server-side (`Graph domain: ServerEntity`), so update rate
  is capped by the server's tick rate, not client FPS.

## Building from source

1. CSDK 12 install (`Reduced_CSDK_12`), with **Full Game Files** set up per
   [the CSDK 12 docs](https://deadlockmodding.pages.dev/modding-tools/csdk-12).
2. Put `velocity_arrow.vpulse` in `content/citadel_addons/velocityarrow/`,
   `velocity_test.vmap` in `content/citadel_addons/velocityarrow/maps/`.
3. Compile the Pulse graph (Pulse Editor's own **Compile** button, or Asset
   Browser → right-click → Compile — identical either way).
4. Compile the map with `GUIMapCompiler/CS2MapCompiler.exe`. For Custom Path,
   use the real `cs2.exe` (`steamapps/common/Counter-Strike Global Offensive/game/bin/win64/cs2.exe`)
   — **not** Deadlock's own `bin_cs2/deadlock.exe`. See gotchas below for why
   this matters.
5. Package via Asset Browser → **CS2 Workshop Manager** → New → Submit. The
   submission itself fails (error 15, expected — Deadlock's Workshop backend
   isn't open) but it generates the real addon package as a side effect:
   `game/citadel_addons/velocityarrow.vpk`.
6. Copy that file into the real Deadlock install's
   `game/citadel/addons/` folder, renamed to whatever `pakNN_dir.vpk` slot
   isn't already taken by Deadlock Mod Manager (check its files first —
   don't collide).

## Launching / testing

```
sv_cheats 1
developer 1
citadel_hero_testing_enabled true
map jump_control        # warm-up load, see gotchas
<swap hero via UI>
map velocity_test nomapvalidation=true
<swap hero via UI>
```

## Hard-won gotchas (read before you re-derive these the slow way)

- **Step 4 above is wrong on purpose, to flag it**: point Custom Path at the
  **real `cs2.exe`** (`steamapps/common/Counter-Strike Global Offensive/game/bin/win64/cs2.exe`),
  not Deadlock's own `bin_cs2/deadlock.exe`. The tool is literally called
  "CS2 Map Compiler" — using Deadlock's binary silently produces maps
  missing registration data the server needs to resolve the map name at all
  (`Spawn Server: <empty>` in the log otherwise).
- `map <name>` refuses custom maps by default — append **`nomapvalidation=true`**.
- The server still won't spawn you without an `info_team_spawn` that has
  **Initial player spawn** checked, matching your team.
- `Get Entity Velocity` does not return usable values in this CSDK build —
  use the origin-delta-via-`Save`/`Load Variable` approach in the graph instead.
- In the Pulse Editor's `Operation` node, type the literal symbol
  (**`-`**, `+`, `*`), not the word (`SUB`, `ADD`, `MUL`). Typing the word
  silently produces wrong results without erroring — this cost a lot of
  debugging time.
- `Save Variable` must fire **after** the value it's overwriting has been
  read elsewhere in the same tick, or you'll diff against stale data forever.
- Server tick rate is 64Hz. Set `Set Next Think`'s `dt` to `0.01`–`0.015625`
  and `Debug World Arrow`'s `flDuration` to roughly the same (not
  significantly higher, or you get overlapping "echo" arrows).
- Always **save to `content/citadel_addons/<addon>/...`** — saving anywhere
  else (e.g. a separate git repo checkout) silently decouples your edits
  from what actually gets compiled.

## On the earlier Metamod/LuaUnlocker plan

Early exploration considered Metamod:Source + LuaUnlocker for per-tick Lua
logic, since CSDK alone was assumed to have no scripting layer. That's not
needed — Pulse graphs cover it. The Metamod/LuaUnlocker install/uninstall
scripts and hash-verified modification log that used to live in this repo
have been removed, since they no longer describe anything the shipped mod
does. Before removing them, the real Deadlock install was directly
re-verified (not just checked against old docs) to contain zero trace of
`metamod.vdf`, `metamod_x64.vdf`, `metamod/`, or `LuaUnlocker/` anywhere
under the game folder — that path was fully explored, abandoned, and
confirmed cleanly backed out.
