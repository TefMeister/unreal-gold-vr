# Head-look needs no C++ at all: `PlayerCalcView` is a **script event** in 227's public source

**Date:** 2026-09-01
**Bears on:** `ENGINE-DOSSIER.md` §6 (camera & projection delivery) and §12 (the unproven
script-side-to-controller-pose bridge)

## Why this was worth checking

§6 lists head-tracking injection as an open choice — "adjust the `FSceneNode` coords in the render
device per eye, **and/or** drive `PlayerCalcView`/ViewRotation from the script/native side" — and
§12 carries the risk that a script package plus "a bridge to controller poses (227 native DLL
binding)" is a mechanism "confirmed to exist, details unproven". Those two are really one question:
**how much C++ does the camera half of this project actually need?**

UE1's UnrealScript is public for all of 227, so this is answerable by reading rather than guessing.

## The finding

In `Engine/Classes/PlayerPawn.uc` in OldUnreal's public 227 source, the view is produced by an
**`event`** taking three `out` parameters — the view actor, the camera location and the camera
rotation. `[reported 2026-09-01]`

Three things follow, and they are all good news:

1. **It is script-implemented, not `native`.** A `PlayerPawn` subclass can override it outright.
   Nothing about redirecting the camera requires a C++ package — this is ordinary UnrealScript, the
   same lever the XIII (UE2) work already used successfully for head-look.
2. **`ViewRotation` is declared `transient norepnotify`.** Writing it every frame therefore costs no
   replication traffic and persists nothing to disk — exactly the property you want for a value
   driven at HMD rate. A single-player game makes this doubly safe.
3. **`FOVAngle`, `DesiredFOV` and `DefaultFOV` are plain script floats.** FOV is reachable from
   script too, which matters for the "get something on the headset early" path even before proper
   per-eye asymmetric frusta exist.

The event's own body confirms the classic UE1 shape §6 assumes: with no view target it takes the
player's own position, adds `EyeHeight`, and adds `WalkBob` **only in first person**. So the
head-bob that a VR conversion must kill is applied *right here, in script* — not buried in the
renderer.

## What this changes about the plan

It splits the camera work cleanly along the boundary we already have:

| Job | Where it belongs | Needs C++? |
|---|---|---|
| HMD **orientation** into the view | override the script event; write `ViewRotation` | **no** |
| Killing `WalkBob` / head-bob | the same override — just don't add it | **no** |
| Per-eye **offset** (the stereo separation itself) | the render device, on the `FSceneNode` | it is our own renderer anyway |
| Per-eye asymmetric **projection** | the render device | same |

§6's "and/or" resolves to **both, at different layers** — and neither of them needs a native package.
That is a real reduction in M1 risk: the renderer must still be written from scratch, but the camera
does not add a second C++ deliverable on top of it.

## The one thing that still needs native code — and its documented procedure

The remaining gap is narrow and specific: **script cannot obtain the HMD or controller pose by
itself.** Something native must publish it. That is §12's bridge, and the mechanism is documented by
OldUnreal community members rather than merely rumoured:

- Write the UnrealScript wrapper class in the package's `Classes/` folder, add the package to
  `EditPackages` in `Unreal.ini`, and build it with **`ucc make -nobind`** — the flag that compiles
  the script package while leaving its native declarations unbound.
- Build the C++ side against the public headers and their matching `.lib` files, and drop the
  resulting DLL, **named after the package**, into the game's `System` folder.
- The engine binds them at spawn time and says so in the log: a line reading **`Bound to
  <Package>.dll`** is the success signal. `[reported 2026-09-01]` — described by `[]KAOS[]Casey`
  on the OldUnreal forums, with modern-compiler fixes from moderator `han`.

**Read that with two caveats, both worth resolving before anyone budgets time on it:**

- The thread is **227i / Visual Studio 2008-era**, while our SDK is 227k with CMake and VS 2022
  (§2). The binding *mechanism* is an engine feature and will not have changed; the *build recipe*
  around it certainly has.
- **Bitness is not addressed anywhere in that thread**, and §3 makes the 64-bit build the intended
  VR host. A native DLL must match the host executable, so the 64-bit path is an assumption here,
  not a fact. `[hypothesis]`

## Concrete next steps this unlocks

1. **Cheapest first, and it needs no VR at all:** subclass `PlayerPawn`, override the view event,
   and drive `ViewRotation` from anything — a console command, a timer, mouse input. If the view
   moves and the world renders correctly, the entire script half of head-look is proven, months
   before a headset is in the loop.
2. **Check the SDK already on disk for a native-package example.** §2 records the extracted 227k SDK
   as shipping headers, `.lib` files and *renderer* sources. If it also ships a blank native-package
   template, the VS 2008 recipe above is superseded by something current and the last caveat
   evaporates. That is a local file listing, so it belongs to the modding side, not this lane.
3. Drop `WalkBob` from the override on day one rather than treating head-bob as a later comfort
   pass — in this engine it costs one omitted line.

## Sources

- OldUnreal, `Unreal-PubSrc` — the public UnrealScript source for 227, `Engine/Classes/PlayerPawn.uc`
  — https://github.com/OldUnreal/Unreal-PubSrc
- OldUnreal forums, "How to get a 'hello world' going in C++" — the native-package build and binding
  procedure, by `[]KAOS[]Casey` with compiler fixes from `han` —
  https://www.oldunreal.com/phpBB3/viewtopic.php?t=3938
- BeyondUnreal wiki, "Customising the player view" — the general UE1/UE2 statement that this event is
  the single point of control over what the player sees —
  https://beyondunrealwiki.github.io/pages/customising-the-player-view.html
