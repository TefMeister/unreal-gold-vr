# Unreal Gold (1998) — VR Engine Research

Engine research toward a VR conversion of **Unreal Gold (1998)** — the very
first Unreal Engine game — running on the **OldUnreal 227k** community patch,
with SteamVR/OpenXR output and motion-controlled weapon aim as the goal.

This repository holds two things:

- **[`PLAYBOOK.md`](PLAYBOOK.md)** — a reusable, engine-agnostic, point-by-point
  method for taking *any* game whose engine nobody has converted to VR and
  getting it there. It is oriented around one North Star: **the game rendering
  in a headset with head tracking**, with everything else built on top. The same
  playbook is copied into each of our VR projects' research repos.
- **[`ENGINE-DOSSIER.md`](ENGINE-DOSSIER.md)** — the distilled, current-truth
  reference for *this* game's engine: the Unreal Engine 1 module layout, the
  pluggable `URenderDevice` renderer model, how the camera reaches the renderer
  (UE1 famously has no view matrix), the OldUnreal 227k specifics, and the dead
  ends — so they don't cost the next session the same time.

The blow-by-blow development history lives in the sibling repositories
(`-dev-archive` for the messy in-progress record, `-modding-notes` for readable
field notes). This repo is the consolidated engine knowledge, not the diary.

## Why this project is unusual (in a good way)

Unlike our other conversions, this one needs **no injection, hooking, or
memory patching**. Unreal Engine 1 loads its renderer as a plugin DLL
(`URenderDevice`), the OldUnreal 227k patch is an Epic-licensed maintenance
patch with public UnrealScript source, and the D3D11 renderer we fork
([ICBINDx11Drv](https://github.com/metallicafan212/ICBINDx11Drv)) is MIT
licensed. The VR mod is simply *another renderer* plus a script package —
built entirely from published, licensed interfaces.

## The five repositories for Unreal Gold VR

Everything for this game lives in five repositories, each with one job — so you
always know where to look. You are in **unreal-gold-vr-engine-research**.

| Repository | What lives here |
| --- | --- |
| [unreal-gold-vr-mod](https://github.com/TefMeister/unreal-gold-vr-mod) | The mod itself — the VR render device + UnrealScript companion package. |
| [unreal-gold-vr-dev-archive](https://github.com/TefMeister/unreal-gold-vr-dev-archive) | Full development history — snapshots, probes, dead ends, raw recon. |
| [unreal-gold-vr-modding-notes](https://github.com/TefMeister/unreal-gold-vr-modding-notes) | Readable field notes / progress ledger. |
| [unreal-gold-vr-staging](https://github.com/TefMeister/unreal-gold-vr-staging) 🔒 | **Private** — unverified WIP builds, cross-machine handoff. |
| **unreal-gold-vr-engine-research** ← you are here | Distilled engine reference (dossier) + reusable VR RE playbook. |

## Status

Project started 2026-08-21. Groundwork phase: repos created, design approved
(native VR render device forked from ICBINDx11Drv; SteamVR first, OpenXR
second; decoupled motion-controller weapon aim). See the dossier for the
current phase and open risks.

## Scope, ethics, and legality

- This is a **non-commercial fan project**. It requires owning a legitimate copy
  of the game and **redistributes no original game assets** — only files we
  create. See [`.gitignore`](.gitignore).
- This particular project builds against published plugin interfaces and
  MIT/publicly licensed source — no reverse-engineering tooling is required for
  the core approach.
- We **credit everyone** whose work or research this builds on, and we honour
  correction/removal requests from actual rights holders. See
  [`CREDITS.md`](CREDITS.md).

## Templates

New engine? Start its dossier from
[`templates/per-engine-research-template.md`](templates/per-engine-research-template.md).

## Contributing & policy

See [CONTRIBUTING.md](CONTRIBUTING.md) — how we credit and link sources, our
**study-everything-public but write-our-own-code** rule (we copy no one else's
source code or files, any license or price), the terms for reusing our work
(free, with credit), and how to request a correction or removal.
