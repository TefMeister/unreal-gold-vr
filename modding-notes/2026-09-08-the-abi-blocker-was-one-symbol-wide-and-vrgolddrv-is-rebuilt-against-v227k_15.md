# 2026-09-08 — the ABI blocker was one symbol wide, and `VRGoldDrv` is rebuilt against v227k_15

`/pd`, dev PC. **The game was not launched and nothing here has been run.** Every fact below came
from files on disk plus one GitHub API read and one SDK download. The rebuilt driver is deployed
and stamped; whether it *renders* is still `[FLAT]`.

Evidence: `dev-archive/recon/2026-09-08-abi-fix-v227k_15/`.

---

## 1. What was blocking, and how it was reproduced without launching

The 2026-09-04 launch died on a Windows loader error: entry point `?PostLoad@UProperty@@UEAAXXZ`
not found for `VRGoldDrv.dll`. That was `[verified-live 2026-09-04, n=1]` and cost a launch.

It never needed to. Every symbol a PE imports is in its import table; every symbol a DLL exports is
in its export table; the answer is a set difference:

```
deployed Core.dll exports PostLoad for:  UClass, UField, UFunction, UObject
                                    NOT:  UProperty
old VRGoldDrv.dll imports:                ?PostLoad@UProperty@@UEAAXXZ
```

`[verified-numerically 2026-09-08]`. The live error, reproduced statically.

Better, the diff bounds the damage. Across **207 imports over 6 DLLs**, the old driver was missing
**exactly one** symbol. This was never a broad version chasm; it was one entry point.

---

## 2. Which SDK — settled by symbol sets, not by date arithmetic

Two inbox drops had taken this as far as it could go without a local read, and both were folded in
and deleted. The `/gr` drop (2026-09-05) established from the GitHub API that **only v227k_12 and
v227k_15 have ever shipped an SDK** — re-confirmed today, same numbers `[verified-live 2026-09-08,
n=1 API read]`. The `/gs` drop (2026-09-07) then corrected that drop's ⭐ section: nothing had been
misread, because the engine prints `Subversion: 11` **and** `Compiled: Aug 15 2026` in the same
banner. **`Subversion:` is OldUnreal's internal counter; `v227k_NN` is the release tag. They are
different numberings and comparing them manufactured the contradiction.**

Both drops then rested the version identification on dates: build 2026-08-15, v227k_15 published
2026-08-16, install downloaded 2026-08-21. Sound, but `[hypothesis]`.

**On this machine both suggested reads were unavailable** — `System64\Unreal.log` is 0 bytes, and
neither `Core.dll` nor `Engine.dll` carries a version resource. So the question was settled a third
way, and it is a stronger one:

| | has `?PostLoad@UProperty@@UEAAXXZ` |
| --- | --- |
| v227k_12 `Core.lib` | **yes** |
| v227k_15 `Core.lib` | **no** |
| deployed `Core.dll` | **no** |

`[verified-numerically 2026-09-08]`. The deployed engine agrees with v227k_15 on precisely the
symbol that broke, and disagrees with v227k_12. That is a second independent **use** of the ABI
rather than a second reading of a date — which is the kind of corroboration §6 asks for.

⚠️ **What it is not:** proof that the deployed build is v227k_15 in every respect. Neither SDK's
symbol set is a subset of the deployed exports (v227k_12 has 132 symbols the engine lacks, v227k_15
has 107), which is expected for an import library generated from a different build, but it does mean
"the deployed engine is v227k_15" stays `[hypothesis]` in general. What is *established* is the only
part that matters here: **the driver's own 123 engine imports all resolve.**

---

## 3. The rebuild changed exactly one symbol

Rebuilt against v227k_15, the driver's import set differs from the old one by one entry, in one
direction each way:

```
only in the old (v227k_12) build:  ?PostLoad@UProperty@@UEAAXXZ
only in the new (v227k_15) build:  ?PostLoad@UField@@UEAAXXZ
```

`[verified-numerically 2026-09-08]`. The override moved up to the base class between the two
releases, and the deployed engine exports the base. That is a complete and unmysterious account of
the failure — no version-skew hand-waving left over.

The check now passes:

```
Core.dll     100 imported,  2846 exported, 0 MISSING
Engine.dll    23 imported,  4770 exported, 0 MISSING
```

### ⚠️ What this does NOT establish

**That the driver works.** It establishes that the loader will not reject it for a missing entry
point — exactly the failure that happened, and nothing beyond it. A signature change hiding behind
an unchanged mangled name is invisible to an import/export diff and would still fail at runtime, and
so would any behavioural difference between the two SDK versions. The dev-PC `[FLAT]` rows are
**unblocked, not answered.**

The two host suites still pass against the rebuilt driver — `StereoMathTest` 70,550 checks and
`GammaTest` 2,070 checks, 0 failures `[verified-numerically 2026-09-08]` — but those exercise
`Inc/VRGoldStereoMath.h` and `Inc/VRGoldGamma.h`, which the SDK bump does not touch. They are
evidence that nothing regressed, not evidence that the driver renders.

---

## 4. `check-abi.sh` — so this class of failure never costs a launch again

`VRGoldDrv/check-abi.sh` diffs every imported symbol against the deployed `System64` DLLs' exports
and lists the missing ones per source DLL. Exit 0 = the loader will accept it.

It skips DLLs that are not in `System64` (kernel32, user32, d3d11, dxgi, d3dcompiler) on purpose:
those come from Windows, so their absence would mean something entirely different and is not this
script's question.

**Run it after every build.** It is the whole of the 2026-09-04 failure, asked of two files on disk.

---

## 5. Two build defects fixed alongside

**(a) The SDK moved its import libraries.** v227k_12: `<Package>/Lib64/`. v227k_15:
`<Package>/Lib/x64/`. `CMakeLists.txt` now probes both, so one file builds against either SDK — which
matters, because the two are *not* interchangeable and which one is correct depends on the deployed
engine. This also corroborates the earlier `[inferred-static]` reading that the asset rename plus
halving (50.7 MB `SDK.zip` → 26.6 MB `SDK-Windows.zip`, plus a new Linux split) signalled a
restructured SDK rather than a patch release.

**(b) The build was not reproducible.** Two builds of identical source differed in **2 bytes** — the
PE `TimeDateStamp` at `0x118` and its copy in the debug directory at `0x27da4` — and therefore
hashed differently. `CONVENTIONS.md` tells a session to *rebuild and compare the hash* to decide
whether a deployed DLL is current, and that check **silently could not work on this project**. Fixed
with the MSVC linker's `/Brepro`: two builds are now byte-identical, **0 differing bytes**
`[verified-numerically 2026-09-08]`.

That is the same defect `doom-2016-vr` fixed the same day with `-Wl,--no-insert-timestamp`. Two
projects, two toolchains, one blind spot — worth assuming it is present on any project whose board
says "rebuild and compare" until someone has actually run the comparison twice.

---

## 6. Deployment

| | |
| --- | --- |
| deployed | `D:\nonSteam\UnrealGold\System64\VRGoldDrv.dll`, `304f2d2ce96d`, 198,144 B, stamped |
| backup | `VRGoldDrv.dll.bak-2026-09-08-v227k_12-abi-broken` — the broken build, preserved |
| SDK | v227k_15 extracted at `C:\Users\Tefa\ue1-sdk-227k_15`; the v227k_12 copy at `ue1-sdk-227k` left untouched |

⚠️ **`deployed.sh check` said `NO-RECORD` for this project** — it had never been stamped on this
machine, so the board's "deployed" was prose. It is stamped now. The installed file was confirmed
byte-identical to the old broken build before it was replaced, so what the board called deployed
really was the DLL that failed to load.

⚠️ **The home PC needs the v227k_15 SDK before its `[FLAT @home]` M2 proof**, and its install shows
the same `Subversion: 11` / `Compiled: Aug 15 2026` pair, so the same rebuild applies there. Either
copy the rebuilt DLL across or repeat the build; `check-abi.sh` will confirm it against that
machine's own `System64` either way.

---

## 7. What to run next time the game is up

```
1. set [Engine.Engine] GameRenderDevice=VRGoldDrv.VRGoldRenderDevice
2. launch System64\Unreal.exe
```

| outcome | meaning |
| --- | --- |
| **it loads and renders** | the ABI fix holds; go straight to the blue-sky BGRA→RGBA check, then the gamma A/B |
| **a loader error naming a different symbol** | `check-abi.sh` missed it — that would mean the deployed `System64` is not the one it checked against, so re-run it and say which symbol |
| **it loads but renders wrongly** | an ABI question no import/export diff can answer — a signature changed behind an unchanged mangled name, or v227k_15 changed behaviour. This is the case the note above says is *not* excluded |
| **the same `UProperty::PostLoad` error** | the deployed DLL is not the one stamped today — check the hash against `304f2d2ce96d` first |

Then, unchanged: `VRGOLD STEREO 1` for the M2 proof — two squashed side-by-side worlds with near
objects offset and the HUD still full-width proves M2; identical halves even at `IPD 20` is a
cbuffer layout bug; a black right half means the second eye is not bound; `VRGOLD STEREO 0` reverts
live.
