# `DrawTile` was never supposed to know about the window — `FSceneNode`'s sub-rect is the eye, and two shipping render devices prove it

**Researched:** 2026-09-11 (`/gr`, estate sweep) · **Project:** `unreal-gold-vr` · **Engine:** Unreal Engine 1 (OldUnreal 227k), our own D3D11 render device

## Why this matters here, specifically

The 2026-09-10 headset run left exactly one real driver defect: `DrawTile()` maps a tile's pixel rect
straight to full-window clip space, so the HUD, explosions and projectile sprites are drawn once
across the whole window instead of once per eye. The wearer's words: *"explosions were coming from the
sides and were off, so was the guns projectiles"*.

**This turns out not to need a new mechanism. The engine already has one, our device is ignoring it,
and two live open-source UE1 render devices show the correct shape.** There is also a cheap
in-house test that settles the one thing the research could not.

---

## 1. ⭐ The answer: `FSceneNode` carries a sub-rect, and that IS the eye

The UE1 render-device contract is public — the Unreal 226 Gold engine headers are vendored in
dpjudas's UT99VulkanDrv (`Thirdparty/Unreal_226_Gold/Engine/Inc/`). `UnRenDev.h` gives the entry
points: `DrawComplexSurface`, `DrawGouraudPolygon`, `DrawTile(FSceneNode*, FTextureInfo&, X, Y, XL,
YL, U, V, UL, VL, FSpanBuffer* Span, FLOAT Z, FPlane Color, FPlane Fog, DWORD PolyFlags)`,
`Draw3DLine`, `Draw2DLine`, `Draw2DPoint`, `Lock`, `Unlock`, `SetSceneNode(FSceneNode*)`
`[inferred-static 2026-09-11, read from the vendored headers]`.

**There is no eye or stereo concept anywhere in that interface.** Every call's only spatial context is
the `FSceneNode*`. And `UnRender.h` documents `FSceneNode` as *"a temporary object representing a
portion of the world view to render"*, carrying — in the engine's own comments — `INT X, Y` (frame
size), **`INT XB, YB` ("offset of top-left active viewport")**, `FLOAT FX, FY`, `FX2, FY2`
(half-size), `Viewport`, `Parent`/`Child`/`Sibling`, `Coords`/`Uncoords`, `Proj`/`RProj`,
**`Mirror` (±1.0)**, `NearClip`, `ViewSides[4]`, `ViewPlanes[4]`, `ComputeRenderSize()`,
`ComputeRenderCoords()`.

**So "which half am I in, inside `DrawTile`?" has an answer that was there all along:
`Frame->XB`, `Frame->YB`, `Frame->X`, `Frame->Y`.** This is the engine's own sub-rect mechanism, and
it exists because the engine already needed it — for editor panes, mirrors and warp-zone child frames.

### Two live devices, two idioms, same principle

- **XOpenGLDrv (Smirftsch / OldUnreal) and UT99VulkanDrv (Magnus Norddahl)** set the GPU viewport from
  the frame rect in `SetSceneNode` — XOpenGL does literally
  `glViewport(Frame->XB, Viewport->SizeY - Frame->Y - Frame->YB, Frame->X, Frame->Y)` — and build the
  frustum from `Frame->FX`/`FY` (`RFX2 = 2*RProjZ/Frame->FX`, `RFY2 = 2*RProjZ*Aspect/Frame->FY`).
  `DrawTile` then positions vertices as `RFX2 * Z * (X - Frame->FX2)` and
  `RFY2 * Z * (Y - Frame->FY2)` — **relative to the current frame's centre, inside the current frame's
  viewport. Never the window.** `[inferred-static 2026-09-11, read from both repos]`
- **D3D9DrvRTX (mmdanggg2, built on Chris Dohnal's D3D9Drv)** takes the other idiom: `DrawTile` adds
  the frame origin explicitly — `RPX1 = X + Frame->XB`, `RPY1 = Y + Frame->YB` — and emits two
  triangles at `Z = 0.5f`.

**⭐ The structural consequence for us, and it is the most valuable line in this topic:** a correctly
structured UE1 device never maps a tile to the *window*; it maps it to the *current* `FSceneNode` rect.
**If the stereo passes are driven by handing the engine a half-width `FSceneNode` per eye
(`XB = 0`, then `XB = SizeX/2`, with `X = SizeX/2`), `DrawTile` becomes correct for free**
`[hypothesis 2026-09-11 — the mechanism is read from source, its sufficiency for our device is not
tested]`.

⚠️ **And it flags a defect we had not noticed.** `Draw2DLine` and `Draw2DPoint` take the same
`FSceneNode*` and have the **identical problem**. Whatever we do for `DrawTile` must cover them, or
the same bug will survive in a less visible form.

---

## 2. ⭐ The `Z` argument looks like a free 2D-vs-3D discriminator — and one cheap test settles it

This matters because **the HUD and the explosions need opposite treatment**, and they may travel the
same path.

In dpjudas's **SurrealEngine** (an open UE1 reimplementation):

- `Render/RenderCanvas.cpp` draws **all** canvas / HUD / menu / font / console tiles with **`Z = 1.0f`**,
  in UI space scaled by `Canvas.uiscale`.
- `Render/VisibleCorona.cpp` takes a **world** position, transforms it by `WorldToView *
  ObjectToWorld`, perspective-divides to screen coordinates, and submits it **through `DrawTile` with
  the real depth**.
- `Render/VisibleSprite.cpp` draws `DT_Sprite` actors as **world-space camera-facing quads** via
  `DrawGouraudPolygon` — building `sideAxis = ViewRotation.YAxis * texwidth * drawscale` and
  `upAxis = ViewRotation.ZAxis * texheight * drawscale` around the actor's location.

So in *that* implementation sprites are 3D and coronas are screen-space-with-depth. ⚠️ **For the
retail/227 renderer this could not be proved either way** — `URender` is not public. Two things point
at sprites going through `DrawTile` there: (a) `DrawTile` carries a `FSpanBuffer* Span` **and** a real
`Z`, which only make sense for occluded in-world billboards, whereas the HUD passes `Span = NULL`,
`Z = 1`; and (b) **Han** (OldUnreal global moderator), in his rendering-redesign thread of
**2016-05-30/31**, proposes giving sprites *"dedicated interfaces separate from DrawTile"* — which only
reads as a change if they currently use it `[inferred-static 2026-09-11]`.

### The test, and it costs one launch we were going to do anyway

**Log the `Z` argument and whether `Span != NULL` for every `DrawTile` call, for one frame, with an
explosion on screen.**

- `Z == 1.0` and `Span == NULL` ⇒ **HUD / menu.** Wants one fixed depth for the whole bucket.
- `Z != 1.0` ⇒ **an in-world element already projected with that eye's transform.** It must be drawn
  *inside* each eye pass and **must not be replayed with a flat horizontal shift** — and replaying it
  with a shift is **exactly what produces "explosions coming from the sides"**.

⭐ **A second, free empirical check:** XOpenGLDrv ships a `NoDrawTile` debug switch (*"XOpenGL will not
render any tiles through the DrawTile API"*), documented in the OldUnreal v227 video-renderers wiki
page. Running stock Unreal with it on tells us immediately which on-screen elements travel that path,
with no code of ours involved.

---

## 3. The strongest fix is not a per-eye shift at all: make the 2D layer a real surface

**UT99 Quest (GhwstVR, 2026) is the only UE1-family project found that actually ships stereo**, and it
did not solve this by shifting tiles. Its own description: *"The 2D layer is a quad"* — *"they get
mapped onto a plane sitting in the world so they have real depth, and the controller pointer runs that
same mapping backwards to work out what you clicked"* — with menus *"on a panel a couple of metres
out, anchored where you opened them"* and the HUD horizon-locked `[reported 2026-09-11]`.

That is the same construct **OpenXR standardises as `XrCompositionLayerQuad`**, which the Khronos spec
describes as *"useful for user interface elements or 2D content rendered into the virtual world"*.

**Render the 2D once into a texture, then draw or submit that texture as a quad at a chosen distance.**
It is rendered once rather than replayed, it is automatically correct in both eyes, it is comfortable
by construction, and — relevant across this account — **it fixes the sibling XIII project's menu
problem with the same mechanism**.

⚠️ GhwstVR also names the cost they have not yet paid down, which is worth knowing before copying the
architecture: *"The engine currently does a full draw pass per eye where it only needs one per frame."*

---

## 4. ⚠️ What 227k does and does not give us, and a note on precedent

The **227k release notes** (OldUnreal wiki; release date still listed TBA) contain **nothing** about
stereo, eyes, or render-device interface extensions — only Canvas validity checks, crosshair scale and
texture types. 227's one tile-related addition of note is a **`WorldPosition` vector parameter on the
UnrealScript-side `Canvas.DrawTile`**, for drawing a tile as an in-level sprite — interesting, but
script-side, not device-side.

⭐ **And a finding worth recording for its own sake: our `VRGoldDrv` appears to be without public
precedent.** No public UE1 or UE2 *render device* doing stereo was found. UT99 Quest is an engine-level
ARM64/OpenXR rebuild, not a `URenderDevice`. DXU24 (Deus Ex) translates UE1 to UE5 at runtime rather
than patching a device. D3D9DrvRTX, XOpenGLDrv, UT99VulkanDrv and SurrealEngine are **all mono**. No
source, thread or commit describing per-eye state tracking inside a UE1/UE2 device was found at all.
The vorpX forum thread *"Unreal Engine 1 games working in Stereo 3D?"* (2016-04-11/12) reached no
conclusion.

That cuts both ways, honestly: there is nobody to copy, and also nobody who has already found the
reason it cannot work.

## 5. What this unlocks

1. **Reframe the ⭐⭐ row from "add an eye loop to `DrawTile`" to "drive the eye passes with a
   half-width `FSceneNode` and stop mapping tiles to the window"** — which is what the engine's own
   interface expects, and covers `Draw2DLine`/`Draw2DPoint` at the same time.
2. **One log line on the next launch classifies the buckets** (`Z`, `Span`), and stock Unreal with
   XOpenGLDrv's `NoDrawTile` classifies them with no code at all.
3. **The quad-layer design is the end state**, and it is shared with `XIII2003-vr` — one mechanism, two
   projects.

## Sources

All read online; no code copied. Full credit list in `CREDITS.md`.

- **Epic Games** — Unreal 226 Gold engine headers `UnRenDev.h`, `UnRender.h`, as vendored by
  **Magnus Norddahl (dpjudas)** in `UT99VulkanDrv/Thirdparty/Unreal_226_Gold/Engine/Inc/`.
- **Magnus Norddahl (dpjudas)** — *UT99VulkanDrv* (`UVulkanRenderDevice.cpp`) and *SurrealEngine*
  (`Render/RenderCanvas.cpp`, `Render/VisibleSprite.cpp`, `Render/VisibleCorona.cpp`). Both live.
- **Smirftsch / OldUnreal** — *XOpenGLDrv* (`Src/DrawTile.cpp`, `Src/XOpenGL.cpp`), and the OldUnreal
  wiki's *Unreal v227 Manual/Video renderers* page (`NoDrawTile`, `BufferTileQuads`) and
  *227 release notes / v227k*.
- **mmdanggg2**, building on **Chris Dohnal**'s D3D9Drv — *D3D9DrvRTX* (`Src/D3D9Render.cpp`,
  `Src/D3D9RenderDevice.cpp`), which replaced the whole `URender` subsystem (`renderSprite` →
  `renderSpriteGeo`, building a scaled, LookAt-oriented world quad at the actor's location) because
  RTX Remix needs ray-traceable world geometry and cannot use screen-space tiles. **The strongest
  existing template for promoting UE1 sprites out of the tile path into real geometry.**
- **Han** (OldUnreal) — *"[WIP] Concept for my Unreal Engine 1 Rendering Ambitions"*, 2016-05-30/31.
- **GhwstVR** — *UT99 Quest* (2026), for the quad 2D layer, the reverse-mapped controller pointer, and
  the honest per-eye-pass cost note.
- **Khronos OpenXR Working Group** — `XrCompositionLayerQuad`, OpenXR 1.1 registry.
- **jjensson** and **Ralf** — vorpX forum, *Unreal Engine 1 games working in Stereo 3D?*, 2016-04.

### Gaps this pass did not close

- **How the retail/227 `URender` submits `DT_Sprite` actors.** The subsystem is not public. We have the
  headers, a reimplementation that differs, a replacement that deliberately changed it, and Han's
  implication. **Not settled — log the `Z` argument.**
- **No public UE1/UE2 render device does stereo** (see §4), so there is no per-eye state-tracking
  implementation to copy.
- **No public documentation of UE2's `FRenderInterface` / `D3D8Drv` internals** — relevant to the
  sibling `XIII2003-vr`; searches restricted to the Unreal docs and the beyondunreal / unrealsp wikis
  returned nothing on the C++ render interface.
