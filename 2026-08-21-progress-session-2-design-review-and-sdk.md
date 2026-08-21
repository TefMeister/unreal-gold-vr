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

## Next

`DrawComplexSurface`: BSP world geometry, which needs the `FSceneNode` camera
transform and a depth buffer — the heart of flat rendering. Then
`DrawGouraudPolygon` (meshes) toward M1 flat parity.
