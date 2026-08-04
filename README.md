# Deadlock Mod: Velocity Direction Arrow

## Goal
Build a small Deadlock (Valve, Source 2) mod that renders a 3D arrow on my
character model pointing in the direction of my horizontal velocity (ignore
Z/vertical) every tick.

**Stretch goal:** two more arrows offset ±90° from the main one for visual
reference.

## Environment
- Windows, Deadlock installed via Steam (app id `1422450`)
- Testing on a separate **alt Steam account**, not main — used specifically
  for this modding work to avoid any VAC/ban risk on the main account
- Steam owns Deadlock on this alt account

## Confirmed so far (don't re-research these)
- No official Valve SDK for Deadlock. Community fills the gap.
- **CSDK 12** (community "Source Development Kit") is the tool for compiling
  assets — models, particles, materials, maps, VPK packing. Setup involves:
  - downloading `Reduced_CSDK_12`
  - pulling full game files via DepotDownloader
    (`-app 1422450 -depot 1422451` and `-depot 1422456`, specific manifest
    IDs on the CSDK 12 page:
    https://deadlockmodding.pages.dev/modding-tools/csdk-12)
  - exporting/re-extracting the VPK via Source 2 Viewer
  - using `csdkcfg.exe` to create an addon under
    `content/citadel_addons/<name>/` (source) and
    `game/citadel_addons/<name>/` (compiled output)
- Binaries: `bin_server` mode is what actually launches Deadlock in-game
  with your addon mounted (requires Full Game Files); `bin`/`bin_tools` are
  for offline asset preview/compiling (each crashes on different things —
  Animgraph vs. projected particles).
- CSDK alone has no scripting/gameplay-logic layer — it's asset compiling
  only. Per-tick logic (reading velocity, computing an angle, updating an
  entity every frame) requires actual code running in the game process.
- Existing debug commands (e.g. `citadel_wall_detection_debug 1`,
  `cl_showpos 1`) are hardcoded C++ behavior baked into the binary — not
  reusable/repointable assets. Searched Deadlock's own `cvarlist` for
  generic `draw`/`debugoverlay` commands (331 "draw" results) but found
  nothing yet that's obviously a scriptable "draw a line/arrow from A to B"
  primitive — CS2's cvar list has `drawline`/`drawcross`/`debugoverlay_*`
  as engine-level commands, unconfirmed whether Deadlock exposes the same.
- The real path to per-tick logic: **Metamod:Source 2.0 + LuaUnlocker**.
  - Metamod:Source 2.0 (rolling pre-release, actively developed,
    https://github.com/alliedmodders/metamod-source) is a C++ engine-hook
    plugin loader with confirmed Deadlock support — its own build manifests
    reference `hl2sdk-deadlock` specifically (see build #1390: "Trigger
    build for hl2sdk-deadlock & dota update").
  - LuaUnlocker (https://github.com/Source2ZE/LuaUnlocker) is an
    open-source Metamod plugin that enables Lua VScript. This is almost
    certainly what's behind a Deadlock forum post (June 2024) showing a
    working velocity speedometer built this way — proof of concept exists,
    but no public source for that specific mod.
  - **Open question, needs verification before building against it:**
    LuaUnlocker's documented build command shows `configure.py -s cs2` —
    unclear if it has a Deadlock-specific build target or needs adapting.
    Check their GitHub issues/discussions for "Deadlock" before assuming.
  - Requires launching Deadlock with `-insecure` (disables VAC for that
    session — also generally blocks matchmaking/ranked while active).

## Risk / safety notes
- Only test with `-insecure` + private lobbies/offline, never bring an
  injected-DLL session into ranked matchmaking.
- Using the alt account specifically to fully decouple this from the main
  account.

## Roadmap

- [ ] **1. Dev workspace setup** — CSDK 12 install, Metamod:Source 2.0
      build/install for Deadlock, LuaUnlocker build on top of it. Verify
      each layer loads before moving on (Metamod loads → LuaUnlocker loads
      → a trivial Lua script runs).
- [ ] **2. Entity API discovery** — once Lua VScript execution is
      confirmed, find/inspect the entity API available (player origin,
      velocity, angle-setting, per-tick hook) — likely undocumented for
      Deadlock specifically; may need reflection/trial or cross-reference
      against Dota 2's VScript API (same engine lineage) as a starting
      point.
- [ ] **3. Mod logic**
  - Per-tick hook that reads player velocity, zeroes the Z component,
    computes yaw via `atan2`
  - Attach a visual (start simple — even a debug-draw line/axis if a
    scriptable primitive turns out to exist, otherwise a compiled
    particle/model attached to the player entity) and update its
    orientation each tick
  - Stretch: two more instances offset ±90° yaw from the main arrow
- [ ] **4. Packaging** — compile/package the CSDK addon alongside the
      script/plugin so the whole thing is testable in one `bin_server`
      launch.

## Working style notes
- Comfortable with Next.js/GitHub/Vercel-style web dev and some "vibe
  coding," but the C++/Source-engine-plugin space is new — go step by
  step, explain engine-specific quirks as they come up.
- Prefer getting a minimal end-to-end thing working (even something ugly)
  over a fully-featured first attempt.
