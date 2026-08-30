# 2026-08-21 — VRGoldDrv renders the 3D world (home PC)

M1, step 4 — the core of flat rendering. `DrawComplexSurface` (BSP with
lightmaps) and `DrawGouraudPolygon` (meshes, vertex-lit) now render through
our own pipeline with a real depth buffer. Verified: the menu's 3D flyby
background (the night map with its torch-lit stonework and the "Return to Na
Pali" logo overlay) renders in place of the teal diagnostic clear —
first build, first run.

## The projection insight (important for VR later)

UE1 has no projection matrix. `FTransform::Project()` in `UnRender.h` is the
whole camera model:

	pixel.x = ViewX * Proj.Z / ViewZ + FX15   (+ XB sub-rect origin)
	pixel.y = ViewY * Proj.Z / ViewZ + FY15   (+ YB)

with view space X right, Y **down**, Z forward. We fold this exactly into
clip coordinates over the full window (w = ViewZ):

	clip.x =  ViewX * (Proj.Z * 2 / WindowW) + ViewZ * ((FX15+XB) * 2 / WindowW - 1)
	clip.y = -ViewY * (Proj.Z * 2 / WindowH) + ViewZ * (1 - (FY15+YB) * 2 / WindowH)
	clip.z =  ViewZ * a + b        (near 1, far 65536; a = f/(f-n), b = -n·a)

The six constants live in one cbuffer updated per `SetSceneNode`. **These
constants are exactly what a per-eye VR projection replaces later** — that is
the entire stereo hook on the projection side.

## What the draw calls deliver (documented facts)

- `DrawComplexSurface`: `FSavedPoly` chains of `FTransform*` points in **view
  space**. Texture UVs come from the facet's `MapCoords` texture plane:
  `dU = (Point | XAxis) − (Origin | XAxis)`, then
  `u = (dU − Pan.X) / (UScale · USize)`. Lightmaps use the same dots with
  their own pan/scale **minus a half-texel bias** (`Pan − 0.5·Scale`) — the
  classic UE1 lightmap alignment.
- `DrawGouraudPolygon`: `FTransTexture` points carry view-space `Point`,
  texel `U/V`, and per-vertex `Light` (RGB 0–1).
- Lighting formula: `base × lightmap × 2` (UE1 overbrighting). A 1×1 50% gray
  stand-in makes unlit surfaces and vertex-lit meshes come out exactly
  `base × color` through the same shader.
- Masking: alpha-test (`clip(alpha − 0.004)`) in the world pixel shader —
  opaque conversions force alpha 255, masked P8 sets palette[0] to 0, so only
  masked texels clip. Depth: solid surfaces test+write; translucent/modulated
  test only.

## Evidence

- `screenshot-world.png` — menu flyby rendering in 3D (partially occluded by
  unrelated windows; the geometry, torch, and logo overlay are visible).
- `vrgold-world.log` — `VRGoldDrv: D3D11 device + swap chain + tile/world
  pipelines ready (1024x768)`; clean 20-second run.

## Known-untested / to verify in gameplay

- Lightmap alignment and texture panning signs (scrolling water/sky) need an
  in-game eyeball pass; the pan formulas are first-principles and may need
  the sign or scale adjusted.
- Mirror surfaces (`Frame->Mirror = -1`) unhandled; fog maps, detail/macro
  textures not yet drawn; DXT (`SupportsTC=0` keeps them decompressed to P8).
- Performance: one draw per facet/poly, no state sorting — fine on this
  machine, unmeasured elsewhere.

Next: in-game verification pass (load a map, walk around), fix what looks
wrong, then M1 polish (fog maps, detail textures, DXT support, screenshots
via ReadPixels) — after which M2 stereo can begin.
