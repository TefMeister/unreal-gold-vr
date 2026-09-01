# Research index

**Last `/gr` pass: 2026-09-01 (second pass, estate sweep) — FULL.** Inbox empty. **The first two
topics in this repo**, both from the dossier's own open items: §6's head-tracking question is
answered — `PlayerCalcView` is a plain **script event** in 227's public source, so head-look and
head-bob removal need **no C++ at all**, and §12's native-DLL bridge narrows to one job (publishing
the HMD pose), with a documented `ucc make -nobind` procedure and two caveats. The second topic is a
trap worth knowing: the `ICBINDx11Drv` on GitHub builds for **UT 469**, not our 227, so the SDK's
copy is the one to study. §5's threading TODO has no public answer and should not be searched again.
_Earlier the same day — CHECK-IN, and still true:_ §6's `FSceneNode` projection TODO has no public
write-up (it is a reading task against the SDK on disk, not a research task), and **UE1 VR prior art
is still none** `[checked 2026-09-01]` — every path leads back to UEVR, which is 4.8–5.x only, so
§11 and §12's greenfield assumption both stand.

Every research topic gathered for this project, newest first. Each row links to a self-contained
write-up in `topics/`. Status tags:

- 🆕 **new** — found, not yet acted on by the modding side.
- 👀 **reviewed** — a modding session has read it and factored it into a decision, but nothing shipped from it yet.
- ✅ **incorporated** — directly led to a real change (code, a test, a note) in one of the other five repos; linked below.
- ❌ **dead end** — checked out, didn't pan out; kept for the record so it isn't re-investigated from scratch.

| Date | Topic | Status | Summary |
| --- | --- | --- | --- |

| 2026-09-01 | [Head-look needs no C++: `PlayerCalcView` is a script event](topics/2026-09-01-playercalcview-is-a-script-event-so-head-look-needs-no-native-code.md) | 🆕 new | Answers §6's open "script side and/or render device?" question: **both, at different layers.** In OldUnreal's public 227 script source the view is produced by an `event` with three `out` parameters — script-implemented, not `native` — so a `PlayerPawn` subclass can own HMD orientation outright, and `ViewRotation` is `transient norepnotify` so writing it at HMD rate costs nothing. `WalkBob` is added **in that same script event**, so head-bob removal is one omitted line. Per-eye offset and projection stay in our renderer. §12's native bridge shrinks to publishing the pose, via a documented `ucc make -nobind` + `Bound to <Package>.dll` procedure — but the write-up is 227i/VS2008-era and silent on 64-bit. |
| 2026-09-01 | [The `ICBINDx11Drv` on GitHub is not the one in the SDK](topics/2026-09-01-the-icbindx11drv-on-github-is-not-the-one-in-the-sdk.md) | 🆕 new | A time-saver and a small win. Upstream's README says it "is only able to be built for UT 469 only" — 227 is a supported extension target, not what you get by cloning — so the SDK's copy is the authority when a 227-only detail matters. It also confirms render-to-texture is already a shipping capability of this engine's renderers, which de-risks a per-eye render target. And §5's worker-thread question has **no public answer at all**: settle it from the SDK source or at runtime, and stop searching. |

## How to add a topic

1. New file in `topics/`, named `YYYY-MM-DD-short-slug.md`.
2. One row added to the table above, newest at the top.
3. Update the status tag here as it moves through review → incorporated/dead-end (the modding side should update this when it acts on a lead, so the index reflects reality without the research side needing to poll).

## A note for this project specifically

A cross-project research sweep already found one Unreal Gold-relevant item worth following up:
**OldUnreal's SDK is at v227k_15 publicly** (three releases ahead of this project's v227k_12),
with rendering fixes (volumetric fog, D3D9Drv detail-texture flicker, OpenGLDrv distance-fog
compatibility) directly relevant to this project's open fog-calibration polish item — flagged in
`flat-to-vr-cross-engine-research`'s sweep log, 2026-08-24. Worth a first topic write-up here:
whether an SDK bump is worth it (write-up should weigh the OldUnreal changelog against this
project's own already-working from-scratch renderer, not just link the SDK release).
