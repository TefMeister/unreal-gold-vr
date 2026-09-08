# There is no SDK for sub-version 11 — so the ABI fix has to move the engine, not the driver

**Status:** 🆕 new · **Priority:** high — the board's only `[PD]` row prescribes a rebuild that
**cannot be performed as written**, and every dev-PC `[FLAT]` row is blocked behind it.

## The row this is about

The 2026-09-04 launch failed with a loader error — `?PostLoad@UProperty@@UEAAXXZ` not found — and the
board's fix reads:

> "Read the running `Core.dll` sub-version in `System64\` and rebuild the driver against **THAT
> sub-version's SDK**, then re-set the renderer and re-launch."

The dossier records the deployed engine as **227 sub ~11** and `MACHINES.md` records the SDK in hand
as **v227k_12**. So the instruction is: get the sub-11 SDK.

## ❌ That SDK does not exist, and never did

Every release on `OldUnreal/Unreal-testing`, read via the GitHub API
`[verified-live 2026-09-05, n=1 API read]`:

| release | published | ships an SDK? | asset |
| --- | --- | --- | --- |
| `v227k` (= **227k_11**) | 2024-03-08 | **no** | `Unreal-227k_11-Windows.zip` only |
| `v227k_12` | 2024-11-30 | **yes** | `OldUnreal-UnrealPatch227k-SDK.zip` (50,655,156 B) |
| `v227k_13` | 2026-01-19 | **no** | — |
| `v227k_14` | 2026-03-01 | **no** | — |
| **`v227k_15`** | **2026-08-16** | **yes** | **`OldUnreal-UnrealPatch227k-SDK-Windows.zip` (26,611,815 B)** |

**Exactly two SDKs have ever been published: v227k_12 and v227k_15.** There is no sub-11, sub-13 or
sub-14 SDK to build against. If the deployed engine really is sub 11, the prescribed fix is
impossible, and the only route is the **other** branch the dossier already lists beside it: *"or
align the deployed engine to the SDK the driver was built with."* Move the engine, not the driver.

## ⭐ But first check this, because it probably changes the answer entirely

The dossier says the deployed engine is *"version 227, subversion 11, **engine built 2026-08-15**"*.
Those two facts do not sit together comfortably:

- **227k_11 was released 2024-03-08.** An engine binary from that release would carry a 2024 build
  date, not an August 2026 one.
- **v227k_15 was published 2026-08-16** — **one day after** the recorded build date. That is exactly
  the gap you would expect between compiling a release and publishing it.

So the balance of evidence says **the deployed engine is 227k_15, and the "subversion 11" reading is
the thing that is wrong** — plausibly a misread field, since Unreal's version fields are `227` plus a
subversion and the OldUnreal tag names (`227k_11`…`227k_15`) are not the only number in play.
`[hypothesis]` — this is inference from two dates, not a measurement.

**It matters enormously, because it flips the fix from "impossible" to "download one file":**

| if the deployed Core is… | the fix is… |
| --- | --- |
| **227k_15** (build date says so) | download `OldUnreal-UnrealPatch227k-SDK-Windows.zip` from the v227k_15 release and rebuild `VRGoldDrv` against it. **One download; the project does not have this SDK.** |
| genuinely **227k_11** | no SDK exists — upgrade the game to v227k_12 or v227k_15 and rebuild |

Either way the destination is the same pair of SDKs, and **either way the project needs the
v227k_15 SDK it does not currently have** if it wants to match a 2026-08-15 engine.

**How to settle it with no launch:** the OldUnreal builds print their full version to `Unreal.log` at
startup, and the deployed `Core.dll`/`Engine.dll` carry a file version resource. One `Unreal.log`
line, or one properties read on `System64\Core.dll`, names the release outright — cheaper than any
inference in this file, and it should be done before acting on any of it.

## ✏️ A stale verdict in this lane, which this supersedes in scope

This project's own `topics/2026-09-02-sdk-227k15-vs-227k12-fog-gamma-changes.md` concluded:

> "The SDK-bump question from 2026-08-24 can be closed as 'checked, not worth it for this reason'."

**That verdict was right about the question it was asked, and is now misleading about a different
one.** It compared the v227k_12 → v227k_15 changelogs for **fog and gamma** and correctly found
nothing worth the bump — every fix named was in `D3D9Drv`/`OpenGLDrv`/`XOpenGLDrv`, reference
renderers `VRGoldDrv` does not fork. Fine.

But on 2026-09-04, **a completely new reason to bump appeared**: the driver will not load against the
deployed engine. A reader who meets the 2026-09-02 line first could reasonably skip the SDK bump —
which may be precisely the fix. The closure should now read *"not worth it for fog/gamma; re-opened
2026-09-05 on ABI grounds."* Nothing in that topic's fog/gamma reasoning is retracted.

**Corroborating detail:** the SDK asset was **renamed and roughly halved** between the two releases —
`OldUnreal-UnrealPatch227k-SDK.zip` (50.7 MB) became
`OldUnreal-UnrealPatch227k-SDK-Windows.zip` (26.6 MB), with a separate `-SDK-Linux` asset appearing
alongside it. A rename plus a per-platform split plus a halving is a **restructured SDK**, not a
patch release — consistent with headers and link libraries having moved between _12 and _15, which
is the class of change that produces exactly this `UProperty::PostLoad` symptom.
`[inferred-static 2026-09-05]` — inferred from asset names and sizes; nothing was downloaded.

## Concrete next steps

1. **Read the deployed engine's real release** from `Unreal.log`'s startup banner or `Core.dll`'s
   file-version resource. No launch of the VR renderer needed. This one fact selects the branch.
2. If it is **227k_15** (expected): fetch `OldUnreal-UnrealPatch227k-SDK-Windows.zip` from
   `https://github.com/OldUnreal/Unreal-testing/releases/tag/v227k_15` and rebuild `VRGoldDrv`
   against it. Confirm the rebuilt DLL no longer imports a symbol the deployed Core lacks — a
   dependency walk over the built DLL answers that **before** any launch, which is worth doing
   given the last "deployed" turned out not to mean "loads".
3. If it is genuinely **227k_11**: upgrade the game to v227k_15 and use the same SDK. Note the
   deployed engine would then be changing under every existing measurement in the dossier, so re-check
   anything version-sensitive.
4. Either way, **apply the same check to the home PC before the `[FLAT @home]` M2 proof** — the board
   already warns the identical loader error is waiting there if the versions do not line up.

## Sources

- https://github.com/OldUnreal/Unreal-testing/releases — all five `227k` releases with full asset
  lists and sizes, read via the GitHub API 2026-09-05. Credit: **OldUnreal** (the 227 patch team).
- This project's own `topics/2026-09-02-sdk-227k15-vs-227k12-fog-gamma-changes.md` (the verdict
  re-scoped above) and `engine-research/ENGINE-DOSSIER.md` §1, §2 and the 2026-09-04 ABI entry.


---

## ✅ Outcome — acted on and confirmed, 2026-09-08 (`/pd`, dev PC, no launch)

Folded in by `/gr` on 2026-09-08 from `external-research/inbox/`.

**The ⭐ call above held, and the "trivial branch" was the right one.** The v227k_15 SDK was
downloaded — `OldUnreal-UnrealPatch227k-SDK-Windows.zip`, **26,611,815 B**, exactly the size
recorded above — `VRGoldDrv` was rebuilt against it, and the ABI check went from **1 missing
symbol to 0**. Deployed and stamped. The API read reproduced exactly: only `v227k_12` and
`v227k_15` carry an SDK asset `[verified-live 2026-09-08, n=1 API read]`.

### ⚠️ How the version was actually settled — both checks proposed here were unavailable

This topic settled the version by **date arithmetic** (build 2026-08-15 against v227k_15 published
2026-08-16) and proposed confirming it by reading `Unreal.log`'s banner or `Core.dll`'s version
resource. **Neither was available on the machine:** the log is 0 bytes, and neither engine DLL
carries a version resource.

It was settled a third way instead, and a stronger one — by the ABI's own behaviour rather
than by metadata about it:

| | has `?PostLoad@UProperty@@UEAAXXZ` |
| --- | --- |
| v227k_12 `Core.lib` | yes |
| v227k_15 `Core.lib` | no |
| **deployed `Core.dll`** | **no** |

`[verified-numerically 2026-09-08]` — the deployed engine agrees with v227k_15 on exactly the
symbol that broke.

**Generalisable lesson, worth reaching for first next time:** when a version must be pinned, a
symbol that is present in one candidate and absent in another is an independent *use* of the ABI;
a log banner or a version resource is a second reading of the same publication metadata, and may
simply not be there.

⚠️ **What this does not establish.** It is not proof the deployed build is v227k_15 in
every respect — neither SDK's symbol set is a subset of the deployed exports (107 and 132
symbols respectively are absent), which is normal for an import library from a different build.
What is established is the part that mattered: **the driver's 123 engine imports all resolve.**

### ✅ The `[inferred-static]` "restructured, not patched" reading is confirmed

Inferred here from the asset rename plus the halved size; confirmed on disk. The import libraries
**moved**, `<Package>/Lib64/` → `<Package>/Lib/x64/`, and `CMakeLists.txt` had to be taught
both layouts.

### What is left is not a research question

The rebuild proves the loader will accept the driver. **Whether it renders is `[FLAT]` and
unchanged.** Nothing in this topic needs further searching.
