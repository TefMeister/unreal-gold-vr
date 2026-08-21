# Engine Dossier — Unreal Gold (Unreal Engine 1 / OldUnreal 227k)

> One consolidated, living reference for this game's engine, filled in as the
> `PLAYBOOK.md` phases are worked. Chronological blow-by-blow belongs in the
> `-dev-archive` / `-modding-notes` repos; this file is the *distilled current
> truth*. Update it whenever a fact changes; correct false leads in place.

**Status:** Phase 0 (ground truth and setup) · **VR-readiness verdict:** feasible — pluggable renderer + licensed source paths mean no injection is needed; prior art (UT99 Quest) proves native UE1 VR works.

## 1. Identity
- Game / build / version: **Unreal Gold**, base 226b upgraded to **OldUnreal patch 227k** (version 227, subversion 11, engine built 2026-08-15). Includes Return to Na Pali.
- Platform & store: non-Steam PC install at `D:\nonSteam\UnrealGold` (OldUnreal full installer over an owned copy). Not an unofficial port — 227 is an Epic-licensed maintenance patch.
- Legitimacy: owned copy confirmed.

## 2. Engine lineage
- Family / base engine: **Unreal Engine 1** (the original 1998 engine). OldUnreal 227k modernizes it: native **32-bit and 64-bit** Windows builds, new renderers, FMOD/OpenAL audio, PhysX option, while keeping the classic module layout (`Core`, `Engine`, `Render`, `Window`, `WinDrv` + pluggable render/audio device DLLs).
- Middleware in 227k: FMOD Ex (`SwFMOD`), OpenAL, PhysX (`PhysXPhysics.dll`), libxmp/mpg123 for music.
- Distinctive formats: `.u` (code), `.unr` (maps), `.utx` (textures), `.uax` (sounds), `.umx` (music), `.usm` (meshes); INIs follow the classic `Unreal.ini`/`User.ini` split with `Default.ini` as the pristine template.
- **Source availability (the big one):**
  - UnrealScript source for all of 227: public at [OldUnreal/Unreal-PubSrc](https://github.com/OldUnreal/Unreal-PubSrc).
  - Full C++ engine source: **not public** (OldUnreal builds from a licensed private repo).
  - C++ render devices: multiple open-source references — [ICBINDx11Drv](https://github.com/metallicafan212/ICBINDx11Drv) (MIT, D3D11), [XOpenGLDrv](https://github.com/OldUnreal/XOpenGLDrv), UT99VulkanDrv, FruitCompanyRenderer (Metal). These build against public UE1 SDK headers. **CONFIRMED (2026-08-21): the official 227k SDK is `OldUnreal-UnrealPatch227k-SDK.zip`, shipped as an asset of the [OldUnreal/Unreal-testing v227k_12 release](https://github.com/OldUnreal/Unreal-testing/releases/tag/v227k_12).** It contains C++ headers plus Windows `.lib` files for Core/Engine/Render (closed-source, link-only), full source for the community renderers (D3D9Drv, OpenGLDrv, XOpenGLDrv, ICBINDx11Drv — study material only under our no-copy rule), and a CMake build system targeting Visual Studio 2022 (32-bit via `-A Win32`, 64-bit by default; install prefix = the game's `System`/`System64` folder). The render-device contract is `Engine/Inc/UnRenDev.h`: `URenderDevice` with ~15 required virtuals (`Init`/`SetRes`/`Exit`/`Flush`, `Lock`/`Unlock`, `DrawComplexSurface`/`DrawGouraudPolygon`/`DrawTile`, `Draw2DLine`/`Draw2DPoint`, `ClearZ`, `PushHit`/`PopHit`, `GetStats`, `ReadPixels`) plus optional 227-era extensions (`SetSceneNode`, clip planes, and 227k's new `PushRenderToTexture`/`PopRenderToTexture`). Epic license: personal, non-commercial use only — compatible with this fan mod.

## 3. Binary & memory
- 32-bit (`SYSTEM\`) and 64-bit (`System64\`) native builds; desktop shortcut launches the 32-bit exe; **the 64-bit build is the intended VR host**.
- Renderer API: pluggable per-INI. Present in this install: `D3DDrv` (D3D7-era), `D3D9Drv`, `OpenGLDrv`, `XOpenGLDrv`, `ICBINDx11Drv` (D3D11), `GlideDrv`, `SoftDrv`. Currently configured: ICBINDx11 — verified working (DX11 shaders compile from `SYSTEM\ICBINDx11Drv\*.hlsl` at runtime, plain text on disk).
- Developer console: classic UE1 console (tilde). UnrealScript source being public makes console/exec plumbing fully readable.

## 4. DRM / anti-debug & injection foothold
- DRM: none.
- Injection: **not needed.** The engine loads any `URenderDevice` DLL named in `[Engine.Engine] GameRenderDevice=`. Our VR renderer is a first-class citizen, not a proxy.

## 5. Threading & frame structure
- UE1 is essentially single-threaded for game + render tick (227k details TBD — verify whether ICBINDx11Drv adds worker threads for decals/precache).
- One-frame walkthrough: TODO once we're inside our own renderer (`Lock` → draw calls (`DrawComplexSurface`, `DrawGouraudPolygon`, `DrawTile`) → `Unlock`/present is the classic URenderDevice contract).

## 6. Camera & projection delivery (the crucial section)
- **UE1 has no view matrix.** The renderer receives an `FSceneNode` per frame containing camera origin + `FCoords` (rotation basis) and FOV; geometry arrives pre-transformed or transformed via the scene node's coords. Stereo therefore comes from rendering the frame twice with per-eye camera origins/bases ("separation comes purely from where each eye sits" — confirmed approach from UT99 Quest).
- Head tracking: inject HMD orientation into the view. Options (to be decided in-fork): adjust the `FSceneNode` coords in the render device per eye, and/or drive `PlayerCalcView`/ViewRotation from the script/native side. XIII (UE2) precedent: overriding `PlayerCalcView` worked for head-look (FRotator = 65536 units/revolution — same convention in UE1; OpenVR yaw sign was negated in XIII, expect the same here).
- Projection: FOV comes from the engine per-viewport; per-eye asymmetric projection must be built from the VR runtime's eye tangents. TODO: study how the SDK's reference renderers derive their projection from `FSceneNode` (FOV/FX/FY), then design our own per-eye frustum path.

## 7. Constant-buffer fill mechanism
- Our own code (`VRGoldDrv`, written from scratch per the 2026-08-21 design revision — no fork), so no discovery needed — we control the cbuffers. Section repurposed: document our renderer's cbuffer layout for the per-eye transforms once written.

## 8. Pass inventory (by render target)
- TODO from inside our renderer. Known from the URenderDevice contract: world surfaces (BSP `DrawComplexSurface`), meshes (`DrawGouraudPolygon`), HUD/menus (`DrawTile` — 2D, must be redirected to an in-world plane for VR), plus 227k post-FX (resolution scaling shader seen in logs).

## 9. cvar / console cheat sheet
| command / cvar | effect | use |
|---|---|---|
| `TIMEDEMO 1` | fps counter | perf baseline |
| (to be filled as discovered) | | |

## 10. Autonomous harness recipe (this game)
- Launch: `Unreal.exe` (32-bit, `SYSTEM\`) or `System64\Unreal.exe`; log at `SYSTEM\Unreal.log` (buffered while running; open with shared read). `Unreal.exe map.unr?params LOG=file.log` conventions apply.
- Known trap: a crash leaves a stale `Running.ini`; the next launch then boots into an "Unreal Recovery Mode" dialog instead of the game. Delete `Running.ini` for clean automated relaunches.
- Frame capture: TODO (our renderer can write screenshots directly — we own the swap chain).

## 11. Dead ends & false leads (save future time)
- **2026-08-21, ini wipe:** the first-run wizard can leave `Unreal.ini` as a near-empty stub (only the wizard's own writes), which crashes every later launch with `Can't find ini:Engine.Engine.GameEngine`. The accompanying "Not enough memory resources" GetLastError is a red herring. Fix: restore `Unreal.ini` from `Default.ini` (both `SYSTEM\` and `System64\`), re-apply renderer/audio picks, set `FirstRun=227`.
- **UEVR does not apply** — it supports UE 4.8–5.4 only. UE1 needs the native-renderer approach.
- **UT99 Quest is not reusable** — closed source, Quest-only APK; it is proof of feasibility and an approach reference only.

## 12. Open risks toward the North Star
- ~~227k-compatible C++ SDK headers for building a render device~~ **RESOLVED 2026-08-21:** the official 227k SDK (see §2) ships the headers, `.lib` files, and CMake build; VS 2022 Build Tools + CMake confirmed present on the home PC.
- **New risk from the no-fork decision (2026-08-21):** writing the D3D11 renderer from scratch (rather than forking ICBINDx11Drv) raises the M1 cost — a full BSP/mesh/tile renderer must reach visual parity before VR work starts. Mitigation: the URenderDevice surface is small (~15 required virtuals), the SDK includes several complete renderers to *study*, and M1 only requires correct flat rendering, not feature parity (no post-FX, no advanced options).
- Decoupled weapon aim requires a script-side package plus a bridge to controller poses (227 native DLL binding) — mechanism confirmed to exist, details unproven.
- Two-pass stereo cost on a 1998 BSP renderer with 2026 post-FX: unknown; dev-PC numbers are non-diagnostic by standing rule (judge on the home rig).
- HUD/menu (`DrawTile`) redirection to an in-world plane: known-solvable (UT99 Quest did it) but non-trivial.
