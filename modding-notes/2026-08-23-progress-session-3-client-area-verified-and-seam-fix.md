# Progress session 3 (2026-08-23, home PC) — client-area fix verified; BSP seam glow fixed

Short home-PC verification session. Two M1-polish items closed, one launch gotcha
found and permanently fixed.

## 1. Client-area fix (staging `7e3f4d4`) deployed and USER-VERIFIED

- Deployed the built DLL over `C:\NonSteam\UnrealGold\System64\VRGoldDrv.dll`
  (old one kept beside it as `VRGoldDrv.dll.pre-clientfix-bak`).
- **User confirms menu clicks now land exactly on target.** The ~1cm offset from
  227's oversized client area (1040×807 handed back for a 1024×768 request →
  DXGI stretch) is gone. `AdjustWindowRectEx` + `SetWindowPos` forcing the
  client rect to the requested size is the confirmed correct fix.

## 2. Launch gotcha: the desktop shortcut launched the 32-BIT game

- First launch of the day popped the **"Unreal First-Time Configuration"
  wizard** (the dossier §11 INI-gutting wizard) listing only stock renderers.
  User cancelled it — correctly — before it wrote anything.
- Root cause: the desktop `Unreal Gold.lnk` pointed at `SYSTEM\Unreal.exe`
  (32-bit), whose separate `SYSTEM\Unreal.ini` is virgin (first-run never done,
  no VRGoldDrv — our DLL is x64 and lives in `System64`). The 64-bit INI was
  untouched and healthy the whole time (`FirstRun=227`, both render-device keys
  on `VRGoldDrv.VRGoldRenderDevice`).
- **Fix: shortcut retargeted to `System64\Unreal.exe`** (working dir
  `System64`). Diagnostic tell for next time: if the wizard appears, check
  WHICH System folder's `Unreal.ini`/`Unreal.log` just got written.

## 3. BSP T-junction seam glow — fixed (staging `c342cec`)

- User reported bright cyan vertical/horizontal hairlines at polygon edges,
  visible only at precise view angles ("took a minute of mouse-lining-up to
  screenshot, but noticeable in play").
- Diagnosis: the lines were **our own M1 diagnostic teal clear** bleeding
  through sub-pixel cracks at **UE1 BSP T-junctions** (a polygon corner meeting
  the middle of a neighbour's edge — e.g. a door-frame-split wall against an
  unsplit floor). Adjacent polygons rasterize independently; rounding along the
  shared edge can differ by a hair, leaving a 1px-wide gap that shows whatever
  was cleared underneath.
- Fix: under-draw clear color teal → **black** — the same mitigation every
  hardware UE1 renderer has used since 1998 (the cracks remain, invisibly).
  The teal "unrendered pixels glow" scaffold is retired with honours: it drew
  the splash/menu/world milestones AND surfaced this bug.
- **User-verified in game: lines gone.**

## State at session end

- Deployed DLL = staging `c342cec` (client-area fix + black clear).
- INI still on VRGoldDrv both keys; 32-bit `SYSTEM` install untouched/virgin.
- Remaining M1 polish (unchanged): fog-brightness calibration vs stock,
  scrolling-texture pan signs, detail/macro textures, DXT, ReadPixels
  screenshots. Then M2 stereo.
