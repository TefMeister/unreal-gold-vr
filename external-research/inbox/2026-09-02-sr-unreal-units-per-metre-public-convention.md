# The public Unreal-unit scale bounds the `StereoIPD=3.4` hypothesis — and says it cannot be settled from documentation

**Date:** 2026-09-02 · **From:** `/sr` (cross-engine research sweep) · **For:** this repo's `/gr` session to
curate into `external-research/` (create-only inbox drop; fold in and delete)

**Bears on:** `ENGINE-DOSSIER.md` §12 (open risks) — *"World scale / IPD units (new 2026-09-02):
`StereoIPD=3.4` assumes ~52.5 UU per metre `[hypothesis]`; M3 must measure it in the headset."* and
`modding-notes/2026-09-02-m2-stereo-proof-built-verified-deployed.md` §1.

## What is public

The Unreal Wiki's *Unreal Unit* page (preserved by Unreal Archive) says three things that matter here
`[reported 2026-09-02]`:

1. **"There is no fixed relation between Unreal Units and real-world units such as meter or inch."**
   That is the page's headline statement, not a caveat.
2. The common **level-design convention** is 16 UU ≈ 1 foot, i.e. **≈ 52.5 UU per metre** — the figure
   the M2 default was taken from. It is a mapping convention, not an engine constant.
3. For **Unreal and Unreal Tournament specifically**, the player collision cylinder is **78 UU tall**.
   Assuming a 1.75 m player, the page derives **≈ 44.6 UU per metre** for these two games — noticeably
   different from the convention. (Separately, several UT-era sources quote "1 UU = 2 cm", i.e. 50 UU/m,
   for the Unreal Tournament games.)

## What that does to the default

| Scale assumed | 3.4 UU is… | 64 mm would be… |
| --- | --- | --- |
| 52.5 UU/m (level-design convention, the M2 assumption) | 64.8 mm | 3.36 UU |
| 50 UU/m (UT "1 UU = 2 cm") | 68 mm | 3.2 UU |
| 44.6 UU/m (from Unreal's 78 UU player height) | 76 mm | 2.85 UU |

So the default is within roughly ±20 % of plausible whichever public figure is right, which is fine
for a proof of stereo and **not** fine for comfort — and the spread itself is the finding: **the
documentation cannot decide this, because the engine has no canonical scale.** The dossier's plan to
measure world scale in the headset at M3 is therefore the only route, not merely the preferred one.

## A cheaper first measurement, before the headset

The page's own advice is that scale follows **player movement metrics**, and the engine gives one for
free: the player collision height (78 UU for Unreal) against whatever eye height the M3 camera ends up
using. Comparing the in-headset apparent height of a doorway or a standing enemy against a 78 UU / 1.75 m
assumption is a one-glance calibration that needs no instrumentation.

## Sources

- Unreal Wiki (via Unreal Archive), *Unreal Unit* — the no-fixed-relation statement, the 16 UU/ft
  convention and the per-game collision table:
  https://unrealarchive.org/wikis/unreal-wiki/Unreal_Unit.html
- Unreal Wiki (via Unreal Archive), *Legacy: General Scale And Dimensions* — the same material in its
  older form: https://unrealarchive.org/wikis/unreal-wiki/Legacy:General_Scale_And_Dimensions.html
- BeyondUnreal wiki mirror, *Unreal Unit* — cross-check: https://wiki.beyondunreal.com/Unreal_Unit

The Unreal Wiki / Unreal Archive is credited in `flat-to-vr-cross-engine-research/ATTRIBUTION.md` as of
this sweep. Nothing was downloaded; pages read online only.
