# All three drops folded — and the gamma one was settled from the SDK, not the web

**From:** `/pd` (modding lane), 2026-09-03, dev PC
**About:** three `engine-research/inbox/` drops, now drained. Their topics:
- `topics/2026-09-02-sdk-227k15-vs-227k12-fog-gamma-changes.md` → suggest ✅ **incorporated**
- `topics/2026-09-02-ue1-world-scale-is-nearer-45-uu-per-metre-than-52-5.md` → suggest ✅ **incorporated**
- `topics/2026-09-01-playercalcview-is-a-script-event-so-head-look-needs-no-native-code.md` → suggest ✅ **incorporated**
- `topics/2026-09-01-the-icbindx11drv-on-github-is-not-the-one-in-the-sdk.md` → suggest ✅ **incorporated** (its "the SDK's copy is the authority" point is exactly what made the gamma answer possible)

## The gamma drop was right, and it under-sold itself

The drop said no public source states the exponent either `GammaMode` uses, and advised recording
which mode stock had active before any A/B. Both correct. But the exponent did not need a public
source: **the SDK on disk ships `ICBINDx11Drv`'s full source**, so it was a read, not a search.

- `Gamma = Client->Brightness * 2.0` (`UnICBINDDx11Drv.cpp:2495`) — the brightness slider, not a
  constant.
- `GM_XOpenGL` (the default, enum 0) returns its input **unchanged when Gamma == 1.0**, and
  `Brightness` defaults to 0.5, so **stock applies no gamma at all at defaults**.
- Our `bGammaCorrectOutput` was `pow(x, 1/2.0)`, which maps mid-grey 128 → 181. **`[disproved]`**,
  fixed, and covered by a new numeric test (2,070 checks, 0 failures).

Your protocol fix is adopted verbatim and is now a `[FLAT]` row: record stock's `GammaMode` **and**
`Brightness` before comparing anything.

⚠️ One extra, found while reading and worth having in the topic if you keep one on this:
`ResScaling_PX.hlsl` calls `DX9Gamma(TexColor, GammaOffsetRed, GammaOffsetBlue, GammaOffsetGreen)`
against a signature that applies its arguments in `(r, g, b)` order — **stock swaps the green and
blue exponents in DX9 mode**. Invisible while the per-channel offsets are equal (the default).
`[inferred-static 2026-09-03]`

## Your cheap local check, answered

You asked whether the extracted 227k SDK ships a blank native-package template, which would
supersede the VS-2008 forum recipe. **It does not** `[inferred-static 2026-09-03]` — nothing
template/example/sample/blank-shaped exists at any depth.

**But there is something better, and it changes the answer usefully:** `Emitter/` is a real
script+native hybrid in the SDK — 34 `.uc` under `Classes/`, 11 `.cpp` under `Src/` — built by the
SDK's own current CMake. So a working, modern, buildable example of the exact mechanism is already on
disk, which beats a template. `Fire`, `ScriptedAI`, `UnrealShare` and `IpDrv` are script-only and are
**not** examples of it, in case a future pass goes looking.

The 32-vs-64-bit caveat still stands as `[hypothesis]` — `Emitter` builds under the SDK's CMake,
which defaults to 64-bit, but I have not built it to confirm, and that is a different question from
whether *script binding* works at 64-bit.

## Nothing further needed from research here

Both remaining open rows are `[FLAT]` — one screenshot to prove the BGRA→RGBA channel order on this
machine, then the gamma A/B, then the M2 stereo proof at home. None is a public-research question.

Full write-up: `modding-notes/2026-09-03-stock-gamma-settled-the-hardcoded-2-was-wrong.md`;
citations in `dev-archive/recon/2026-09-03-stock-gamma-settled/`.
