Supersedes: `inbox/2026-09-05-gr-no-sdk-exists-for-subversion-11.md` — its ⭐ section, the `[hypothesis]` that "*subversion 11* is the misread fact" and that §1 and §9 are disagreeing with each other

# The engine prints BOTH "Subversion: 11" and "Compiled: Aug 15 2026" in one banner — so nothing was misread, and the two numbers are two different numberings

**Filed by `/gs`, 2026-09-07, home PC (twenty-fifth run).** Read-only sweep; nothing was edited and
nothing was launched. This is a read of a log file already on this machine's disk.

## What was measured

`C:\NonSteam\UnrealGold\System64\Unreal.log`, the home PC's install, log opened 2026-08-23 13:22:

```
Log: Log file open, 08/23/26 13:22:06
Init: Name subsystem initialized
Init: Version: 227 Subversion: 11
Init: Compiled: Aug 15 2026 13:48:22
...
Init: Computer: RTX
Init: User: TD3KX
```

`[verified-numerically 2026-09-07]` — copied verbatim from the file.

## Why this changes the ⭐ section of the 09-05 drop

That drop reasoned: 227k_11 shipped 2024-03-08, so its binaries cannot carry a 2026-08-15 build
date; therefore the deployed engine is probably **227k_15** and *"subversion 11"* in §1 is **the
misread fact**. It then split the fix into two branches, one of them impossible.

**The misread branch is gone.** §1 did not misread anything: the engine's own startup banner prints
`Subversion: 11` **and** `Compiled: Aug 15 2026` on consecutive lines, in the same run, on a
different machine from the one that wrote §1. Both facts are the engine's own report.

So the contradiction was never between §1 and §9 — it is between **two different numbering
schemes**: OldUnreal's internal `Subversion:` counter and the GitHub release tag's `_NN` suffix.
They are not the same number and should never have been compared.

## What that leaves, and it is the cheap branch

The home install was downloaded **2026-08-21** (`MACHINES.md` line 98). The binaries were compiled
**2026-08-15**. `v227k_15` was published **2026-08-16** — the only release between the compile date
and the download date. A download on 08-21 gets the newest release.

**So the deployed engine is almost certainly `v227k_15`, reporting internal subversion 11.**
`[hypothesis]` — three dates lining up, nothing measured against the release binaries themselves.

That is the 09-05 drop's own **trivial** branch: *"download the v227k_15 SDK and rebuild. The
project does not have this SDK — `MACHINES.md` records v227k_12, which is what the failing driver
was built against. One download."* The impossible branch ("genuinely 227k_11, no SDK exists,
upgrade the engine first") can be set aside unless the download disproves it.

⚠️ It also explains the ABI failure cleanly without any version-skew mystery: the driver was linked
against **v227k_12**'s Core and the deployed Core is **v227k_15**'s, which is exactly the kind of
gap that loses an export like `?PostLoad@UProperty@@UEAAXXZ`.

## Both machines look identical here, which matters for who does the work

`MACHINES.md` line 21 records the dev PC's game at `D:\nonSteam\UnrealGold` with SDK **v227k_12**;
its §1 entry records **227 sub ~11, engine built 2026-08-15** — the same pair this home-PC banner
prints. So this is not a per-machine skew and the rebuild does not have to happen on the machine
that will run it. `[inferred-static]`

## What the owner should do

1. Fold this and the 09-05 drop in together, and **delete both by name** — they finish together.
2. §9 / the 2026-09-04 ABI entry: drop "confirm the running sub-version and link against its
   matching SDK". Replace with: only **v227k_12** and **v227k_15** ship SDKs; the banner's
   `Subversion:` is not the release tag; the deployed build compiled 2026-08-15 and is almost
   certainly v227k_15.
3. §1: keep "227 subversion 11" — it is what the engine says — but annotate that it is the
   **internal** counter, not the release suffix, so nobody re-derives the contradiction.
4. The `[PD]` row stays `[PD]`: one SDK download plus a rebuild, still no launch needed to get that
   far. Confirming it loads is `[FLAT]`, and unchanged.

Lane: /gs
