# 2026-09-02 — M2 stereo maths verified numerically, DLL built and deployed (home PC, static only)

**The game was NOT launched.** Nothing in this folder has been run inside Unreal. This is
compile- and numeric-verification evidence only.

## What is here

| File | What it is |
| --- | --- |
| `StereoMathTest-output.txt` | Output of `build/Release/StereoMathTest.exe` built from the committed source: `70550 checks, 0 failures`, exit 0. |

The test source is `staging/unreal-gold-vr/VRGoldDrv/Test/StereoMathTest.cpp`; the header it checks
is `staging/unreal-gold-vr/VRGoldDrv/Inc/VRGoldStereoMath.h` — the *same* header compiled into the
DLL, not a transcription of it.

## Why the passing result means something (mutation checks)

A test that cannot fail proves nothing, so three deliberate breaks were compiled into a scratch copy
of the header and run through the unchanged test:

| Mutation | Result |
| --- | --- |
| Eye-shift sign flipped (`Eye 0 → -h` instead of `+h`) | **32,080 failures** — the parallax-sign check and the per-pixel comparison both fire |
| Cropped mode uses the full frame width (`Wv = SizeX` for mode 2) | **16,000 failures** — every cropped-mode x pixel |
| Sub-rect origin `XB` dropped from the stereo `OffsetX` | **4,800 failures** — exactly the letterboxed frame case, x only |

The unmodified header: 0 failures. `[verified-numerically 2026-09-02]`

## What the DLL on disk is

`C:\NonSteam\UnrealGold\System64\VRGoldDrv.dll` is now the M2 build (SHA-256 prefix `89d00a20d7a4663e`,
202,752 bytes). The previous DLL (2026-08-23 12:13, 154,624 bytes = the `c342cec` seam-fix build the
user last played on) is kept beside it as `VRGoldDrv.dll.2026-09-02-pre-m2-stereo.bak`.
`Unreal.ini` was **not** touched; it already selects `VRGoldDrv.VRGoldRenderDevice` and has no
`[VRGoldDrv.VRGoldRenderDevice]` section, so the DLL starts in **mono** (StereoMode default 0) and
should render exactly as before until someone types `VRGOLD STEREO 1`.
