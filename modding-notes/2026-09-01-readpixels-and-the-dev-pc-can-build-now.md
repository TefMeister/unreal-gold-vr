# 2026-09-01 — `ReadPixels` implemented, and the dev PC can finally build VRGoldDrv

**Date:** 2026-09-01, dev machine. **The game was never launched** (a parallel session owns the
machine's one "game may run" slot). Compile-verified only.

---

## 1. The dev PC was blocked on a missing SDK, and now isn't

`STATUS.md` wanted the dev PC to "grind polish items autonomously per the machine-roles preference",
but the OldUnreal 227k SDK only existed on the home PC (`C:\Users\TD3KX\ue1-sdk-227k`), so nothing
here could compile. Fixed:

* **SDK on the dev PC:** `C:\Users\Tefa\ue1-sdk-227k` — `OldUnreal-UnrealPatch227k-SDK.zip` from the
  **v227k_12** release of `OldUnreal/Unreal-testing`, mirroring the home PC's path and version.
* **Deliberately v227k_12, not the newer v227k_15** the 2026-08-24 research sweep flagged. The code
  was written against `_12`, and changing SDK version is a decision with consequences that belongs
  to the user, not a side effect of unblocking a build.
* **Toolchain:** CMake ships inside the VS2022 build tools here —
  `D:\VSBuildTools\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe`. It is **not on
  PATH**; prepend it. Then:

```
cmake -B build -A x64 -DUE1_SDK_ROOT=C:/Users/Tefa/ue1-sdk-227k
cmake --build build --config Release
```

**Verified: configures and builds clean on the dev PC**, MSVC 19.44, Windows SDK 10.0.26100.

Unreal Gold itself is installed here at `D:\nonSteam\UnrealGold` (`SYSTEM` 32-bit + `System64`,
stock `ICBINDx11Drv` present), so the dev PC now has game, SDK and toolchain.

## 2. `ReadPixels` was an empty stub

```cpp
void UVRGoldRenderDevice::ReadPixels( FColor* Pixels, UBOOL bGammaCorrectOutput )
{
}
```

Every screenshot taken on our render device returned whatever was already in the engine's buffer.

**Why this is a priority rather than a nicety.** The dev machine has no headset, so a screenshot is
its *only* verification channel — and every remaining M1 polish item (fog-brightness calibration vs
stock, blend modes, detail/macro textures, DXT) is a "does it look right" question. Without
`ReadPixels`, none of them can be checked on the machine that is supposed to be grinding them.

### Two details that would have produced a wrong-but-plausible image

* **Channel order.** `FColor` is `R,G,B,A` in memory — `Core/Inc/UnMath.h` says so explicitly:
  *"This is always RGBA, regardless of byte order! --ryan."* Our swap chain is
  `DXGI_FORMAT_B8G8R8A8_UNORM`, i.e. `B,G,R,A`. Without the swap, every screenshot comes out with
  red and blue exchanged — which looks exactly like a renderer bug and would send someone hunting one
  that does not exist. Checked against the SDK header rather than assumed.
* **Row pitch.** D3D11's `RowPitch` is not `SizeX*4`. Copied row by row.

Also handled: multisampled back buffers `ResolveSubresource` rather than `CopyResource`; the
destination is clamped to `SizeX`/`SizeY`, because the engine sizes its buffer from those and they
can lag the back buffer by a frame across a resize; and the staging texture is created per call
rather than cached, because a screenshot is rare and caching one would hold a full-size texture
resident permanently to save nothing measurable.

### ⚠️ One value in here is a guess, and is flagged as such

`bGammaCorrectOutput` applies a **fixed gamma of 2.0**. `[hypothesis]` — that constant is not
measured and not derived from the viewport's brightness setting. It is a plausible default, no more.

**Before anyone uses a gamma-corrected screenshot for the fog-brightness calibration specifically,
check it against stock `ICBINDx11Drv` output**, because that task is precisely a brightness
comparison and this constant would bias it. Uncorrected screenshots (the common path) are unaffected.

## 3. Status

`[compile-verified 2026-09-01]` — builds clean, MSVC, x64 Release.
`[untested]` — no screenshot has been taken through it; the game was not launched.

### Next, on this machine, no headset needed

1. Deploy `build/Release/VRGoldDrv.dll` to `System64`, launch, take a screenshot, confirm it is a
   real image and **that red and blue are the right way round** (a blue sky is the quick tell).
2. Then the polish queue becomes checkable here: fog brightness vs stock, scrolling-texture pans,
   detail/macro textures, DXT.

🤖 Static/compile work only; the game was not launched and no game file was modified.
