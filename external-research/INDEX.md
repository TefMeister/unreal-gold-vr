# Research index

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
