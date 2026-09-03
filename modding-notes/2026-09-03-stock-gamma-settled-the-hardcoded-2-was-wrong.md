# The screenshot gamma of 2.0 was wrong, and it would have corrupted the fog calibration

**2026-09-03, dev PC, `/pd` (parallel development).**
**The game was not launched. Nothing here has been run against Unreal Gold.**

The board carried one `[PD]` row: *settle the `bGammaCorrectOutput` fixed-gamma-2.0 `[hypothesis]`
against stock `ICBINDx11Drv` before any gamma-corrected screenshot feeds the fog-brightness
calibration — that task IS a brightness comparison, so a wrong gamma would corrupt it silently.*

Settled. It was wrong, and the row's own reasoning for caring was exactly right.

---

## 1. What stock actually does

The 227k SDK ships the community renderers' full source as study material, so this needed no launch
at all — just a careful read. `[inferred-static 2026-09-03]` Citations and formulas are in
`dev-archive/recon/2026-09-03-stock-gamma-settled/`; no third-party source is copied, per this
project's study-don't-copy rule.

- **The exponent is the brightness slider.** `Gamma = Viewport->GetOuterUClient()->Brightness * 2.0f`
  (`UnICBINDDx11Drv.cpp:2495`). Not a constant.
- **The default mode is `GM_XOpenGL`** (enum value 0; `Inc/GammaModes.h` defines only that and
  `GM_DX9 = 1`, with a third value commented out — so the `GM_PerObject` branch in the shader is
  dead code). Its curve returns the input **unchanged when `Gamma == 1.0`**, else raises it to
  `1/Gamma`.
- **`Brightness` defaults to 0.5**, so `Gamma` is exactly **1.0**, so **stock applies no gamma at all
  at default settings**. This machine's `Unreal.ini` line 194 reads `Brightness=0.500000`.

## 2. So our 2.0 was not a small error

Our `ReadPixels` applied `pow(x, 1/2.0)` unconditionally whenever the flag was set. Measured, not
estimated: that maps **mid-grey 128 → 181**, 64 → 128, 32 → 90. `[verified-numerically 2026-09-03]`

Compared against a stock screenshot taken at default brightness — where stock applies nothing at all
— our frame would have looked dramatically brighter. In a task whose entire purpose is deciding
whether our fog is too bright, that reads as "the fog is wrong" when the difference is our own
screenshot pipeline. The hypothesis tag on that line, and the instruction not to calibrate through
it, were doing real work.

## 3. The fix

`Inc/VRGoldGamma.h` — a pure header, no SDK types, so the DLL and the test compile the same code
(the pattern this project already uses for `VRGoldStereoMath.h`):

- `VRGoldGammaFromBrightness(b)` → `b * 2` — stock's own mapping.
- `VRGoldBuildGammaTable(Gamma, Table)` → the XOpenGL curve as a 256-entry lookup, identity at
  `Gamma == 1`, and identity again for non-positive or NaN gamma so a bad brightness can never
  produce a black screenshot.
- `ReadPixels` now reads `Viewport->GetOuterUClient()->Brightness` and falls back to 0.5 if the
  viewport or client is missing.

### Verified numerically

`Test/GammaTest.cpp` compiles the *shipped* header and checks it against ground truth constructed
independently here from stock's stated formula, in double precision with a different rounding
expression. **2,070 checks, 0 failures.** `[verified-numerically 2026-09-03]` The stereo test still
passes unchanged at 70,550 checks `[verified-numerically 2026-09-03]`, so nothing regressed.

**The test can fail** — four deliberate mutations of the header, each rebuilt and re-run:

| Mutation | Result |
|---|---|
| exponent inverted (`pow(In, Gamma)`) | **1,771 failures** |
| brightness scale `2.0` → `1.0` | **3 failures** |
| identity early-out removed | 0 failures |
| upper clamp to 255 removed | 0 failures |

The last two are honest negatives rather than gaps: `pow(x, 1/1)` is `x` anyway, and for inputs in
0..1 the curve can never exceed 1, so both of those lines are defensive rather than behavioural. The
two mutations that change actual behaviour are both caught.

⚠️ **A build-system trap worth recording**, because it made the first mutation run lie: restoring the
header with `Copy-Item` preserves the *source* file's timestamp, so MSBuild saw an older header than
its object file and skipped the rebuild — the test then reported on stale code. The first run of this
table was measured that way and was discarded. Every row above was produced with the header's
`LastWriteTime` forced to now before rebuilding. A test result is only evidence if the thing you
changed was actually rebuilt.

## 4. What is NOT established

- **That our screenshot now matches a stock screenshot byte for byte.** Stock applies its gamma
  inside the render pipeline and its `ReadPixels` sets the shader gamma to `1.0` for the readback
  pass (`UnHitTesting.cpp:159-171`) so the capture does not apply it twice; our pipeline has no gamma
  anywhere, so we apply it once at readback. Those land in the same place only if that reading of
  stock's readback is right, and that is an inference from source, not a measurement.
- **What settles it is one screenshot from each renderer at the same `Brightness`**, which is a
  `[FLAT]` step, not this one. At the default 0.5 both should now be the raw buffer, so they should
  agree; if they do not, the readback polarity above is the thing that is wrong.

### ⚠️ The A/B protocol needs two settings written down

`/gr` made this point and the source confirms it: **"stock gamma" is not one curve.** `GM_DX9` uses
a per-channel exponent `1/(1.25*(Gamma + offset))`, and `GammaOffset` shifts either mode. So before
any comparison, record stock's **`GammaMode`** and **`Brightness`**. Otherwise the A/B is against a
moving target.

### ⚠️ And a bug in stock, found on the way

`ResScaling_PX.hlsl` calls `DX9Gamma(TexColor, GammaOffsetRed, GammaOffsetBlue, GammaOffsetGreen)`
against a signature that applies its arguments in `(r, g, b)` order — **green and blue exponents are
swapped**. `[inferred-static 2026-09-03]` Invisible while the per-channel offsets are equal (the
default), visible the moment they are not. Not our bug, no action needed, but one more reason a stock
screenshot is only a reference once its gamma settings are recorded.

## 5. Two other drops folded in the same pass

- **World scale.** The public UE1 figure is **~44.6 UU/m** (13.6 UU/ft, from the 78-UU player
  collision height), not the 52.5 UU/m folk constant, which fits UT2003/4 `[reported 2026-09-02]`.
  So 64 mm ≈ **2.85 UU** and the old `StereoIPD=3.4` default was 15–25% wide — a slightly
  miniaturised world. **Default changed to 2.85.** Still `[reported]`; M3 measures it in the headset.
- **Head-look needs no C++.** The view comes from a script `event` in `PlayerPawn.uc` with three
  `out` parameters `[reported 2026-09-01]`, so HMD orientation is a script override writing
  `ViewRotation` (declared `transient norepnotify` — no replication cost, nothing persisted), while
  per-eye offset and projection stay in the render device. Two free consequences: **`WalkBob` is
  added inside that same event and only in first person, so head-bob removal is one omitted line**,
  and FOV is script-reachable. §6's "and/or" is resolved to *both, at different layers*.
- **`/gr` asked a question I could answer locally: does the SDK ship a blank native-package
  template?** **No** `[inferred-static 2026-09-03]` — nothing template/example/sample-shaped exists
  at any depth. **But `Emitter/` is a real script+native hybrid** (34 `.uc` under `Classes/`, 11
  `.cpp` under `Src/`) built by the SDK's own current CMake, which is a better reference than a
  template: a working, modern, buildable example of the exact mechanism. `Fire`, `ScriptedAI`,
  `UnrealShare` and `IpDrv` are script-only and are not examples of it.

## 6. Deployed

`VRGoldDrv.dll` is now in `D:\nonSteam\UnrealGold\System64\` on this machine for the first time
(there was none before, so nothing was overwritten and there is nothing to back up).

⚠️ **`Unreal.ini` was deliberately not touched** — it still selects
`ICBINDx11Drv.ICBINDx11RenderDevice`, so the game launches exactly as it does today. Switching
renderers is a game-config change and is the user's to make; the one line is named in the board row.

## Files

- `staging/unreal-gold-vr/VRGoldDrv/Inc/VRGoldGamma.h`, `Test/GammaTest.cpp`, `CMakeLists.txt`
  (both tests now build from one `foreach`).
- `dev-archive/recon/2026-09-03-stock-gamma-settled/` — the SDK citations and the test run.
