# Unreal Gold VR

A VR conversion mod for **Unreal Gold (1998)** — true stereo rendering, 6DOF
head tracking, and motion-controlled weapon aim, built on the
[OldUnreal 227k](https://www.oldunreal.com) patch. SteamVR first, OpenXR to
follow.

> **Status: work in progress — nothing playable is released yet.** This
> repository will hold releases only; watch it if you want to know the moment
> there is something to try.

## What this will be

Unreal Engine 1 loads its renderer as a plugin DLL, so this mod is a *native
VR render device* (forked from the MIT-licensed
[ICBINDx11Drv](https://github.com/metallicafan212/ICBINDx11Drv)) plus an
UnrealScript companion package — no injection, no hooking, no patched game
binaries.

## What you will need

- Your own legitimate copy of **Unreal Gold** (this mod contains **no** game
  files).
- The free **OldUnreal 227k** patch.
- A PC VR headset via **SteamVR** (Quest over Link/Virtual Desktop works).

## The five repositories for Unreal Gold VR

Everything for this game lives in five repositories, each with one job — so you
always know where to look. You are in **unreal-gold-vr-mod**.

| Repository | What lives here |
| --- | --- |
| **unreal-gold-vr-mod** ← you are here | The mod itself — the VR render device + UnrealScript companion package. |
| [unreal-gold-vr-dev-archive](https://github.com/TefMeister/unreal-gold-vr-dev-archive) | Full development history — snapshots, probes, dead ends, raw recon. |
| [unreal-gold-vr-modding-notes](https://github.com/TefMeister/unreal-gold-vr-modding-notes) | Readable field notes / progress ledger. |
| [unreal-gold-vr-staging](https://github.com/TefMeister/unreal-gold-vr-staging) 🔒 | **Private** — unverified WIP builds, cross-machine handoff. |
| [unreal-gold-vr-engine-research](https://github.com/TefMeister/unreal-gold-vr-engine-research) | Distilled engine reference (dossier) + reusable VR RE playbook. |

## Credits, scope, and legality

Non-commercial fan project; requires an owned copy; redistributes no original
assets. We credit everyone whose work this builds on — see
[`CREDITS.md`](CREDITS.md) — and we honour correction/removal requests from
rights holders promptly.
