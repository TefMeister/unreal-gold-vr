# Research index

**Last `/gr` pass: 2026-09-05 (estate sweep) — FULL** (board OPEN block, dossier §1/§2/§9, INDEX, the two SDK topics; one GitHub API read of every OldUnreal 227k release)**.** Inbox empty; the `/sr` 2026-09-03 drop is still in `engine-research/inbox/`. **The "no public-research question here" reading is out of date — the 2026-09-04 ABI failure created one, and it is answered.** The board's only `[PD]` row says to rebuild `VRGoldDrv` against the deployed sub-version's SDK; **no such SDK exists** — only v227k_12 and v227k_15 ever shipped one. The good news is the recorded engine build date (2026-08-15) sits one day before v227k_15's publication, so the deployed engine is probably 227k_15 and the fix is likely a single download this project does not have. One log-banner read decides. Pointer filed to `engine-research/inbox/`; the 2026-09-02 SDK-bump closure re-scoped to fog/gamma only.

_Previous: **Last `/gr` pass: 2026-09-04 (estate sweep) — CHECK-IN** (board OPEN block + INDEX)**.** Inbox empty; the `/sr` 2026-09-03 drop is still in `engine-research/inbox/`. **Nothing new, and not searched** — all three rows are `[FLAT]` and the modding side said on 2026-09-03 that none is a public-research question._
_Previous: **Last `/gr` pass: 2026-09-03 (estate sweep, second pass today) — CHECK-IN** (inbox drained — one modding verdict covering three earlier drops; INDEX; the two topics it extends)**.** All four topics → ✅. **The gamma question this lane left open was settled from the SDK source on disk, not the web**: `Gamma = Brightness × 2.0`, `GM_XOpenGL` is identity at the default `Brightness = 0.5`, so stock applies **no gamma at defaults** and our `pow(x, 1/2.0)` was `[disproved]` and fixed (2,070-check test). Bonus: stock DX9 mode swaps the green and blue gamma exponents. The SDK has no blank native template, but `Emitter/` is a buildable script+native hybrid. **No new research — the modding side says both remaining rows are `[FLAT]` and none is a public-research question.** The "re-read the 227 Release Notes PDF" follow-up is retired._
_Previous: **Last `/gr` pass: 2026-09-03 (estate sweep) — CHECK-IN** (board OPEN block + INDEX + the two gamma topics)**.** Inbox empty; my three drops are still unread in `engine-research/inbox/` and were left alone. **Attempted the `[PD]` gamma row and came up inconclusive — recorded rather than padded.** The 2026-09-02 pass established that `GammaMode`…_
_Previous: **Last `/gr` pass: 2026-09-02 (user-scoped run, second pass same day) — FULL.** Drained the one waiting `/sr` inbox drop (corroborating detail on the UE1 scale topic — folded in as an addendum, not a correction). New topic: the SDK v227k_12→v227k_15 changelog …_
re-read)**.** Inbox empty. M2 stereo was built and numerically verified on the home PC today, so the
lane's job was the one `[hypothesis]` it introduced: **the UE1 world scale.** The Unreal Wiki's
per-generation table puts Unreal/UT99 at **44.6 UU/m**, not the 52.5 folk constant (which fits
UT2004), so `StereoIPD=3.4` is ~20 % wide and ~2.85 is the better default. Pointer in
`engine-research/inbox/`. Also filed the other way: XIII (UE2) received today's per-batch two-viewport
design as a cross-project hand-off.
_2026-09-01 (second pass, estate sweep) — FULL:_ Inbox empty. **The first two
topics in this repo**, both from the dossier's own open items: §6's head-tracking question is
answered — `PlayerCalcView` is a plain **script event** in 227's public source, so head-look and
head-bob removal need **no C++ at all**, and §12's native-DLL bridge narrows to one job (publishing
the HMD pose), with a documented `ucc make -nobind` procedure and two caveats. The second topic is a
trap worth knowing: the `ICBINDx11Drv` on GitHub builds for **UT 469**, not our 227, so the SDK's
copy is the one to study. §5's threading TODO has no public answer and should not be searched again.
_Earlier the same day — CHECK-IN, and still true:_ §6's `FSceneNode` projection TODO has no public
write-up (it is a reading task against the SDK on disk, not a research task), and **UE1 VR prior art
is still none** `[checked 2026-09-01]` — every path leads back to UEVR, which is 4.8–5.x only, so
§11 and §12's greenfield assumption both stand.

Every research topic gathered for this project, newest first. Each row links to a self-contained
write-up in `topics/`. Status tags:

- 🆕 **new** — found, not yet acted on by the modding side.
- 👀 **reviewed** — a modding session has read it and factored it into a decision, but nothing shipped from it yet.
- ✅ **incorporated** — directly led to a real change (code, a test, a note) in one of the other five repos; linked below.
- ❌ **dead end** — checked out, didn't pan out; kept for the record so it isn't re-investigated from scratch.

| Date | Topic | Status | Summary |
| --- | --- | --- | --- |
| 2026-09-05 | [There is no SDK for sub-version 11, so the ABI fix has to move the engine](topics/2026-09-05-there-is-no-sdk-for-subversion-11-so-the-abi-fix-must-move-the-engine.md) | 🆕 new | ⛔ **The only `[PD]` row prescribes a rebuild that cannot be performed as written.** It says to rebuild `VRGoldDrv` against the deployed sub-version's SDK — but **only two 227k SDKs have ever been published**, v227k_12 and v227k_15 `[verified-live 2026-09-05, n=1 API read]`; there is none for 227k_11, _13 or _14. ⭐ **The likely resolution is far better than that sounds:** the dossier's deployed engine is recorded as **built 2026-08-15** and **v227k_15 was published 2026-08-16**, so the "subversion 11" reading is probably the wrong fact and the engine is already 227k_15 — which makes the fix **one download** (`OldUnreal-UnrealPatch227k-SDK-Windows.zip`, 26.6 MB, which this project does not have) rather than an impossibility. `[hypothesis]` — and **one `Unreal.log` startup banner or `Core.dll` version-resource read settles it with no launch**, which is step 1. ✏️ Also re-scopes the 2026-09-02 row below: that "SDK bump not worth it" closure answered **fog/gamma** and is now misleading on **ABI** grounds. Corroboration that the ABI really did move: the SDK asset was renamed and roughly halved between the two releases (50.7 MB `SDK.zip` → 26.6 MB `SDK-Windows.zip`, with a new Linux split) — a restructure, not a patch. |
| 2026-09-02 | [v227k_12→v227k_15: what changed for fog/gamma, and does it matter to VRGoldDrv](topics/2026-09-02-sdk-227k15-vs-227k12-fog-gamma-changes.md) | ✅ incorporated — ⚠️ **its SDK-bump closure is RE-OPENED on ABI grounds 2026-09-05 (see the row above); the fog/gamma reasoning below stands unretracted** | **✅ 2026-09-03: the curve came from the SDK source on disk — `Gamma = Brightness × 2.0`, `GM_XOpenGL` identity at defaults, so stock applies NO gamma by default; our fixed 2.0 was `[disproved]` and fixed. Stock DX9 mode swaps the G and B exponents. Protocol fix adopted (record `GammaMode` + `Brightness` first).** Closes the 2026-08-24 SDK-bump question: the three releases past v227k_12 fix fog only in `D3D9Drv`/`OpenGLDrv`/`XOpenGLDrv` code this from-scratch renderer never touched, and add nothing gamma-related. But `ICBINDx11Drv`'s own `GammaMode` setting picks between two different gamma algorithms ("XOpenGL"/"DX9") — stock has no single canonical curve, which reframes `bGammaCorrectOutput`'s open comparison: record which `GammaMode` (and `GammaCorrectScreenshots`) was active in the stock screenshot before comparing. |
| 2026-09-02 | [UE1's world scale is nearer 45 UU per metre than 52.5 — the `StereoIPD=3.4` default is about 20 % too wide](topics/2026-09-02-ue1-world-scale-is-nearer-45-uu-per-metre-than-52-5.md) | ✅ incorporated | **✅ Folded into the dossier by the modding side 2026-09-03.** The Unreal Wiki gives Unreal/UT99 **13.6 UU/ft = 44.6 UU/m** (78-UU player), UT2003/4 50.3 UU/m, and calls 16 UU/ft (52.5 UU/m) the rule the *newer* games sit closest to. 64 mm on UE1 is ~2.85 UU, not 3.4. M3 still measures; this names the starting number and its source. **Corroborated by an independent `/sr` pass, 2026-09-02** — same figure, same primary source; addendum in the topic file adds a cheap pre-headset sanity check (78 UU player height vs. a doorway/NPC). |
| 2026-09-01 | [Head-look needs no C++: `PlayerCalcView` is a script event](topics/2026-09-01-playercalcview-is-a-script-event-so-head-look-needs-no-native-code.md) | ✅ incorporated | **✅ Folded into the dossier by the modding side 2026-09-03.** Answers §6's open "script side and/or render device?" question: **both, at different layers.** In OldUnreal's public 227 script source the view is produced by an `event` with three `out` parameters — script-implemented, not `native` — so a `PlayerPawn` subclass can own HMD orientation outright, and `ViewRotation` is `transient norepnotify` so writing it at HMD rate costs nothing. `WalkBob` is added **in that same script event**, so head-bob removal is one omitted line. Per-eye offset and projection stay in our renderer. §12's native bridge shrinks to publishing the pose, via a documented `ucc make -nobind` + `Bound to <Package>.dll` procedure — but the write-up is 227i/VS2008-era and silent on 64-bit. |
| 2026-09-01 | [The `ICBINDx11Drv` on GitHub is not the one in the SDK](topics/2026-09-01-the-icbindx11drv-on-github-is-not-the-one-in-the-sdk.md) | ✅ incorporated | **✅ 2026-09-03: "the SDK copy is the authority" is what made the gamma answer possible. No blank native template in the SDK, but `Emitter/` is a buildable script+native hybrid example.** A time-saver and a small win. Upstream's README says it "is only able to be built for UT 469 only" — 227 is a supported extension target, not what you get by cloning — so the SDK's copy is the authority when a 227-only detail matters. It also confirms render-to-texture is already a shipping capability of this engine's renderers, which de-risks a per-eye render target. And §5's worker-thread question has **no public answer at all**: settle it from the SDK source or at runtime, and stop searching. |

## How to add a topic

1. New file in `topics/`, named `YYYY-MM-DD-short-slug.md`.
2. One row added to the table above, newest at the top.
3. Update the status tag here as it moves through review → incorporated/dead-end (the modding side should update this when it acts on a lead, so the index reflects reality without the research side needing to poll).

## A note for this project specifically

A cross-project research sweep already found one Unreal Gold-relevant item worth following up:
**OldUnreal's SDK is at v227k_15 publicly** (three releases ahead of this project's v227k_12),
with rendering fixes (volumetric fog, D3D9Drv detail-texture flicker, OpenGLDrv distance-fog
compatibility) directly relevant to this project's open fog-calibration polish item — flagged in
`flat-to-vr-cross-engine-research`'s sweep log, 2026-08-24. Worth a first topic write-up here:
whether an SDK bump is worth it (write-up should weigh the OldUnreal changelog against this
project's own already-working from-scratch renderer, not just link the SDK release).
