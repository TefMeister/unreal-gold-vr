# The `ICBINDx11Drv` you find on GitHub is not the build that ships in the 227k SDK

**Date:** 2026-09-01
**Bears on:** `ENGINE-DOSSIER.md` §2 (SDK study material) and §5 (the threading TODO)

## The trap

§2 records the 227k SDK as shipping the community renderers — D3D9Drv, OpenGLDrv, XOpenGLDrv,
**ICBINDx11Drv** — as study material, and §3 records ICBINDx11 as the renderer this install is
currently configured to use and verified working. Anyone reaching for the upstream project to study
it will find `metallicafan212/ICBINDx11Drv` on GitHub, MIT-licensed, and reasonably assume it is the
same thing.

**Its own README says otherwise: "Currently, this is only able to be built for UT 469 only."**
`[reported 2026-09-01]`

Its stated lineage is Harry Potter 2's engine and UT 469, with "UT469, HP2, and 227 specific
extensions are also implemented" — so 227 is a *supported extension target*, not the configuration
you get by cloning it. The copy inside the SDK is the one already aimed at our engine.

**Practical consequence:** when a detail matters — a 227-only entry point, how `SetSceneNode` or the
render-to-texture path is actually driven — trust the SDK's copy on disk over the upstream README,
and expect upstream build instructions (Windows 10 SDK, VS 2022) to describe a different target than
ours. Under the no-copy rule both are study material only in any case.

Worth knowing for a second reason: the README does confirm the renderer implements render-to-texture
("RT textures") among its feature set, which is the capability §2's note about 227k's
`PushRenderToTexture` / `PopRenderToTexture` extension implies. A per-eye render target is therefore
a thing this engine's renderers already do in the shipping patch, not something our renderer would be
pioneering.

## The §5 threading TODO: there is no public answer

§5 asks whether ICBINDx11Drv adds worker threads for decals or precache. **Its README makes no claim
about threading at all** — no multithreading, no worker threads, no render-thread language — and no
public write-up covers it either. `[checked 2026-09-01]`

So this stays a TODO, but a narrowed one: it is not a question anybody has answered in public, and
it will be settled either by reading the SDK's source on disk or by watching our own renderer's
thread behaviour at runtime. **Nobody should spend another research session searching for it.** The
working default meanwhile is §5's existing one — UE1 is essentially single-threaded for game and
render tick — since a renderer that quietly spawned worker threads would be an odd thing to leave
entirely undocumented.

## Sources

- `metallicafan212/ICBINDx11Drv` — README: feature list, the UT 469 build limitation, MIT licence,
  build requirements — https://github.com/metallicafan212/ICBINDx11Drv
- OldUnreal, `Unreal-testing` v227k_12 release notes — ICBINDx11Drv's inclusion in the patch —
  https://github.com/OldUnreal/Unreal-testing/releases/tag/v227k_12
