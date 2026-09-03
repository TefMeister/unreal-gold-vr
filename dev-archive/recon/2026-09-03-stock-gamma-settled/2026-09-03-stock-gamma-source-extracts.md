# Stock 227k gamma, settled from the SDK sources

**2026-09-03, dev PC, `/pd`, static only. The game was not launched.**

Read from the OldUnreal 227k SDK on this machine (`C:\Users\Tefa\ue1-sdk-227k`), which ships the
community renderers' full source as study material. `ICBINDx11Drv` is MIT, by **metallicafan212**
(credited in `CREDITS.md`).

⚠️ **Citations and formulas only — no source is copied.** This project's standing rule is
study-don't-copy even for MIT code, so what follows is file:line references plus the arithmetic they
state, which is interface metadata. Anyone can re-read the originals at these locations.

## Where the exponent comes from

| Location | What it establishes |
|---|---|
| `ICBINDx11Drv/Src/UnICBINDDx11Drv.cpp:2495` | `Gamma` is assigned `Viewport->GetOuterUClient()->Brightness * 2.0f` — the exponent is driven by the **brightness slider**, not a constant. |
| `ICBINDx11Drv/Src/UnICBINDDx11Drv.cpp:2555` | The per-frame shader `Gamma` is that value plus `GammaOffset`, forced to `1.0` for ortho viewports. |
| `ICBINDx11Drv/Src/UnICBINDDx11Drv.cpp:2565-2567` | The DX9-mode per-channel exponents are `1 / (1.25 * (Gamma + GammaOffset<Channel>))`. Note the 1.25 factor — a different curve from the XOpenGL one. |
| `ICBINDx11Drv/Src/UnConfig.cpp:171-177` | Registers the config property: `AddByteProp(CPP_PROP(GammaMode), GM_XOpenGL, GEnum)` — **`GM_XOpenGL` is the default explicitly**, not merely by enum order. This install's `Unreal.ini` sets `GammaOffset*` but no `GammaMode`, so it takes that default. |
| `ICBINDx11Drv/Inc/GammaModes.h` | The mode enum is exactly two values: `GM_XOpenGL = 0`, `GM_DX9 = 1`. A third, `GM_PerObject`, is **commented out** — so the `GM_PerObject` branch in `ResScaling_PX.hlsl:34` is dead code. |
| `ICBINDx11Drv/Shaders/PostFX/ResScaling_Common.h` | `XOpenGLGamma`: returns the input unchanged when `Gamma == 1.0f`, else raises it to the power `1.0f / Gamma`. `DX9Gamma`: raises each channel to its own exponent. |
| `ICBINDx11Drv/Shaders/PostFX/ResScaling_PX.hlsl:40-47` | Selects between the two by `GammaMode`. |
| `ICBINDx11Drv/Src/UnHitTesting.cpp:159-171` | `ReadPixels` saves the frame gamma, sets it to `1.0f` while `bGammaCorrectOutput` is true, re-renders the back buffer to a normalised target, and restores it afterwards. |
| `ICBINDx11Drv/ICBINDx11Drv_Settings.int` | Exposes `GammaMode` ("XOpenGL" / "DX9"), `GammaOffset`, and per-channel `GammaOffsetRed/Green/Blue` (the per-channel ones labelled "DX9 mode only"). |

## The consequences

1. **The default is the identity.** `Brightness` defaults to 0.5, so `Gamma = 1.0`, so
   `XOpenGLGamma` returns its input unchanged. This machine's
   `D:\nonSteam\UnrealGold\System64\Unreal.ini` line 194 reads `Brightness=0.500000` — and so
   does **`Default.ini`** at the same line, which is what makes 0.5 the shipped default rather
   than just this install's setting.
2. **Our hardcoded 2.0 was therefore wrong**, and wrong in the direction that inflates a brightness
   comparison: it mapped mid-grey 128 to 181.
3. **"Stock gamma" is not one curve** — two modes, plus offsets. Any A/B against stock has to record
   which `GammaMode` and what `Brightness` were active, or it is measured against a moving target.

## ⚠️ A bug in stock, found while reading

`ResScaling_PX.hlsl:47` calls `DX9Gamma(TexColor, GammaOffsetRed, GammaOffsetBlue, GammaOffsetGreen)`
against the signature `DX9Gamma(float3 In, float r, float g, float b)`, which applies the exponents
in the order `(r, g, b)`. **The green and blue exponents are passed swapped.**
`[inferred-static 2026-09-03]`

Invisible while the per-channel offsets are equal, which is the default — visible the moment a user
sets them apart. It is not our bug and needs no action here, but it is one more reason a stock
screenshot is only a reference once its gamma settings are written down.

## Reproducing

Everything above is a plain read of files under `C:\Users\Tefa\ue1-sdk-227k`. Nothing was run, and
the SDK was not modified.
