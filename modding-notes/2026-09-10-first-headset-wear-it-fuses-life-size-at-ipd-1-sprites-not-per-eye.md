# 2026-09-10 — first headset wear: it fuses, life-size at IPD 1.0, sprites are not per eye

Home PC `RTX`, Quest 3 over **Virtual Desktop in half side-by-side 3D mode** (the driver
renders SBS to the monitor; VD turns it into stereo). Tefa driving, **two launches**, console
via Tab. Driver `VRGoldDrv.dll` 198,144 B (`01d7ccb7c30e`, v227k_15 SDK). Before the first
launch `Unreal.ini` `FullscreenViewportX/Y` was set 1024×768 → 3440×1440 so a fullscreen SBS
frame fills the monitor; backup `Unreal.ini.bak-2026-09-10-pre-vr`.

## Launch 1 — modes

`open Vortex2` → `VRGOLD STEREO 1` (windowed 1280×720) → Alt+Enter (`corrected client area
1296x759 -> 3440x1440`) → `VRGOLD STEREO 2` → `exit`. Two `VRGOLD IPD` attempts were typed and
**never reached the driver** (no `Cmd:` line).

> it fused, but the world felt stretched … the scale was off, world was too small but the gun
> in my hand was too big. VRGOLD IPD did nothing noticable and VRGOLD STEREO 1 made the world
> un-stretched and like it should be

- **It fuses.** `[verified-live 2026-09-10, n=1]`
- **Mode 2 (cropped) looks stretched under VD half-SBS, mode 1 (squashed) looks right.** VD
  assumes each half is squashed to half width and stretches it back; a cropped half gets
  stretched ×2. Expected by format, not a bug `[inferred-static]`. Mode 2 is kept for a real
  per-eye compositor later.
- Later in the same run: *"explosions were coming from the sides and were off, so was the
  guns projectiles"* — see the defect below.
- *"i did not check the gun with VRGOLD STEREO 1"* — the gun-size reading was mode 2 only.

## Launch 2 — scale, in mode 1

`vrgold stereo 1` → fullscreen → `vrgold ipd 1` → `vrgold ipd 0.7` → (one lost line,
`TeamSay gold ipd 1`) → `vrgold ipd 1` → `exit`. `ipd 1.5` never arrived.

> gun looked right in mode 1, world felt life-size at IPD 1

- **World life-size at `StereoIPD=1.0`**; too small at the shipped 2.85; not at 0.7.
  Bracket 0.7 / 1.0 / 2.85, 1.0 chosen `[reported 2026-09-10, n=1 wearer]`.
- **The gun is the right size in mode 1** — the launch-1 "too big" was the mode-2 stretch.
- 2.85 came from an assumed 44.6 units/m; 1.0 reading right implies ~15.6 units/m *under this
  viewer*. `[hypothesis]` — VD's SBS presentation is not a calibrated compositor.

## The defect: `DrawTile` has no eye loop

`VRGoldDrv.cpp:657` maps a tile's pixel rect straight to full-window clip space
(`X0 = (OrgX+X)/SizeX*2-1`) and draws it once. The world passes (`:383`, `:1332`) loop over
eyes; `DrawTile` does not. UE1 draws every world sprite — explosions, most projectiles,
coronas — and the HUD through `DrawTile` at mono-projected screen coordinates, so in stereo
each lands once, across both halves, at neither eye's position: exactly "from the sides and
off". Fix shape: when two eyes are active draw the tile twice, each into its own half
(x → x/2 + half offset), shifted per eye by `±HalfIPD × focal / Z` using the `Z` the engine
passes (world sprites carry depth; HUD tiles carry 1.0 and so sit at screen depth). Verify with
the 2026-09-09 harness on a Vortex2 firefight capture.

## Board after this

- Closed: the `[FLAT]` "mode 2 never exercised" row; the `[VR]` M3 world-scale row; the
  gun-size row.
- Open: `[PD] ⭐⭐` `DrawTile` per eye; `[PD] ⭐` ship `StereoIPD=1.0` as the default (settings
  do not persist — the engine rewrites the driver section on exit); the older flat rows.

## Console lesson, again

Three of six `VRGOLD` lines across the two launches were lost to the console not being open
(one surfaced as `Cmd: TeamSay gold ipd 1`, as on 2026-09-09). The driver prints nothing on
screen, so a lost line and an accepted one look identical from inside the headset. Read
`Unreal.log` after every wear before believing any "it did nothing".
