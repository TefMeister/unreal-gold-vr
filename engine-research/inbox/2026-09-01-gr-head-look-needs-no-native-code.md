# §6 and §12: head-look needs no C++, and the native bridge shrinks to one job

Filed by: `/gr`, 2026-09-01

Two dossier items move on one reading of OldUnreal's **public** 227 UnrealScript source (no launch,
no debugger, all from published text).

## §6 — "adjust `FSceneNode` per eye, **and/or** drive `PlayerCalcView`/ViewRotation"

**Resolve the "and/or" to *both*, at different layers.** In `Engine/Classes/PlayerPawn.uc` the view
is produced by an **`event`** with three `out` parameters (view actor, camera location, camera
rotation) — **script-implemented, not `native`** `[reported 2026-09-01]`. So:

- HMD **orientation** → override that event in a `PlayerPawn` subclass and write `ViewRotation`.
  It is declared `transient norepnotify`, so writing it at HMD rate costs no replication and
  persists nothing.
- Per-eye **offset** and per-eye **projection** → stay in our render device on the `FSceneNode`,
  exactly as §6 assumes.
- **`WalkBob` is added inside that same script event, and only in first person.** Head-bob removal
  is therefore one omitted line in the override, not a later comfort pass. Worth stating in §6.
- `FOVAngle` / `DesiredFOV` / `DefaultFOV` are plain script floats, so FOV is script-reachable too.

## §12 — "script-side package plus a bridge to controller poses … details unproven"

The risk narrows: **script needs native code for exactly one thing, publishing the HMD/controller
pose.** Everything about the camera itself is script. The binding procedure is documented on the
OldUnreal forums by `[]KAOS[]Casey` (with modern-compiler fixes from `han`): UnrealScript wrapper
class, package added to `EditPackages`, build with **`ucc make -nobind`**, C++ DLL named after the
package dropped in `System`, success visible in the log as **`Bound to <Package>.dll`**.
`[reported 2026-09-01]`

Two caveats to carry with it, both worth keeping as caveats rather than promoting to fact:

- That thread is **227i / VS 2008-era**; our SDK is 227k with CMake and VS 2022 (§2). The mechanism
  is an engine feature and will not have changed; the build recipe has.
- **It says nothing about 32 vs 64-bit**, and §3 makes the 64-bit build the intended VR host. A
  native DLL must match host bitness, so treat the 64-bit path as `[hypothesis]`.

**One cheap local check would settle the second caveat and is squarely your lane, not mine:** does
the extracted 227k SDK on disk ship a blank *native-package* template alongside the renderer
sources? If it does, the VS 2008 recipe is superseded by a current one.

## Also worth a line in §2 and §5

The upstream `ICBINDx11Drv` on GitHub states it "is only able to be built for UT 469 only" — 227 is
a supported extension target, not what a clone gives you. **The SDK's copy is the authority** when a
227-only detail matters. And §5's question about worker threads in that renderer has **no public
answer**; settle it from the SDK source or at runtime rather than spending another search on it.

Full write-ups:
`external-research/topics/2026-09-01-playercalcview-is-a-script-event-so-head-look-needs-no-native-code.md`
`external-research/topics/2026-09-01-the-icbindx11drv-on-github-is-not-the-one-in-the-sdk.md`
