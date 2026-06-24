# Log — flight-sim-gcs

Append one entry per loop (format in CLAUDE.md §5). Newest at the bottom.

## 2026-06-24 — M0: fork from flight-sim2 + lean strip (GCS-first reset)

**Status**: GREEN (lean build verified; ready for GCS work)
**Files changed**: forked flight-sim2 → flight-sim-gcs; deleted 7 src modules + brand asset; new PRD/CLAUDE/README/Log; trimmed main.js/world.js/ui.js/hud.js/camera.js/index.html
**Tests**: 194 unit PASS · console-check 0 app errors · crosswind autoland PASS (110m)
**Decisions**:
- The flight-sim2 lineage drifted toward a game (cockpit models, multi-map, weather,
  multiplayer, scenario scoring). Reset to a **GCS-first** project: the MAVLink loop
  (telemetry up, missions/commands down, vehicle responds) is the product.
- **Stripped** (deleted module + all wiring): cockpit, weather, clouds, scenario,
  multiplayer, aiTraffic, drone; reduced world to a single map (plains) + single
  condition (day); removed map/condition pickers, cockpit camera mode, brand logo.
- **Kept intact**: 6-DOF physics, sensors, gated-KF nav, autopilot/missions, MAVLink
  telemetry (`telemetry.js`) + missionLink (SSE) + `bridge/` (HTTP↔MAVLink UDP),
  recorder, HITL, engineering/HILS console, damage/audio/effects. Determinism + the
  `window.__advance/__hils/setWind/injectFault/__engView` surface preserved.
- Clutter removed: docs/ (3.2MB), build/, examples/, PROMPT_ralph.md, CREDITS.md.
**Next**:
- M1: stand up the GCS loop end-to-end — `npm run bridge`, confirm telemetry in
  QGroundControl (UDP 14550) and a mission upload flies; harden the round-trip.
**Notes**:
- Run with GCS: `npm run bridge` → http://localhost:8765 + MAVLink :14550; start QGC.
- flight-sim2 is untouched and preserved.
