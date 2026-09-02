# UE1's world scale is nearer 45 UU per metre than 52.5 — the `StereoIPD=3.4` default is about 20 % too wide

**Status:** 🆕 new · **Priority:** low-medium — it only tunes a default the dossier already tags
`[hypothesis]`, but it is the number every comfort judgement in M3 will be measured against, and the
public figure is not the one the default assumed.

## What the community wikis actually say

The Unreal Wiki (mirrored at Unreal Archive) states the folk rule — "16 UU roughly correspond to
1 foot, or 52.5 UU to 1 meter" — and then immediately qualifies it: **"the actual scale differs for
the various games"**, giving `[reported 2026-09-02]`:

| Generation | UU per foot | UU per metre | player collision height |
| --- | --- | --- | --- |
| **Unreal / UT99 (UE1)** | **13.6** | **44.6** | **78 UU** |
| UT2003 / UT2004 (UE2) | 15.3 | 50.3 | 88 UU |

with the note that "the newer UT games are closest to" the 16-UU-per-foot rule. The companion
"General Scale and Dimensions" page for Unreal/UT99 gives the same 78-UU player (34 UU collision
radius), 83 UU minimum door height, 24 UU maximum step, and offers both `1 foot = 16 UU` and
"12 units/foot is more reasonable" — i.e. the community never settled on one figure for UE1, and the
44.6 UU/m row is the one derived from the player's actual height.

## What it means for this project

- `StereoIPD=3.4` was computed as 64 mm × 52.5 UU/m. At **44.6 UU/m the same IPD is 2.85 UU**;
  at the 12-UU/ft reading (39.4 UU/m) it is 2.5 UU. The default is therefore likely **15–25 % too
  wide**, which in a headset reads as a slightly miniaturised world — noticeable, not disabling.
- The dossier is right that M3 must **measure** the scale in the headset; this only says which
  number to start from and that the folk constant is the UE2 one.
- A measurement recipe that needs no headset: the player's own collision height is 78 UU by the
  engine's own definition; treat it as 1.75–1.80 m and the scale falls out at 43–45 UU/m. Any
  stereo comfort tuning should be reported alongside the scale assumed.

## Sources

- https://unrealarchive.org/wikis/unreal-wiki/Unreal_Unit.html — the per-generation table
- https://unrealarchive.org/wikis/unreal-wiki/Legacy:General_Scale_And_Dimensions.html — Unreal/UT99 dimensions
- https://ut99.org/viewtopic.php?t=13301 · https://ut99.org/viewtopic.php?t=2273 — community discussions of the same numbers
