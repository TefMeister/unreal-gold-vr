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
VR render device* plus an UnrealScript companion package — no injection, no
hooking, no patched game binaries. The render device is **written from
scratch** against the official OldUnreal 227k SDK; existing renderers such as
the MIT-licensed
[ICBINDx11Drv](https://github.com/metallicafan212/ICBINDx11Drv) are studied
and credited as prior art, but no one else's code is used — every line is our
own, by deliberate policy. That makes the road harder, and that is rather the
point: the playable mod is almost the by-product. The real goal is the
knowledge gained on the way there, written down and shared so anyone can do
the same for any game — see the
[engine dossier](https://github.com/TefMeister/unreal-gold-vr-engine-research)
and the cross-engine
[flat-to-VR library](https://github.com/TefMeister/flat-to-vr-cross-engine-research).

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

## Contributing & policy

See [CONTRIBUTING.md](CONTRIBUTING.md) — how we credit and link sources, our
**study-everything-public but write-our-own-code** rule (we copy no one else's
source code or files, any license or price), the terms for reusing our work
(free, with credit), and how to request a correction or removal.
