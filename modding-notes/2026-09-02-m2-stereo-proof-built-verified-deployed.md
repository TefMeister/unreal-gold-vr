# 2026-09-02 — M2 stereo proof: built, maths verified numerically, deployed — one launch from "proven"

**Date:** 2026-09-02, home PC, `/pd` lane. **The game was never launched** and nothing here has
been run inside Unreal. Everything below is `[compile-verified 2026-09-02]` or
`[verified-numerically 2026-09-02]` unless tagged otherwise.

---

## 1. What M2 is, and how it was done

The design doc's M2 is "side-by-side stereo on the flat monitor with per-eye camera offsets". It is
now implemented in `VRGoldDrv` (`staging/unreal-gold-vr/VRGoldDrv/`):

* **UE1 has no view matrix** (dossier §6), so a per-eye camera is a per-eye **translation of view
  space**: an eye sitting `h` units to the right of the centre camera sees every point `h` units
  further left. The vertex shader's constant buffer gained one float, `EyeShiftX`, added to
  view-space X before the (unchanged) M1 projection:
  `clip.x = (ViewX + EyeShiftX) * ScaleX + ViewZ * OffsetX`.
* **Two constant buffers, one per eye** (`ProjCB[2]`), filled in `SetSceneNode` from a pure-float
  header, `Inc/VRGoldStereoMath.h`, that has no SDK or D3D types on purpose so a standalone test can
  compile the identical code.
* **Every world batch draws twice** (`DrawWorldScratch`): once per eye, each through its own
  half-width `D3D11_VIEWPORT` with its own constant buffer, then the full-window viewport is
  restored. One shared depth buffer: the two viewports do not overlap, so the eyes never depth-test
  against each other.
* **The 2D layer (`DrawTile`: menus, HUD, console) stays full-window mono.** Deliberate: the mouse
  maps to the whole window, so a squashed/duplicated menu would be unclickable. Moving the HUD into
  the world is the design doc's M5, not M2.
* **Two layouts:** `StereoMode=1` *SBS squashed* — each eye's full frame scaled 2:1 into its half
  (the standard half-SBS 3D-TV layout); `StereoMode=2` *SBS cropped* — each eye's central half at
  natural aspect (nicer to look at on a monitor, loses side FOV).
* **Off by default.** `StereoMode=0` leaves the M1 path in place; the test asserts the mono
  constants are **bit-identical** to the pre-M2 formulas.
* **Convergence is deliberately not modelled.** Parallel cameras with identical projections put zero
  disparity at infinity, which is what a headset compositor wants (per-eye poses are translations;
  the asymmetric frustums come from the runtime's eye tangents at M3). On a flat 3D display this
  means everything sits "in front of the screen" — fine for a proof, not a display tuning.

### Config and console

Three config properties, `[VRGoldDrv.VRGoldRenderDevice]` in `Unreal.ini` (`CPF_Config`, category
`Options`), and a console interface:

| Setting / command | Meaning |
| --- | --- |
| `StereoMode=0/1/2` · `VRGOLD STEREO n` | mono / SBS squashed / SBS cropped |
| `StereoIPD=3.4` · `VRGOLD IPD x` | eye separation in Unreal units |
| `StereoSwapEyes=0/1` · `VRGOLD SWAPEYES n` | right eye into the left half (cross-eye viewing) |
| `VRGOLD STATUS` | prints size, mode, IPD, swap, eye count, frame |

Console changes apply on the next scene node and are **not** written to the INI. `GetStats`
(the `STAT` line) now also shows the mode and eye count.

**⚠️ `StereoIPD=3.4` is a `[hypothesis]`:** 64 mm at the commonly quoted UE1 scale of ~52.5 units
per metre. World scale is M3's job to measure; this only sets a default that should look roughly
right.

---

## 2. Verification — what was actually checked

### Numeric (§6 of the `/pd` rules: test the shipped code, against independent ground truth)

`Test/StereoMathTest.cpp` (built with `-DVRGOLD_BUILD_TESTS=ON`) compiles **the same header the DLL
compiles**, runs its constants through the exact arithmetic the GPU performs (vertex shader →
perspective divide → D3D11 viewport transform, in `float`), and compares the window pixel against
ground truth constructed independently:

* UE1's own projection, `pixel.x = X·Proj.Z/Z + FX15 + XB` (`FTransform::Project`, `UnRender.h`),
  evaluated in `double` for a camera translated to `x = ∓h`;
* the side-by-side layout stated geometrically (mode 1: `VpX + px/2`; mode 2: `VpX + (px − (cx − W/4))`).

Five frame shapes (1024×768 … 2560×1440, FOV 75°–110°, one letterboxed with `XB/YB ≠ 0`), three
IPDs including 0, both modes, both swap states, 200 random view points each, plus: mono constants
bitwise equal to M1; eye count for unknown modes = 1; viewports are two exact non-overlapping halves;
parallax **sign** (a finite point sits further right in the left eye's image) and **magnitude**
(`2h·Proj.Z/Z`, halved in squashed mode); zero IPD gives identical eyes.

**Result: 70,550 checks, 0 failures.** `[verified-numerically 2026-09-02]`

**The test can fail.** Three mutations of a scratch copy of the header were run through the
unchanged test: eye-shift sign flipped → 32,080 failures; cropped mode using the full frame width →
16,000; sub-rect origin dropped from the stereo offset → 4,800 (exactly the letterboxed case).
Evidence: `dev-archive/recon/2026-09-02-vrgolddrv-m2-stereo-math-verified/`.

### Build

`cmake -B build -A x64 -DUE1_SDK_ROOT=C:/Users/TD3KX/ue1-sdk-227k -DVRGOLD_BUILD_TESTS=ON` then
`cmake --build build --config Release`: configures and builds clean (VRGoldDrv.dll + StereoMathTest.exe),
no warnings. `[compile-verified 2026-09-02]`

### Deployed

`C:\NonSteam\UnrealGold\System64\VRGoldDrv.dll` = this build (SHA-256 `89d00a20d7a4663e…`,
202,752 bytes). Previous DLL kept as `VRGoldDrv.dll.2026-09-02-pre-m2-stereo.bak` (the 2026-08-23
`c342cec` build the user last played). **`Unreal.ini` was not touched**: it already selects
`VRGoldDrv.VRGoldRenderDevice` and has no `[VRGoldDrv.VRGoldRenderDevice]` section, so the game
starts in **mono**. Note this build also carries the dev PC's 2026-09-01 `ReadPixels`
implementation, which the home PC had not received yet.

---

## 3. What is NOT established

* **Nothing has rendered.** The per-eye draw loop, the viewport restore for the 2D layer, the
  config-property registration and the `Exec` parsing are compile-verified only.
* **Whether 227k's engine-side code is happy with per-batch viewport switching** — e.g. anything that
  reads back the viewport or draws 2D between world batches — is untested. If HUD elements appear in
  one half only, the 2D layer was drawn while an eye viewport was still set (a bug in *my* restore
  logic, not in the maths).
* **`SetSceneNode` is called once per frame by the engine** in this design; if 227k calls it per
  sub-scene (mirrors, portals, render-to-texture), each call recomputes both eyes, which is correct
  but was not measured for cost.
* **The IPD default and the sign of "right"** are unmeasured. Sign is covered by `SWAPEYES`.

### The one launch that settles it, and what each outcome means

Launch normally (the INI already points at `VRGoldDrv`). The game should look **exactly as before**
(mono). Then open the console and type:

```
VRGOLD STEREO 1
```

| What you see | Meaning |
| --- | --- |
| Two squashed copies of the world side by side, near objects visibly offset, **menu/HUD still full-width and clickable** | **M2 proven.** Try `VRGOLD STEREO 2` for the cropped layout, `VRGOLD IPD 10` to exaggerate the parallax, `VRGOLD SWAPEYES 1` if the depth looks inside-out when cross-viewed. |
| Two copies but **identical** (no offset even at `VRGOLD IPD 20`) | `EyeShiftX` is not reaching the shader: cbuffer layout mismatch between `FVRGoldProjection` and the HLSL `cbuffer` — a code bug, not a maths one. |
| World in the left half only, right half black | The second eye's viewport or constant buffer is not being bound (`ProjCB[1]` NULL / loop not reached). |
| HUD/menu confined to one half or doubled | My full-window viewport restore in `DrawWorldScratch` is not covering some 2D path. |
| Wrong geometry (skewed, offset by a constant) in stereo but correct in mono | The derivation is wrong in a way the test's ground truth shares — report the frame size and `VRGOLD STATUS` output; this would be the one outcome that indicts the maths. |
| Anything wrong **in mono** before typing the command | Not M2's doing — the only mono-path changes are the extra cbuffer float and per-batch `VSSetConstantBuffers`; suspect the build/deploy, restore the `.bak`. |

`VRGOLD STEREO 0` returns to mono live. Nothing needs the INI edited; to persist a setting add
`[VRGoldDrv.VRGoldRenderDevice]` / `StereoMode=1` to `System64\Unreal.ini`.

---

## 4. Files

* `staging/unreal-gold-vr/VRGoldDrv/Inc/VRGoldStereoMath.h` — new: per-eye viewport + constants.
* `staging/unreal-gold-vr/VRGoldDrv/Inc/VRGoldDrv.h` — `ProjCB[2]`, stereo config members, `Exec`,
  `UpdateEyeViewports`, `EyeShiftX` in `FVRGoldProjection`.
* `staging/unreal-gold-vr/VRGoldDrv/Src/VRGoldDrv.cpp` — shader cbuffer, `StaticConstructor` config
  properties, per-eye `SetSceneNode`, `Exec`, `UpdateEyeViewports`, two-eye `DrawWorldScratch`.
* `staging/unreal-gold-vr/VRGoldDrv/Test/StereoMathTest.cpp` + `CMakeLists.txt` option
  `VRGOLD_BUILD_TESTS`.
* `dev-archive/recon/2026-09-02-vrgolddrv-m2-stereo-math-verified/` — test output + mutation record.

(The `staging` files are committed by the parent session of this `/pd` run, not by this note's commit.)
