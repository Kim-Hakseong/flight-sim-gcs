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

## 2026-06-24 — M1: GCS loop verified end-to-end (telemetry up · mission/command down)

**Status**: GREEN (full MAVLink round-trip verified headlessly; ready for real QGC)
**Files changed**: tests/gcs-loop-check.mjs(new), tests/gcs-browser-check.mjs(new), bridge/server.mjs(cleanup), package.json(gcs-check script)
**Tests**: 194 unit PASS · gcs-loop-check 11/11 · gcs-browser-check 5/5
**Decisions**:
- Verified the whole GCS loop without needing QGroundControl, via two headless
  acceptance tests + a "fake GCS" UDP endpoint bound to the QGC port (14550):
  - **gcs-loop-check** (packet level): bridge HEARTBEAT (1 Hz); POST /telemetry →
    ATTITUDE/GLOBAL_POSITION_INT/VFR_HUD/GPS_RAW_INT with geodetically-sane values
    (sim local → lat/lon/alt); mission upload handshake (COUNT→REQUEST_INT→ITEM_INT
    →ACK); MISSION_START (CMD 300) → COMMAND_ACK + HEARTBEAT base_mode gains AUTO.
  - **gcs-browser-check** (full loop): serve the sim FROM the bridge → the browser
    missionLink receives the uploaded mission over SSE (autopilot len=2), engages on
    MISSION_START (phase=TAKEOFF), and the flying sim POSTs telemetry that the bridge
    relays back to the GCS (GLOBAL_POSITION_INT, 61 frames). QGC→bridge→sim→bridge→QGC.
- The bridge GCS implementation was already substantial; M1 hardened + proved it.
- Cleanup: removed the dead `/mp/state` endpoint (multiplayer was stripped in M0),
  fixed the boot banner name. Added `npm run gcs-check`.
- Harness bug found+fixed: a CDP `awaitPromise:true` on the `location.href=`
  navigation eval left the session on a stale context → mission appeared undelivered.
  Product loop was fine; the test was flaky.
**Next**:
- M2: command loop — ARM/DISARM, explicit MODE handling, NAV_TAKEOFF/NAV_LAND/RTL via
  COMMAND_LONG with proper COMMAND_ACK results; surface mode/arm state in the sim HUD.
**Notes**:
- Live with real QGC: `npm run bridge` → open http://localhost:8765 → start QGC
  (auto-connects UDP 14550) → telemetry shows; upload a mission + MISSION_START flies it.
- Headless re-verify: `npm run gcs-check`; browser loop: start Chrome with
  --remote-debugging-port=PORT then `node tests/gcs-browser-check.mjs PORT`.
