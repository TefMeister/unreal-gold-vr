# Unreal Gold VR

A VR conversion mod for **Unreal Gold (1998)** — true stereo rendering, 6DOF
head tracking, and motion-controlled weapon aim, built on the
[OldUnreal 227k](https://www.oldunreal.com) patch. SteamVR first, OpenXR to
follow.

> ### ⚠️ Status: **v0.1.0-alpha released — EARLY DEVELOPMENT, not ready for a playthrough.**
>
> It fuses into real 3D in a headset and the world is life-size at `StereoIPD=1.0`,
> but explosions, projectile sprites and the HUD are still drawn once across the
> whole window instead of once per eye, and there is no head tracking yet.
>
> 🤢 **Caution: may cause severe motion sickness and discomfort.** Play seated,
> keep it short, take the headset off at the first sign of discomfort. This
> caution stays until the mod is confirmed genuinely comfortable to play.
>
> Get it from the [**Releases page**](https://github.com/TefMeister/unreal-gold-vr/releases).

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
[engine dossier](../engine-research/)
and the cross-engine
[flat-to-VR library](https://github.com/TefMeister/flat-to-vr-cross-engine-research).

## What you will need

- **Unreal Gold** — free. OldUnreal's official installer downloads the game for
  you: [oldunreal.com → Unreal full-game installers](https://www.oldunreal.com/downloads/unreal/full-game-installers/)
  (Windows file: `Unreal_Gold.exe`). This mod contains **no** game files.
- The free **OldUnreal 227k patch, release `v227k_15`** — the mod is built
  against exactly this release and will not load on an older one. The installer
  above normally applies the newest patch for you; if your game is older, get it
  here: [OldUnreal Unreal patch v227k_15](https://github.com/OldUnreal/Unreal-testing/releases/tag/v227k_15)
  (Windows file: `OldUnreal-UnrealPatch227k-Windows.exe`).
- A PC VR headset via **SteamVR** (Quest over Link/Virtual Desktop works).

## The folders for Unreal Gold VR

Everything for this game lives in one repository, one folder per job — so you
always know where to look. You are in **`mod/`**.

| Folder | What lives here |
| --- | --- |
| **`mod/`** ← you are here | The mod itself — the VR render device + UnrealScript companion package. |
| [`dev-archive/`](../dev-archive/) | Full development history — snapshots, probes, dead ends, raw recon. |
| [`modding-notes/`](../modding-notes/) | Readable field notes / progress ledger. |
| [staging/unreal-gold-vr](https://github.com/TefMeister/staging/tree/main/unreal-gold-vr) 🔒 | **Private** — unverified WIP builds, cross-machine handoff. |
| [`engine-research/`](../engine-research/) | Distilled engine reference (dossier) + reusable VR RE playbook. |
| [`external-research/`](../external-research/) | Ongoing public-research leads, gathered separately from hands-on modding work. |

## Credits, scope, and legality

Non-commercial fan project; needs a legitimate copy of Unreal Gold (free from
OldUnreal, linked above); redistributes no original assets. We credit everyone whose work this builds on — see
[`CREDITS.md`](CREDITS.md) — and we honour correction/removal requests from
rights holders promptly.

## Contributing & policy

See [CONTRIBUTING.md](CONTRIBUTING.md) — how we credit and link sources, our
**study-everything-public but write-our-own-code** rule (we copy no one else's
source code or files, any license or price), the terms for reusing our work
(free, with credit), and how to request a correction or removal.
