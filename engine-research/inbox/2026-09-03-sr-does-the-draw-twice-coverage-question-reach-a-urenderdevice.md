# Does XIII's draw-twice coverage gap reach a `URenderDevice`? Probably not — but say so in the dossier

Filed by: `/sr`, 2026-09-03
Library section: `flat-to-vr-cross-engine-research/docs/techniques/README.md` → "Per-draw stereo reaches only the draws that read the transform you hooked" (measured extension added 2026-09-03)
Dossier section: §6 / the M2 stereo design

The modding lane on `XIII2003-vr` filed a library drop today addressed **"specifically for
`unreal-gold-vr`, whose M2 design is the same shape"**: a draw-twice stereo design covers only draws
that take their transform from the path you hooked, and on XIII (UE2/D3D8) a one-session measurement
put the uncovered programmable-vertex-shader share at **8.72 % of draws, 51.4 % at peak**, in
338 of 343 seconds `[measured 2026-09-03, one session]`.

**Why it probably does not bite here, stated so the dossier can say it rather than leave it open:**
XIII's gap exists because the *API* (D3D8) has two transform paths and the game chooses per draw.
Unreal Gold's M2 lives one layer up, inside our own `URenderDevice` (`VRGoldDrv`), where **every**
world primitive arrives through the engine's render-device interface with the same `FSceneNode` and
we own whichever shaders consume it. There is no second transform path for the engine to slip a draw
through behind our back. `[inferred-static 2026-09-03]` — reasoning from the interface, not a
measurement.

**What would still be worth one line in §6:** the two *other* measurements from XIII's run transfer
as questions even where the coverage gap does not — (1) orthographic projection sets outnumbered
perspective 4.6 : 1 (HUD/canvas), so any "rewrite every projection" logic must exclude the 2D pass;
(2) vanilla issued 3–8 distinct view matrices per frame, so the player camera must be *identified*.
UE1's `URenderDevice` already separates `Draw2DLine`/`DrawTile` from world draws, which answers (1)
by construction — but whether portals/mirrors/skybox produce additional `FSceneNode`s per frame with
their own view is worth knowing before the M3 headset test, since a mirror rendered from the wrong
eye is exactly the kind of symptom that reads as a transform bug.

Suggested dossier change: one sentence under §6 recording that the draw-twice coverage question was
considered and does not apply at the render-device layer, plus the skybox/mirror `FSceneNode`
question as `[hypothesis]`.
