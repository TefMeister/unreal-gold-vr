# v227k_12 → v227k_15: what actually changed for fog/gamma, and does it matter to VRGoldDrv

**Status:** 🆕 new · **Priority:** low — informative, not actionable code-wise; relevant to the queued
fog-brightness calibration polish item.

## The question

This project's `external-research/INDEX.md` flagged, back on 2026-08-24, that OldUnreal's public SDK
is three releases ahead (v227k_15) of the v227k_12 this project builds against, citing fog and
D3D9Drv/OpenGLDrv fixes as possibly relevant. The dossier separately carries an open item: `ReadPixels`
applies a fixed **gamma 2.0** `[hypothesis]`, unmeasured, and must be checked against stock
`ICBINDx11Drv` before a gamma-corrected screenshot is trusted for fog-brightness calibration.
`VRGoldDrv` is written from scratch (no fork, per the project's standing no-fork rule) — so nothing
here is a "should we pull this code in" question, only "does the reference implementation's behavior
tell us anything about what our own renderer should do."

## Release-by-release changelog, v227k_12 → v227k_15 `[reported 2026-09-02]`

Read from the OldUnreal/Unreal-testing GitHub Releases page.

- **v227k_12 (30 Nov)** — this project's baseline. Two entries relevant here: "Most renderers will no
  longer apply gamma correction to screenshots that are not saved to disk (e.g. save-game previews)"
  — implying stock renderers **do** gamma-correct screenshots that *are* saved to disk, which is the
  case that matters for a fog-brightness comparison. Also ships `ICBINDx11Drv` for the first time.
- **v227k_13 (19 Jan)** — `ICBINDx11Drv` gains 227's distance-fog and custom Z-testing support, plus
  new texture types and an alpha-blend fix for unmasked tiles. No gamma changes.
- **v227k_14 (01 Mar)** — "several bugs in the volumetric fog rendering code" fixed, described as a
  minor regression-fix release. No gamma changes, and no detail on what the fog bugs were.
- **v227k_15 (16 Aug)** — the fog/detail-texture items the 2026-08-24 note flagged: OpenGLDrv distance
  fog now works as expected, D3D9Drv detail-texture flicker and fade-in/out fixed, D3D9Drv can render
  detail textures in fog zones again, plus unrelated XOpenGLDrv/OpenGLDrv translucency and BSP-flicker
  fixes. **No gamma-related entries in any of the three releases past v227k_12.**

**Verdict on the SDK-bump question:** nothing between v227k_12 and v227k_15 touches gamma or
screenshot handling, and every fog fix named is specific to `D3D9Drv`/`OpenGLDrv`/`XOpenGLDrv` code
paths this project doesn't use (`VRGoldDrv` is its own from-scratch D3D11 renderer, not a fork of any
of these). There is nothing here worth reading for technique before the fog-brightness pass — the
fixes are in reference renderers whose fog code this project never touched. The SDK-bump question from
2026-08-24 can be closed as "checked, not worth it for this reason" rather than left open.

## What IS relevant: gamma is not one fixed algorithm even in the reference renderers

This is the part that bears on `bGammaCorrectOutput`. `[reported 2026-09-02]`, from `ICBINDx11Drv`'s
own settings file (`ICBINDx11Drv_Settings.int`, metallicafan212):

- **`GammaMode`** is itself a *choice* between two named algorithms, **"XOpenGL"** and **"DX9"** — the
  reference D3D11 renderer doesn't have one canonical gamma curve, it picks which of two other
  renderers' gamma math to imitate.
- **`GammaOffset`** (plus per-channel `GammaOffsetRed/Green/Blue`) is a *brightness-slider offset*
  layered on top of whichever `GammaMode` curve is active — not itself the base curve.
- Separately, `GammaCorrectScreenshots` (a documented option from the 227 engine, not `ICBINDx11Drv`
  specifically) gates whether gamma correction is applied to screenshots at all, and the per-channel
  offsets above are **never** applied to screenshot gamma correction even when that flag is on —
  screenshot gamma and live-render gamma are two different code paths with different inputs.
- No source found states the actual exponent/curve either `GammaMode` option uses (not sRGB's 2.4-ish
  piecewise curve by name, not a bare "2.2" or "2.0" constant) — the 227 Release Notes PDF exists but
  is image/stream-heavy and did not yield extractable text on this pass; worth a follow-up if the exact
  curve is ever needed to *match* stock rather than just to know it's unmeasured.

**So `bGammaCorrectOutput`'s fixed-gamma-2.0 is not contradicted by anything found, but it is also not
corroborated** — the reference renderer treats "which gamma curve" as a user-facing *choice* between at
least two algorithms, and treats screenshot gamma as a separate code path from live-render gamma
entirely. That reframes the dossier's open item: the honest comparison isn't "does our gamma-2.0 match
stock's gamma," it's "which of stock's two `GammaMode`s (and whatever `GammaCorrectScreenshots` is set
to) was active in whatever screenshot the comparison uses" — the calibration test needs to record and
hold that setting fixed, or the A/B comparison is comparing against a moving target.

## Sources

- https://github.com/OldUnreal/Unreal-testing/releases — release-by-release changelog, v227k_12 through v227k_15
- https://github.com/metallicafan212/ICBINDx11Drv/blob/master/ICBINDx11Drv_Settings.int — `GammaMode`/`GammaOffset`/`GammaOffsetRed/Green/Blue` definitions (study material only, per the project's no-fork rule; nothing copied, only setting names and descriptions quoted)
- https://www.oldunreal.com/patch/unreal/oldunreal/227ReleaseNotes.pdf — the 227 release notes PDF; fetched but did not yield extractable gamma text on this pass (image/stream-heavy), flagged for a future targeted look if the exact curve is ever needed

## ✅ Outcome 2026-09-03 — the exponent came from the SDK on disk, not the web (folded from `inbox/`)

This topic said no public source states the curve either `GammaMode` uses, and advised recording
which mode stock had active before any A/B. Both were right — but the exponent did not need a public
source: **the 227k SDK on disk ships `ICBINDx11Drv`'s full source**, so it was a read, not a search.
`[inferred-static 2026-09-03]`

- `Gamma = Client->Brightness * 2.0` (`UnICBINDDx11Drv.cpp:2495`) — the **brightness slider**, not a
  constant.
- `GM_XOpenGL` (the default, enum 0) returns its input **unchanged when Gamma == 1.0**, and
  `Brightness` defaults to `0.5`, so **stock applies no gamma at all at defaults**.
- Our `bGammaCorrectOutput` was `pow(x, 1/2.0)`, mapping mid-grey 128 → 181. **`[disproved]`**, fixed,
  and covered by a new numeric test (2,070 checks, 0 failures).
- The protocol fix adopted verbatim, now a `[FLAT]` row: record stock's `GammaMode` **and**
  `Brightness` before comparing anything.
- ⚠️ Extra, found while reading: `ResScaling_PX.hlsl` calls
  `DX9Gamma(TexColor, GammaOffsetRed, GammaOffsetBlue, GammaOffsetGreen)` against a signature that
  applies its arguments in `(r, g, b)` order — **stock swaps the green and blue exponents in DX9
  mode.** Invisible while the per-channel offsets are equal (the default). `[inferred-static 2026-09-03]`

The "look at the 227 Release Notes PDF again" follow-up above is therefore **retired** — the SDK
source is the authority, exactly as the 2026-09-01 `ICBINDx11Drv` topic argued. Full write-up:
`modding-notes/2026-09-03-stock-gamma-settled-the-hardcoded-2-was-wrong.md`.
