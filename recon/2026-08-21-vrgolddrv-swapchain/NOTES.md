# 2026-08-21 — VRGoldDrv owns the screen: D3D11 swap chain + cleared frame (home PC)

M1, step 2. The from-scratch render device now creates a real D3D11 device and
a flip-model swap chain (`DXGI_SWAP_EFFECT_FLIP_DISCARD`, BGRA8, 2 buffers) on
the viewport window handle from `UViewport::GetWindow()`, clears the back
buffer every `Lock`, and presents every `Unlock(Blit=1)`. Draw calls remain
no-ops, so the visible output is a solid teal clear color — deliberately
unmistakable proof that the pixels on screen come from our device and nothing
else.

Evidence in this folder:

- `vrgold-swapchain.log` — engine log:
  `VRGoldDrv: Init 1024x768 colorbytes=4 fullscreen=1` followed by
  `VRGoldDrv: D3D11 device + swap chain ready (1024x768)`, then a normal boot.
- `screenshot-during-run.png` — desktop capture taken mid-run: the game
  window is a solid teal 1024×768 rectangle. (The rest of the desktop shows an
  unrelated Resident Evil 2 session that happened to be running.)

Run behaviour: ~17 seconds without crashing before the test harness killed the
process; INI restored to stock `ICBINDx11Drv` afterward.

Implementation notes (for the dossier later):

- `SetRes` calls `UViewport::ResizeViewport` first (`BLIT_Fullscreen |
  BLIT_HardwarePaint` when fullscreen, plain `BLIT_HardwarePaint` otherwise),
  then `IDXGISwapChain::ResizeBuffers` with the render target unbound and the
  RTV released (mandatory before ResizeBuffers).
- The swap chain always stays windowed; "fullscreen" is a screen-sized window
  (`DXGI_MWA_NO_ALT_ENTER` set). This sidesteps exclusive-mode headaches and is
  the friendliest mode for the VR mirror window later.
- Clear color: the engine's `ScreenClear` when `LOCKR_ClearScreen` is set,
  otherwise our debug teal (draws are still no-ops, so every frame must start
  from a known color).

Next: `DrawTile` — convert `FTextureInfo` textures to D3D11 SRVs and batch
textured quads, which makes the menus and HUD visible.
