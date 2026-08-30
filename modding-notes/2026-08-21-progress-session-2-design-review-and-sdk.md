# Session 2 (2026-08-21, home PC) — design reviewed, no-fork redirect, 227k SDK confirmed

## What happened

1. **The design doc was reviewed and approved by the project owner** — with one
   significant change: **we do not fork ICBINDx11Drv.** The project's standing
   "study, don't copy" policy wins even though the MIT license would permit a
   fork. The VR render device (`VRGoldDrv`) will be written from scratch against
   the official UE1/227k SDK headers, with ICBINDx11Drv (and the other renderers
   shipped in the SDK) used strictly as study material, credited as inspiration.
   A revision note was added to the top of the design doc; everything else in it
   stands. M1 is now: *our from-scratch render device builds (64-bit) and runs
   the game flat.*

2. **The home test rig is now set up for this project.** Unreal Gold was
   installed at `C:\NonSteam\UnrealGold` — OldUnreal 2.2.7.10 (227k), both
   32-bit and 64-bit System folders, stock ICBINDx11Drv rendering. All five
   project repos are cloned locally, and Visual Studio 2022 Build Tools with the
   C++ toolchain and CMake 3.31 are present, so this machine can build as well
   as run headset tests.

3. **The open SDK question is resolved.** The official 227k SDK ships as
   `OldUnreal-UnrealPatch227k-SDK.zip`, an asset of the
   [OldUnreal/Unreal-testing v227k_12 release](https://github.com/OldUnreal/Unreal-testing/releases/tag/v227k_12).
   It contains the C++ headers and Windows `.lib` files for Core/Engine/Render,
   full source for the community renderers, and a CMake build targeting VS 2022.
   The render-device contract is `Engine/Inc/UnRenDev.h`: `URenderDevice` with
   roughly 15 required virtual functions plus optional 227-era extensions
   (including 227k's new render-to-texture states). Epic's license on the core
   modules is personal/non-commercial — compatible with this fan mod. Details
   recorded in the engine dossier (§2, §12).

## Consequences of the no-fork decision

Writing the D3D11 renderer ourselves raises the cost of M1: we must reach
correct flat rendering (BSP surfaces, meshes, tiles/HUD) before any VR work
begins. In exchange, every line in the mod is ours, consistent with the policy
applied to every other project. The `URenderDevice` surface is small, the SDK
provides several complete renderers to learn from, and M1 needs correctness,
not feature parity.

## And then it happened: the scaffold works (same session)

`VRGoldDrv` was scaffolded in the staging repo (`VRGoldDrv/` — header,
implementation, standalone CMakeLists against the SDK), compiled and linked on
the first attempt (VS 2022, x64 Release), and passed a live load test:

```
Log: Bound to VRGoldDrv.dll
Init: VRGoldDrv: Init 1024x768 colorbytes=4 fullscreen=1
Init: Game engine initialized
```

The engine accepted our from-scratch device as `GameRenderDevice` and booted to
the menu root window with no crash over a 20-second run. The device draws
nothing yet, by design. Full log and incidental findings (including this
install's gutted-`Unreal.ini` stub, repaired per dossier §11) are in the
dev archive: `recon/2026-08-21-vrgolddrv-scaffold-loads/`.

## Later the same evening: the device owns the screen

The D3D11 step landed immediately after: `VRGoldDrv` now creates its own D3D11
device and flip-model swap chain on the viewport window, clears every frame,
and presents. With all draw calls still no-ops, the game window becomes a
solid teal rectangle — deliberately unmistakable proof that every pixel on
screen comes from our code. Verified by screenshot mid-run plus the engine
log (`VRGoldDrv: D3D11 device + swap chain ready (1024x768)`); evidence in the
dev archive, `recon/2026-08-21-vrgolddrv-swapchain/`.

## And still the same evening: the 2D layer renders

`DrawTile` landed next: embedded HLSL shaders, a dynamic vertex buffer, a
CacheID-keyed texture cache (P8 palettized with correct masking, BGRA8), and
the classic UE1 blend modes. The proof screenshot shows the 227k splash
screen — Epic MegaGames, GT Digital, Digital Extremes, the Unreal logo,
OpenAL, and PhysX logos — every pixel drawn by our from-scratch device,
masked cleanly over the teal diagnostic clear. Evidence:
`recon/2026-08-21-vrgolddrv-drawtile/` in the dev archive.

One evening took the device from "compiles" to "loads", "owns the screen",
and now "draws the game's 2D layer" — with zero borrowed lines.

## And before midnight: the 3D world renders

`DrawComplexSurface` (BSP with lightmaps) and `DrawGouraudPolygon` (meshes)
landed together, with a real depth buffer — and worked on the first build.
The menu's 3D flyby background now renders where the teal used to be.

The discovery worth sharing: **UE1 has no projection matrix.** The whole
camera model is one line in `UnRender.h` —
`pixel = View · Proj.Z / ViewZ + FX15` — and it folds exactly into six
clip-space constants in a single constant buffer, updated per `SetSceneNode`.
Making the future VR mod stereo, on the projection side, means nothing more
than swapping those six numbers per eye. Full derivation and the texture-
mapping formulas (facet plane dots, the lightmap half-texel bias, the ×2
overbright) are in the dev archive: `recon/2026-08-21-vrgolddrv-world/`.

## Late evening: the first human playtest — fine-tuning live with the owner

The project owner played Vortex Rikers and NyLeve's Falls on the from-scratch
renderer and fed back findings in real time; each was diagnosed, fixed,
rebuilt, and re-verified within minutes. All four fixes below are
**user-verified in-game**:

1. **Menu text was solid gold blocks** → 227 draws UI text with
   `PF_AlphaBlend` (glyph shapes live in the texture's alpha channel), and a
   premultiplied variant rides on `PF_Highlighted`. Added both blend modes and
   stopped forcing palette alpha opaque when those flags are present.
   *Verified: menus readable and clickable.*
2. **Scene darker than stock; no fog** → both were the same missing feature:
   **fog maps**, UE1's additive per-surface layer that is simultaneously the
   volumetric light glow and the haze. The engine only computes them (and
   coronas) when the render device claims `SupportsFogMaps` /
   `VolumetricLighting` / `Coronas`. Claimed them, sampled the fog map
   additively (same texture-plane mapping as lightmaps, own pan/scale,
   half-texel bias), added per-vertex fog for meshes.
   *Verified: the classic green Vortex Rikers corridor glow is back, and
   coronas render with their lens streaks.*
3. **Corpses/trees/gun/birds flickering psychedelic colors** → per-vertex
   `Fog` values on meshes are **only valid when the engine passes
   `PF_RenderFog`**; read without the flag they're stale memory that changes
   every frame. Gated the fog add on the flag.
   *Verified: meshes normally lit, no flicker.*
4. **Menu buttons needed the mouse ~1 cm above their visuals** → the log
   caught it: 227 handed us a window whose **client area was 1040×807 for a
   1024×768 request** (exactly one window-frame larger), so DXGI stretched
   the back buffer and visuals sat up to ~39 px below the engine's hit-test
   coordinates. Fix: force the client area to exactly the requested size
   after every resize (`AdjustWindowRectEx` + `SetWindowPos`).
   *Fix built; verification pending the next restart.*

Also user-confirmed in passing: the skybox and moon render correctly (sky
zones exercise `SetSceneNode` + `ClearZ` between passes), terrain and rock
lightmaps look right, and the game is playable start of Vortex Rikers through
to the NyLeve exterior with quicksave/load working.

## Next

Verify the client-area fix (menu clicks should be pixel-accurate). Then
remaining M1 polish: fog-brightness calibration against stock, scrolling-
texture pan signs, detail/macro textures, DXT formats, `ReadPixels`
screenshots — and on to M2 stereo.
