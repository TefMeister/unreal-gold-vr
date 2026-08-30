# Session 1 raw log — 2026-08-21 — INI rescue + recon

Raw working record; the readable summary is in `-modding-notes`
(`2026-08-21-progress-session-1-groundwork.md`).

## Install forensics

- Install: `D:\nonSteam\UnrealGold`, Unreal Gold 226b + OldUnreal 227k
  (`Manifest.ini`: `Version=226b` … `Version=227k`; log: `Version: 227
  Subversion: 11`, compiled 2026-08-15). Patch installed 15:51 local; first
  launch attempt 15:54.
- Crash signature in `SYSTEM\Unreal.log`:
  ```
  Warning: Failed to load "Class None.ini:Engine.Engine.GameEngine": Can't find ini:Engine.Engine.GameEngine in configuration file..
  Critical: appError called:
  Critical: Can't find ini:Engine.Engine.GameEngine in configuration file.
  Critical: Windows GetLastError: Not enough memory resources are available to process this command. (8)
  ```
  The GetLastError line is stale noise, not the cause.
- Evidence: `SYSTEM\Unreal.ini` was 1,042 bytes (vs ~30 KB `Default.ini`) —
  only `[Engine.Engine]` render/audio picks, `[URL]`, D3D wizard block,
  `[WindowPositions]`, `[FirstRun]`. `System64\Unreal.ini` was 32 bytes
  (just `[URL] AltLocalMap=UPack.unr`). Both preserved as
  `Unreal.ini.broken-2026-08-21` in their folders.
- `Detected.log` shows the wizard's renderer probe (D3DDrv detection) ran
  fine at 15:54:47 — the wipe is about what got *written*, not detection.
- Second-order failure: stale `Running.ini` (0 bytes, 15:55:14) from the
  crash made the next launch open as window title **"Unreal Recovery Mode"**
  (a small ~14 MB dialog process, not the game).

## Fix applied (verified working)

1. `Copy Default.ini → Unreal.ini` in both `SYSTEM\` and `System64\`.
2. Re-applied picks in both: `GameRenderDevice=ICBINDx11Drv.ICBINDx11RenderDevice`,
   `AudioDevice=SwFMOD.SwFMOD` (System64's default was XOpenGL; unified on
   ICBINDx11 for consistency).
3. `FirstRun=yes → FirstRun=227` (two pre-existing `[FirstRun]` sections in
   Default.ini — both set; an appended duplicate section was removed again).
4. Deleted stale `Running.ini` in both folders.
5. Relaunch: window title "Unreal", ~152 MB working set, log shows
   `Init: Unreal engine initialized`, `LoadMap: Entry`,
   `Bound to ICBINDx11Drv.dll`, HLSL shader compiles from
   `SYSTEM\ICBINDx11Drv\*.hlsl`, zero criticals.

Operational notes for future automation:
- `Unreal.log` is 0 bytes while the game runs (buffered); read via shared-read
  handle or after exit.
- Always delete stale `Running.ini` before an automated relaunch.

## Recon results

See `-engine-research/ENGINE-DOSSIER.md` (sections 2–4, 11) — renderer plugin
model, source availability (Unreal-PubSrc public UScript; ICBINDx11Drv MIT by
metallicafan212; no public C++ engine source), UT99 Quest prior art
(closed, Quest-only), UEVR not applicable to UE1.
