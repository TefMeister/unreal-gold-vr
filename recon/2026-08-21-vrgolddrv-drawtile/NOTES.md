# 2026-08-21 — VRGoldDrv renders the 2D layer: DrawTile works (home PC)

M1, step 3. The from-scratch render device now draws the engine's 2D layer —
`DrawTile` handles menus, HUD, fonts, console, and splash graphics — through
our own pipeline:

- Embedded HLSL (VS passes clip-space quads through; PS samples × vertex
  color), compiled at Init with `D3DCompile`.
- A dynamic vertex buffer (one quad per call for now — batching later).
- A texture cache keyed by `(CacheID << 1) | masked`, converting **TEXF_P8**
  (palette → RGBA8; palette index 0 forced transparent when `PF_Masked`) and
  **TEXF_BGRA8/BGRA8_LM** (swizzled). `FColor`'s memory order is R,G,B,A, so
  palettes copy straight into `DXGI_FORMAT_R8G8B8A8_UNORM`. Unsupported
  formats get a magenta placeholder and a log line.
- Classic UE1 blend modes: masked → alpha blend, `PF_Translucent` →
  `ONE / INV_SRC_COLOR`, `PF_Modulated` → `DEST_COLOR / SRC_COLOR` (vertex
  color forced white), otherwise opaque. `PF_NoSmooth` selects a point
  sampler. Depth off, cull off for the 2D layer.
- Realtime textures re-upload when `bRealtimeChanged` is set; `Flush` empties
  the cache.

Evidence in this folder:

- `screenshot-menu.png` — mid-run desktop capture: the 227k splash screen
  (Epic MegaGames, GT Digital, Digital Extremes, the Unreal logo, OpenAL,
  PhysX logos) rendered by our device, masked correctly over our teal clear.
- `vrgold-drawtile.log` — engine log:
  `VRGoldDrv: D3D11 device + swap chain + tile pipeline ready (1024x768)`.

Tile geometry notes: `DrawTile` rects are viewport pixels; add the
`FSceneNode::XB/YB` sub-rect origin, then map to clip space. UVs are texel
values scaled by `1 / (UScale * USize)` per axis.

Run behaviour: 18 seconds without crashing before the harness killed it; INI
restored to stock ICBINDx11Drv afterward.

Next: `DrawComplexSurface` (BSP world geometry — needs the FSceneNode
camera transform + a depth buffer) and `DrawGouraudPolygon` (meshes), toward
M1 flat parity.
