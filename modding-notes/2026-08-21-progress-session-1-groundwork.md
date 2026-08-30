# Session 1 — 2026-08-21 — install rescue, feasibility research, groundwork

> **Superseded in part:** this entry describes the original plan to fork
> ICBINDx11Drv. On design review (see session 2) that changed — the render
> device is written from scratch; existing renderers are study material and
> prior art only. Kept as-is because this ledger records history.

## What happened

1. **Install fixed.** The fresh Unreal Gold + OldUnreal 227k install crashed on
   launch: the first-run wizard had left `Unreal.ini` (both `SYSTEM\` and
   `System64\`) as near-empty stubs, missing the entire
   `[Engine.Engine] GameEngine=` bootstrap → instant
   `Can't find ini:Engine.Engine.GameEngine` appError, and the crash left a
   stale `Running.ini` that pushed the next launch into the "Unreal Recovery
   Mode" dialog. Fix: restored both INIs from `Default.ini`, re-applied the
   wizard picks (ICBINDx11 renderer, FMOD audio), set `FirstRun=227`, deleted
   `Running.ini`. **Verified:** game boots, Entry map loads, DX11 shaders
   compile, no criticals. Broken INIs kept beside the fixed ones as
   `Unreal.ini.broken-2026-08-21`.

2. **Feasibility research.** Key findings (details in the engine-research
   dossier):
   - UE1 loads renderers as plugin DLLs (`GameRenderDevice=` in the INI) — a
     VR renderer can be a first-class plugin, **no injection needed**. A first
     for our projects.
   - [ICBINDx11Drv](https://github.com/metallicafan212/ICBINDx11Drv) (MIT, by
     metallicafan212) is a modern open-source D3D11 render device — and the
     exact renderer this install already runs. It becomes our fork base.
   - OldUnreal publish the 227 UnrealScript source publicly (Unreal-PubSrc);
     the full C++ engine source is not public, but we don't need it.
   - Prior art: **UT99 Quest** by GHWST — a native UE1 VR rebuild on Quest 3
     with decoupled aim. Closed source, Quest-only, so nothing reusable, but
     it proves the approach and confirms stereo-by-eye-position (UE1 has no
     view matrix). No PCVR UE1 conversion appears to exist — this would be new
     ground.

3. **Design approved** (see [2026-08-21-vr-design.md](2026-08-21-vr-design.md)):
   native VR render device forked from ICBINDx11Drv + UnrealScript companion
   package; SteamVR first, OpenXR second; Quest via Virtual Desktop on the
   home rig; milestones M1 (fork parity) → M6 (OpenXR).

4. **Groundwork done:** the standing five repos created and seeded
   (`unreal-gold-vr-{mod,dev-archive,modding-notes,staging,engine-research}`),
   local backup clones in place, dossier started, credits drafted,
   cross-machine STATUS.md updated.

## Next session

- Locate/confirm 227k-compatible C++ SDK headers for render devices (check
  what ICBINDx11Drv builds against and what 227k ships or OldUnreal provide).
- Clone the ICBINDx11Drv fork base into staging and get **M1 fork parity**
  building (64-bit).
