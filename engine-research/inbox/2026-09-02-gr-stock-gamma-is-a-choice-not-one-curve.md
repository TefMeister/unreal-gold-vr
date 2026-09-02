# Stock UE1 gamma is a choice between algorithms, not one fixed curve — reframes the `bGammaCorrectOutput` open item

**Date:** 2026-09-02 · **From:** `/gr` (external research) · **For:** this repo's modding session to
fold into `ENGINE-DOSSIER.md` (create-only inbox drop; fold in and delete)

**Bears on:** §11 (Dead ends & false leads), the `bGammaCorrectOutput` line: *"applies a **fixed
gamma 2.0** `[hypothesis]` — not measured. Check it against stock `ICBINDx11Drv` before using a
gamma-corrected screenshot for the fog-BRIGHTNESS calibration."*

## What's public `[reported 2026-09-02]`

`ICBINDx11Drv`'s own settings (`ICBINDx11Drv_Settings.int`) expose a **`GammaMode`** option that
picks between two named algorithms, **"XOpenGL"** and **"DX9"** — the reference D3D11 renderer has no
single canonical gamma curve, it imitates one of two other renderers' gamma math depending on this
setting. On top of whichever curve is active, `GammaOffset` (+ per-channel R/G/B variants) layers a
brightness-slider-style offset. Separately, the engine's own `GammaCorrectScreenshots` flag gates
whether gamma correction is applied to screenshots at all, and per-channel offsets are **never**
applied to screenshot gamma even when that flag is on — screenshot gamma and live-render gamma are
different code paths. No source found states the actual exponent either `GammaMode` uses. Full
write-up with the release-by-release SDK survey that produced this:
`external-research/topics/2026-09-02-sdk-227k15-vs-227k12-fog-gamma-changes.md`.

## Suggested dossier change

Don't drop the `[hypothesis]` tag on `bGammaCorrectOutput`'s gamma-2.0 — nothing here confirms or
contradicts it. But the comparison protocol needs a fix: before comparing a `VRGoldDrv` screenshot
against a stock `ICBINDx11Drv` one for fog-brightness, **record which `GammaMode` and
`GammaCorrectScreenshots` setting stock had active** — otherwise the A/B is against a moving target,
since stock itself doesn't have one fixed answer to "what gamma curve applies to a screenshot."
