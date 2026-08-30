# Credits & Attribution

This project is a modding effort built on the public research, tools, and
creative work of many people who came before us. None of this would be
possible without them. We list every source, tool, and prior work we have
drawn on below — by name or handle, as accurately as we could verify it —
including those that helped only as inspiration.

If we have missed someone, the omission is a mistake, not a slight. Please see
the "Get credited, or ask us to stop" section at the bottom.

## The original game

Unreal (1998) and Unreal Gold are the creative work of their developers and
publishers. We are only modding the game; we did not make it, and all rights
to the game and its assets belong to their owners. No game files are included
in any repository in this project.

| Work | Creator(s) | Note |
|---|---|---|
| Unreal (1998) | Epic MegaGames and Digital Extremes (developers); GT Interactive (publisher) | The original game, and the debut of the Unreal Engine. |
| Unreal Mission Pack: Return to Na Pali (included in Unreal Gold) | Legend Entertainment (developer); GT Interactive (publisher) | The expansion bundled into Unreal Gold. |
| Unreal Engine 1 | Epic Games (engine architecture by Tim Sweeney and the Epic team) | The engine, including the pluggable `URenderDevice` renderer model this project relies on. |

## Tools, frameworks, and prior research this project builds on

| Source / Work | Creator(s) | Link |
|---|---|---|
| OldUnreal 227 patch (227k) — the Epic-licensed community maintenance patch this project targets, including its 64-bit builds and modern renderers | Smirftsch and the OldUnreal community | https://www.oldunreal.com / https://github.com/OldUnreal |
| ICBINDx11Drv ("I Can't Believe It's Not DX11") — the MIT-licensed Direct3D 11 render device for UE1; studied as prior art and reference for our from-scratch VR renderer (no code reused, per our study-don't-copy policy) | metallicafan212 and contributors | https://github.com/metallicafan212/ICBINDx11Drv (OldUnreal fork: https://github.com/OldUnreal/ICBINDx11Drv) |
| XOpenGLDrv (modern OpenGL render device; reference for 227 renderer integration) | Smirftsch / OldUnreal and contributors | https://github.com/OldUnreal/XOpenGLDrv |
| Public UnrealScript source for 227 | OldUnreal | https://github.com/OldUnreal/Unreal-PubSrc |
| UT99 Quest — the native Meta Quest VR rebuild of Unreal Tournament (1999); closed-source, but living proof that native UE1 VR with motion controls is achievable, and the source of the "UE1 has no view matrix; stereo comes from where each eye sits" insight | GHWST | https://ut99vr.pages.dev/ |
| OpenVR / SteamVR (VR runtime and compositor; our first target) | Valve | https://github.com/ValveSoftware/openvr |
| OpenXR (cross-vendor VR runtime standard; our second target) | The Khronos Group and contributors | https://www.khronos.org/openxr/ |
| Virtual Desktop / VDXR (Quest-to-PC VR streaming and OpenXR runtime used for all headset testing) | Guy Godin / Virtual Desktop, Inc. | https://www.vrdesktop.net |
| Legacy-flat-to-VR strategy guides (the staged-milestone approach our playbook follows) | Brobert-in-aus (`guides` repo) | https://github.com/Brobert-in-aus/guides |
| Superpowers (skills framework used during development) | Jesse Vincent (GitHub: obra) and contributors at Prime Radiant | https://github.com/obra/superpowers |
| AI development assistance | Claude (Anthropic) | https://www.anthropic.com |

Project lead and author: **TefMeister**.

Where a handle or attribution above is uncertain, we have said so, or we have
linked the source so anyone can check it. If you can correct or confirm a
detail, please open a GitHub issue — we would much rather fix it than leave it
wrong.

## Get credited, or ask us to stop

**If you helped and are not credited:** if you contributed anything to this
work — code, research, tools, documentation, or even just an idea that inspired
a part of it — and you do not see yourself credited above, that is an oversight
on our part, not a judgement about your contribution. Please contact us by
opening a GitHub issue on this repository, and we will correct the credits as
soon as possible.

**If you want your work removed or not used:** if you are the owner or creator
of something referenced or used here, and you would rather your work not be
referenced in this project, or you want specific content removed, please tell
us by opening a GitHub issue. We will honour that request promptly — no
argument and no delay — and we will find another way to do the job that does
not rely on your material. This is your work; we are only grateful to have
learned from it.
