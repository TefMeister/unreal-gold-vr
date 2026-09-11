# Hand the engine a half-width `FSceneNode` per eye and `DrawTile` fixes itself — plus two defects and two free tests

**From:** `/gr` (estate sweep, 2026-09-11) · **For:** the modding lane, to fold into
`ENGINE-DOSSIER.md` — the render-device section, and the ⭐⭐ `DrawTile` row

**Full write-up:** [`external-research/topics/2026-09-11-fscenenodes-sub-rect-is-the-eye-and-drawtile-never-knew-about-the-window.md`](../../external-research/topics/2026-09-11-fscenenodes-sub-rect-is-the-eye-and-drawtile-never-knew-about-the-window.md)

## 1. ⭐ The row is solvable, and the mechanism was in the interface all along

The UE1 render-device contract is public — the Unreal 226 Gold headers are vendored in dpjudas's
UT99VulkanDrv. `UnRenDev.h` lists `DrawComplexSurface`, `DrawGouraudPolygon`, `DrawTile`, `Draw3DLine`,
`Draw2DLine`, `Draw2DPoint`, `Lock`, `Unlock`, `SetSceneNode` — and **there is no eye or stereo concept
anywhere in it.** Every call's only spatial context is the `FSceneNode*`.

`UnRender.h` documents `FSceneNode` as *“a temporary object representing a portion of the world view to
render”*, carrying (its own comments) `X, Y` frame size, **`XB, YB` = “offset of top-left active
viewport”**, `FX, FY`, `FX2, FY2` half-size, `Mirror` (±1.0), `NearClip`, `ViewPlanes[4]`,
`ComputeRenderSize()`, `ComputeRenderCoords()` `[inferred-static 2026-09-11]`.

**That sub-rect IS the eye.** It exists because the engine already needed it — editor panes, mirrors,
warp-zone child frames.

Two live devices, two idioms, same principle:

- **XOpenGLDrv** (Smirftsch/OldUnreal) and **UT99VulkanDrv** (Magnus Norddahl) set the GPU viewport from
  the frame rect in `SetSceneNode` — XOpenGL literally
  `glViewport(Frame->XB, Viewport->SizeY - Frame->Y - Frame->YB, Frame->X, Frame->Y)` — build the
  frustum from `Frame->FX`/`FY`, and position tile vertices as `RFX2 * Z * (X - Frame->FX2)` and
  `RFY2 * Z * (Y - Frame->FY2)`: **relative to the current frame's centre, inside the current frame's
  viewport. Never the window.**
- **D3D9DrvRTX** (mmdanggg2, on Chris Dohnal's D3D9Drv) adds the origin explicitly instead:
  `RPX1 = X + Frame->XB`, `RPY1 = Y + Frame->YB`, two triangles at `Z = 0.5f`.

**⇒ Suggested reframing of the ⭐⭐ row:** not *“add an eye loop to `DrawTile`”* but **“drive the eye
passes with a half-width `FSceneNode` per eye (`XB = 0`, then `XB = SizeX/2`, `X = SizeX/2`) and stop
mapping tiles to the window”** — at which point `DrawTile` is correct for free
`[hypothesis 2026-09-11; the mechanism is read from source, its sufficiency for our device is untested]`.

## 2. ⚠️ A defect that is NOT on the board

**`Draw2DLine` and `Draw2DPoint` take the same `FSceneNode*` and have the identical problem.** Whatever
we do for `DrawTile` must cover them, or the bug survives in a less visible form.

## 3. ⭐ Two free classification tests, and the reason it matters

The HUD and the explosions may travel the same path and need **opposite** treatment.

In **SurrealEngine** (dpjudas's open UE1 reimplementation): `RenderCanvas.cpp` draws **all** canvas /
HUD / menu / font / console tiles at **`Z = 1.0f`**; `VisibleCorona.cpp` takes a **world** position,
transforms and perspective-divides it, and submits it **through `DrawTile` with the real depth**;
`VisibleSprite.cpp` draws `DT_Sprite` actors as **world-space camera-facing quads** via
`DrawGouraudPolygon` `[inferred-static 2026-09-11]`.

⚠️ **For the retail/227 `URender` this is NOT settled** — the subsystem is not public. Two things
point at sprites using `DrawTile` there: `DrawTile` carries both a `FSpanBuffer* Span` and a real `Z`
(only meaningful for occluded in-world billboards; the HUD passes `Span = NULL`, `Z = 1`), and **Han**
(OldUnreal) proposed in 2016-05 giving sprites *“dedicated interfaces separate from DrawTile”* — which
only reads as a change if they currently use it.

- **Test A, one launch we were doing anyway:** log `Z` and `Span != NULL` per `DrawTile` call for one
  frame with an explosion on screen. `Z == 1.0` + `Span == NULL` ⇒ HUD/menu, wants one fixed depth.
  `Z != 1.0` ⇒ **an element already projected with that eye's transform**, which must be drawn inside
  each eye pass and **must not be replayed with a flat horizontal shift** — replaying it with a shift
  is exactly what produces *“explosions coming from the sides”*.
- **Test B, no code of ours at all:** XOpenGLDrv ships a **`NoDrawTile`** debug switch (*“XOpenGL will
  not render any tiles through the DrawTile API”*, OldUnreal v227 video-renderers wiki). Run stock
  Unreal with it and see which elements disappear.

## 4. The end state, and it is shared with `XIII2003-vr`

**UT99 Quest** (GhwstVR, 2026) — the only UE1-family project found that actually ships stereo — did not
solve this with per-eye shifts: *“The 2D layer is a quad”*, *“mapped onto a plane sitting in the world
so they have real depth, and the controller pointer runs that same mapping backwards to work out what
you clicked”*; menus *“on a panel a couple of metres out”*, HUD horizon-locked `[reported]`. OpenXR
standardises this as **`XrCompositionLayerQuad`**. Rendered once, automatically correct in both eyes,
comfortable by construction — and **it fixes the sibling XIII project's menu with the same mechanism.**
⚠️ GhwstVR names the unpaid cost: *“the engine currently does a full draw pass per eye where it only
needs one per frame.”*

**Template for promoting sprites out of the tile path:** `D3D9DrvRTX` replaced the whole `URender`
subsystem (`renderSprite` → `renderSpriteGeo`, a scaled LookAt-oriented world quad at the actor's
location) because RTX Remix needs ray-traceable geometry and cannot use screen-space tiles — the same
structural fix a stereo device needs, reached for a different reason.

## 5. ⚠️ Two things for the dossier's own record

- **227k release notes contain nothing about stereo, eyes, or render-device interface extensions** —
  only Canvas validity checks, crosshair scale, texture types. Its one tile-related addition is a
  `WorldPosition` parameter on the **UnrealScript-side** `Canvas.DrawTile` (script-side, not device-side).
- **No public UE1 or UE2 render device does stereo.** UT99 Quest is an engine-level ARM64/OpenXR rebuild,
  not a `URenderDevice`; DXU24 translates UE1 to UE5 at runtime; D3D9DrvRTX, XOpenGLDrv, UT99VulkanDrv
  and SurrealEngine are all mono. The vorpX thread *“Unreal Engine 1 games working in Stereo 3D?”*
  (2016-04) reached no conclusion. **So `VRGoldDrv` has no precedent to copy — and equally, nobody has
  published a reason it cannot work.** Worth stating in the dossier either way.
