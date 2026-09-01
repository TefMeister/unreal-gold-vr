# Research index

**Last `/gr` pass: 2026-09-01 — CHECK-IN. Nothing new, and that is the finding.** Inbox empty. Two
targeted searches against the dossier's live gaps, both returning honest negatives worth recording
rather than a manufactured topic:

- **§6's open TODO — *"study how the SDK's reference renderers derive their projection from
  `FSceneNode` (FOV/FX/FY)"*** — has **no public technical write-up**. No article, forum post or
  documentation covers `URenderDevice` / `FSceneNode` projection derivation. **The answer is not on
  the web; it is in the 227k SDK already on disk**, which §2 identifies and which ships several
  complete reference renderers as study material. Worth knowing before anyone spends a session
  searching: this one is a reading task, not a research task.
- **UE1 VR prior art: still none** `[checked 2026-09-01]`. Every search path leads back to UEVR,
  which supports 4.8–5.x only. §11's "UEVR does not apply" and §12's greenfield assumption both
  stand, re-validated as of today. UT99 Quest remains the only feasibility proof and remains
  closed-source and Quest-only.

No topic file was written, because neither result is a lead — they are confirmations that two
load-bearing assumptions are still true.

Every research topic gathered for this project, newest first. Each row links to a self-contained
write-up in `topics/`. Status tags:

- 🆕 **new** — found, not yet acted on by the modding side.
- 👀 **reviewed** — a modding session has read it and factored it into a decision, but nothing shipped from it yet.
- ✅ **incorporated** — directly led to a real change (code, a test, a note) in one of the other five repos; linked below.
- ❌ **dead end** — checked out, didn't pan out; kept for the record so it isn't re-investigated from scratch.

| Date | Topic | Status | Summary |
| --- | --- | --- | --- |

*(No topics yet — this repo was seeded 2026-08-24. Run `/game-research unreal-gold-vr` to start.)*

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
