# SikkyDemoReview

A demo review addon for **Team Fortress 2** (64-bit). It adds instant seeking, a clickable timeline, a demo browser with match stats, damage numbers for STV demos, and a set of camera and HUD helpers for reviewing 6v6, highlander and ultiduo matches.

Everything lives in one folder inside `tf/custom`. No game files are modified and the addon only loads when TF2 is started with `-insecure`, so normal play and matchmaking are not affected.

---

## Installation

1. Copy the `SikkyDemoReview` folder to `Team Fortress 2/tf/custom/`.
2. Add `-insecure` to the TF2 launch options (Steam → TF2 → Properties → Launch Options).
3. Start the game. The console prints `SikkyDemoReview <version> loaded` and a line listing which features are working.

To play normally (for example on Valve or community servers with VAC), remove `-insecure`. The addon is then not loaded at all.

```
tf/custom/SikkyDemoReview/
├── addons/
│   ├── SikkyDemoReview.dll             the addon
│   ├── SikkyDemoReview.vdf             plugin loader entry
│   ├── SikkyDemoReview.log             log (useful after TF2 updates)
│   └── SikkyDemoReview_demos.cache     demo browser cache
└── cfg/
    └── sdr_demo.cfg                    demo-only settings, run when a demo starts
```

---

## Sikky Demo Viewer (demo browser)

Replaces the built-in Demo Playback window. Opens from the main menu demo button or with **Shift+F2**.

- Lists every `.dem` file anywhere under `tf/`, newest first.
- **Search** by map, player name or file name. Holding Backspace deletes continuously.
- **Filter tags** next to the search box: `6v6`, `9v9`, `ULTIDUO`, `STV`, `POV`. Tags of the same group combine with OR (6v6 + 9v9), tags of different groups with AND (6v6 + STV).
- Each tile shows the map, match format, final score, date, length, STV or POV, server and file name. Thumbnail colour: 6v6 blue, 9v9 gold, ultiduo purple.
- **Click a tile** to expand it: players per team with class icons (main class plus other classes played) and **K / A / D / DMG / DPM / HPM**, counted like logs.tf (round time only, no humiliation). DPM and HPM use each player's own time in the game, so substitutes are counted fairly.
- **WATCH DEMO** starts the demo (Enter also works). Double-clicking a tile does not start it.
- **DELETE** moves the demo, and its bookmark `.json`, to the Recycle Bin after a confirmation.
- Scrollbar on the right (drag it or click the track) plus mouse wheel. An opened tile scrolls fully into view, and when you come back from a demo the last watched one is shown.
- Match date is read from the file name (`2026-10-17_21-40-07`, `...-20220830-2120-...`) and remembered by file content, so renaming a demo keeps its date.

---

## Playback controls

These keys only work while a demo is playing.

| Key | Action |
|---|---|
| **Left / Right arrow** | Seek 10 s back / forward (instant, from keyframes) |
| **Down arrow** | Pause / resume |
| **Num - / Num +** | Playback speed −10% / +10% |
| **Num 0** | Jump to 5 s before the next frag of the spectated player (for a medic: also his next uber or drop) |
| **Space** | Cycle spectator mode |
| **1 – 9** | Spectate BLU players (scouts 1–2, soldiers 3–4, demo 5, medic 6; highlander up to 9) |
| **Q W E R T Y U I O** | Spectate RED players in the same order |
| **Ctrl** | Show / hide the mouse cursor (for the timeline). Right mouse button also hides it |
| **X** | Show / hide timeline markers (off by default, so the bar does not spoil the score) |
| **Z** | Stats panel |
| **C** | Damage taken by the spectated player |
| **V** | Auto camera |

At demo start the camera goes to the first BLU player in first person.

---

## Timeline

A bar at the bottom of the screen with the current time, speed and the key hints.

- **Click anywhere on the bar** to seek there.
- **Markers** (toggle with **X**):
  - **Round flags**: who won each round. Clicking one goes to 5 s before the capture and watches the capper in first person.
  - **Ubers**: always shown. Clicking one goes to 5 s before and watches the medic.
  - **Drops**: shown when spectating a medic. Clicking one goes to 5 s before and watches the killer.
  - **Kills and deaths** of the spectated player.
- Markers sit in separate lanes (flags, kills, ubers, deaths), so the one under the mouse is easy to hit.

---

## Stats panel (Z)

A scoreboard on the right side of the screen with class icons and **K / A / D / DMG / DPM / HPM** for every player, up to the current moment of the demo. It works after seeking too and uses the same rules as logs.tf.

## Damage taken (C)

A list next to the crosshair with the damage the spectated player takes and who dealt it. Each entry stays on screen for 3.5 s. Hits for 0 (ubered), a scout's own Boston Basher hits and their bleed are left out, and damage without an attacker is labelled *Fall Damage*.

## Auto camera (V)

Switches the camera automatically, in first person:

- **Medic drops** and **point captures** have priority: from 5 s before, it watches the player who kills the medic, or the capper.
- Otherwise it watches whoever gets the next kill, from 5 s before until 1 s after.
- Pressing a spectate key (1–9, Q–O) turns it off.

---

## Viewing improvements

Applied automatically while a demo plays:

- **Damage numbers and hit sounds** for the spectated player in STV demos.
- **Sniper lines**: a beam shows where each sniper is aiming.
- **Min viewmodels** forced on, with offsets filled in for weapons that lack them.
- **Viewmodel FOV**: 80, or 58 for the soldier's *Original* (aliases `sdr_vm_default` and `sdr_vm_original`).
- **Interpolation** set to exactly one snapshot interval of the demo, for the most faithful movement.
- **Objective HUD** lifted a little so it does not overlap the timeline.
- **Win panel** and spectator player list refreshed correctly after seeking.
- The Demo Playback window closes by itself when a demo starts.

`cfg/sdr_demo.cfg` is executed when a demo starts. It sets the arrow binds, crosshair and viewmodel options. Edit it to taste; it does not touch your normal config.

---

## Old demos

Demos recorded before the 2024 100-player update normally do not load in today's TF2 (`CreateDecoders failed`). While such a demo plays, the addon adjusts the client in memory and restores everything afterwards:

- Array sizes and table names that changed (`MAX_PLAYERS` 33 → 101, item attributes, player conditions).
- Item models that moved to `models/workshop/` (e.g. the Black Box) are loaded on demand.
- **2015-era STV demos** have no `player_hurt` events. DMG comes from the player resource instead, and damage numbers and damage taken are rebuilt from health changes (about 80% of hits shown exactly).
- The round counter dots are shown even when the demo does not send `mp_winlimit`.

Known limitation: official maps that Valve recompiled since (e.g. `cp_process_final`) can show wrong brushes in very old demos, such as missing doors or visible trigger boxes. Fixing that needs the old map files.

---

## After a TF2 update

The console shows `working: ...` and, if something broke, `NOT working (TF2 update?): ...`. Each feature is installed on its own, so one failing does not disable the others. Details are in `addons/SikkyDemoReview.log`. Most engine locations are found automatically at start-up; the few fixed interface indices are listed in `src/addon/gamevt.h`.

## Building

Cross-compiled for Windows x64 with llvm-mingw:

```
x86_64-w64-mingw32-clang++ -std=c++17 -O2 -shared -static -o SikkyDemoReview.dll \
  addon/sdr.cpp addon/hitevents.cpp addon/timeline.cpp addon/browser.cpp \
  core/demo.cpp core/keyframe.cpp core/index.cpp core/summary.cpp -lshell32
```

`tools/sdrtool.cpp` is a command-line helper for inspecting demos (summary, events, player resource values, keyframe jump verification).

## Credits

Sniper line-of-sight beams are based on the idea from Casting Essentials' SniperLOS (BSD-2).
