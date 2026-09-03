# Research index

**Last `/gr` pass: 2026-09-03 (estate sweep) — CHECK-IN** (board OPEN block + INDEX + the two gamma topics)**.** Inbox empty; my three drops are still unread in `engine-research/inbox/` and were left alone. **Attempted the `[PD]` gamma row and came up inconclusive — recorded rather than padded.** The 2026-09-02 pass established that `GammaMode` is a *choice* between imitating XOpenGLDrv or D3D9Drv, so "stock" is not one curve; this pass tried to pin the actual exponent by reading XOpenGLDrv's published source. Two dead ends worth recording so the next pass skips them: **GitHub code search now requires authentication**, so `search?q=…&type=code` returns a sign-in wall to any fetcher; and `XOpenGLDrv.int` does **not** carry the gamma option text (the repo landing page's mention of `GammaMultiplier` / `GammaOffsetScreenshots` points somewhere else, most likely `Src/`). ⚠️ That negative is unconfirmed by a positive control, so treat it as "not found here" rather than "absent". The remaining route is a direct read of a named file under `Src/`, or the SDK's own on-disk copy — which the 2026-09-01 topic already argues is the more trustworthy of the two.
_Previous: **Last `/gr` pass: 2026-09-02 (user-scoped run, second pass same day) — FULL.** Drained the one waiting `/sr` inbox drop (corroborating detail on the UE1 scale topic — folded in as an addendum, not a correction). New topic: the SDK v227k_12→v227k_15 changelog …_
re-read)**.** Inbox empty. M2 stereo was built and numerically verified on the home PC today, so the
lane's job was the one `[hypothesis]` it introduced: **the UE1 world scale.** The Unreal Wiki's
per-generation table puts Unreal/UT99 at **44.6 UU/m**, not the 52.5 folk constant (which fits
UT2004), so `StereoIPD=3.4` is ~20 % wide and ~2.85 is the better default. Pointer in
`engine-research/inbox/`. Also filed the other way: XIII (UE2) received today's per-batch two-viewport
design as a cross-project hand-off.
_2026-09-01 (second pass, estate sweep) — FULL:_ Inbox empty. **The first two
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
| 2026-09-02 | [v227k_12→v227k_15: what changed for fog/gamma, and does it matter to VRGoldDrv](topics/2026-09-02-sdk-227k15-vs-227k12-fog-gamma-changes.md) | 🆕 new | Closes the 2026-08-24 SDK-bump question: the three releases past v227k_12 fix fog only in `D3D9Drv`/`OpenGLDrv`/`XOpenGLDrv` code this from-scratch renderer never touched, and add nothing gamma-related. But `ICBINDx11Drv`'s own `GammaMode` setting picks between two different gamma algorithms ("XOpenGL"/"DX9") — stock has no single canonical curve, which reframes `bGammaCorrectOutput`'s open comparison: record which `GammaMode` (and `GammaCorrectScreenshots`) was active in the stock screenshot before comparing. |
| 2026-09-02 | [UE1's world scale is nearer 45 UU per metre than 52.5 — the `StereoIPD=3.4` default is about 20 % too wide](topics/2026-09-02-ue1-world-scale-is-nearer-45-uu-per-metre-than-52-5.md) | 🆕 new | The Unreal Wiki gives Unreal/UT99 **13.6 UU/ft = 44.6 UU/m** (78-UU player), UT2003/4 50.3 UU/m, and calls 16 UU/ft (52.5 UU/m) the rule the *newer* games sit closest to. 64 mm on UE1 is ~2.85 UU, not 3.4. M3 still measures; this names the starting number and its source. **Corroborated by an independent `/sr` pass, 2026-09-02** — same figure, same primary source; addendum in the topic file adds a cheap pre-headset sanity check (78 UU player height vs. a doorway/NPC). |
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
