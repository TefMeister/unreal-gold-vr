# Evidence — VRGoldDrv M2 stereo proven live, home PC, 2026-09-09

One launch, `/lm`, home PC (`RTX`). Game closed gracefully through its own console `exit`.

| file | what it shows |
| --- | --- |
| `01-startup.png` | the attract demo, first frame ever rendered by VRGoldDrv on this machine |
| `02-after-esc.png` | main menu, title bar reads **Version 227k** |
| `12-loaded.png` | `open Vortex2` — live gameplay, HUD present, renders normally |
| `13-mono-baseline.png` | the mono control, same spot, taken seconds before the stereo command |
| `15-stereo1.png` | **`VRGOLD STEREO 1`** — two squashed side-by-side eyes, HUD still full-width |
| `16-ipd20.png` | **`VRGOLD IPD 20`** — separation visibly ~7x wider |
| `17-swapeyes.png` | **`VRGOLD SWAPEYES 1`** — the halves exchange |
| `18-back-to-mono.png` | **`VRGOLD STEREO 0`** — reverts live, back to the baseline |
| `Unreal.log` | the driver's own account, flushed on exit |
| `ugdrive.py` | the throwaway harness used to drive it (window find / BitBlt capture / scancode input) |

## The numbers, so this does not rest on eyeballing

Best horizontal alignment shift between the left and right eye, per depth band:

| band | IPD 2.85 (default) | IPD 20 |
| --- | --- | --- |
| upper (near ceiling beams) | +2 px | +23 px |
| middle (far wall) | +1 px | +8 px |
| lower (floor + near pillar) | +5 px | +38 px |

Three things follow, and each rules out a different failure:
- **The halves are never byte-identical** ⇒ not a duplicated image.
- **The shift VARIES BY DEPTH** (+5 near vs +1 mid at default; +38 vs +8 at IPD 20) ⇒ real
  per-eye projection, not a flat offset of one rendered frame.
- **The shift scales with IPD** (~7x the IPD gives ~7.6x the near-band shift) ⇒ `StereoIPD`
  actually reaches the projection.

`SWAPEYES 1` checked the same way: the left half afterwards matches the *previous right* half
(mean abs diff **1.16**) far better than its own previous left half (**6.22**).

`STEREO 0` restores the baseline: mean abs diff **1.32** against the pre-stereo control, versus
a left/right half difference of **11.77** (an ordinary un-split frame) where the split frame
scored **3.24**.

`[verified-live 2026-09-09, n=1 launch]` for the whole sequence.

## What the log says in the driver's own words

```
Log:  Bound to VRGoldDrv.dll
Init: VRGoldDrv: Init 1280x720 colorbytes=4 fullscreen=0
Init: VRGoldDrv: D3D11 device + swap chain + tile/world pipelines ready (1280x720)
Init: VRGoldDrv: stereo config StereoMode=0 StereoIPD=2.850000 StereoSwapEyes=0 -> 1 eye(s)
Cmd:  vrgold stereo 1
Console: VRGoldDrv: StereoMode=1 -> 2 eye(s)
Exit: VRGoldDrv: Exit after 80156 frames (221 cached textures)
```

80,156 frames, no driver warning or error anywhere in the log, graceful shutdown. The two
`EntryIII.unr` warnings are stock Unreal Gold content and unrelated.

## ⚠️ One transcription fault of mine, recorded because the log shows it

`Cmd: vrgold ipd 2vrgold ipd 2.85` — the harness had no scancode for `.` and threw mid-command,
leaving `vrgold ipd 2` on the console line; the retry appended to it. The driver therefore ended
the run at `StereoIPD=2.000000`, not 2.85. **It changes nothing about the result** (every stereo
capture above was taken before that, at 2.85 or 20) and nothing persisted — the driver writes no
`[VRGoldDrv.VRGoldRenderDevice]` section back to `Unreal.ini`, so the next launch starts at the
compiled defaults again. Verified by reading the ini after exit.
