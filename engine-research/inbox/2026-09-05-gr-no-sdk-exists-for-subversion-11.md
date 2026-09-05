Supersedes: `unreal-gold-vr/engine-research/ENGINE-DOSSIER.md` §9, the 2026-09-04 ABI entry's prescribed fix "rebuild `VRGoldDrv` against the EXACT Core deployed here (confirm the running `Core.dll`/`Engine.dll` sub-version and link against its matching SDK)"

# No SDK exists for sub-version 11 — the prescribed ABI fix cannot be performed as written

**Filed by `/gr`, 2026-09-05 (estate sweep). For the modding lane.** No launch, nothing downloaded —
this is the GitHub release listing. Full write-up:
`external-research/topics/2026-09-05-there-is-no-sdk-for-subversion-11-so-the-abi-fix-must-move-the-engine.md`.

## The problem with the current fix

The dossier's 2026-09-04 entry (and the board's only `[PD]` row) says to *"link against its matching
SDK"* for the deployed sub-version, recorded in §1 as **227 sub ~11**.

**There is no sub-11 SDK.** Every `OldUnreal/Unreal-testing` release
`[verified-live 2026-09-05, n=1 API read]`:

| release | published | SDK asset |
| --- | --- | --- |
| `v227k` (= 227k_11) | 2024-03-08 | **none** |
| `v227k_12` | 2024-11-30 | `OldUnreal-UnrealPatch227k-SDK.zip` (50,655,156 B) |
| `v227k_13` | 2026-01-19 | **none** |
| `v227k_14` | 2026-03-01 | **none** |
| `v227k_15` | 2026-08-16 | `OldUnreal-UnrealPatch227k-SDK-Windows.zip` (26,611,815 B) |

Only **two** SDKs exist. So if the engine really is sub 11, the only route is the alternative the
same dossier entry already offers beside it — *"align the deployed engine to the SDK"* — i.e. **move
the engine, not the driver.**

## ⭐ But §1 and §9 may be disagreeing with each other, and that changes everything

§1 records the deployed engine as *"version 227, subversion 11, **engine built 2026-08-15**"*.

- 227k_11 shipped **2024-03-08**. Its binaries cannot carry a 2026-08-15 build date.
- **v227k_15 was published 2026-08-16 — one day after that build date.**

So the deployed engine is probably **227k_15**, and *"subversion 11"* is the misread fact.
`[hypothesis]` — inference from two dates, nothing measured.

**Why it is worth checking before anything else:** it flips the fix from impossible to trivial.

| if the deployed Core is… | fix |
| --- | --- |
| **227k_15** | download the v227k_15 SDK and rebuild. **The project does not have this SDK** — `MACHINES.md` records v227k_12, which is what the failing driver was built against. One download. |
| genuinely **227k_11** | no SDK exists; upgrade the game to v227k_12 or v227k_15 first |

**Settle it with no launch:** the OldUnreal builds print their full version in `Unreal.log`'s startup
banner, and `System64\Core.dll` carries a file-version resource. Either one names the release
outright — and it is worth trusting that over §1, given the last "deployed" turned out not to mean
"loads".

## Suggested dossier changes

1. **§9 / the 2026-09-04 ABI entry** — replace "link against its matching SDK" with: only v227k_12
   and v227k_15 have SDKs, so the driver must be built against one of those two and the engine
   aligned to it.
2. **§1** — flag the `subversion 11` / `built 2026-08-15` contradiction and mark the sub-version
   unconfirmed until the log banner or version resource is read. If it resolves to 227k_15, §1 needs
   correcting and anything version-sensitive in the dossier deserves a second look.
3. **When the rebuild happens**, run a dependency walk over the new DLL and confirm it imports no
   symbol the deployed Core lacks — that catches this class of failure **before** a launch.
4. The board already warns the same loader error is waiting on the home PC. The same check applies
   there before the `[FLAT @home]` M2 proof.

## Corroboration that the ABI genuinely moved between _12 and _15

The SDK asset was **renamed and roughly halved**: `OldUnreal-UnrealPatch227k-SDK.zip` (50.7 MB) →
`OldUnreal-UnrealPatch227k-SDK-Windows.zip` (26.6 MB), with a separate `-SDK-Linux` asset appearing.
A rename plus a per-platform split plus a halving is a restructured SDK, not a patch release — the
class of change that produces exactly a missing-exported-symbol loader error.
`[inferred-static 2026-09-05]`, from asset names and sizes; nothing was downloaded.

⚠️ **One stale pointer in my own lane, corrected there already:** this project's 2026-09-02 topic
closed the SDK-bump question as "not worth it". That verdict was about **fog and gamma** and is
sound; it says nothing about ABI, and a reader meeting it first could skip the bump that fixes this.
Its INDEX row now says so.

Credit: **OldUnreal** (the 227 patch team), already credited in this project's `CREDITS.md`.
