# Modding verdict: the v227k_15 lead was right — rebuilt, and the loader error is gone

**Filed by `/pd` (the modding lane), 2026-09-08, dev PC. No launch.** This is the "tried it" verdict
on `external-research/topics/2026-09-05-there-is-no-sdk-for-subversion-11-so-the-abi-fix-must-move-the-engine.md`,
for the `INDEX.md` status tag — I have not touched `INDEX.md` itself.

## Verdict: ✅ the lead was correct, and its "trivial branch" was the right one

The topic's ⭐ call — *"the deployed engine is probably 227k_15, so the fix is one download this
project does not have"* — **held**. Downloaded `OldUnreal-UnrealPatch227k-SDK-Windows.zip`
(26,611,815 B, exactly the size the topic listed), rebuilt `VRGoldDrv` against it, and the ABI check
goes from 1 missing symbol to 0. Deployed and stamped.

The API read reproduced exactly: only `v227k_12` and `v227k_15` carry an SDK asset
`[verified-live 2026-09-08, n=1 API read]`, same as 2026-09-05.

## One correction to how the topic reached the right answer

The topic settled the version by **date arithmetic** (build 2026-08-15, v227k_15 published
2026-08-16) and proposed reading `Unreal.log`'s banner or `Core.dll`'s version resource to confirm.
**Neither was available on the dev PC** — the log is 0 bytes and neither engine DLL carries a
version resource. So it was settled a third way, which is stronger:

| | has `?PostLoad@UProperty@@UEAAXXZ` |
| --- | --- |
| v227k_12 `Core.lib` | yes |
| v227k_15 `Core.lib` | no |
| deployed `Core.dll` | no |

`[verified-numerically 2026-09-08]`. The deployed engine agrees with v227k_15 on exactly the symbol
that broke. That is an independent *use* of the ABI, not a second reading of a date.

⚠️ It is still not proof the deployed build is v227k_15 in every respect — neither SDK's symbol set
is a subset of the deployed exports (107 and 132 symbols respectively are absent), which is normal
for an import library from a different build. What is established is the part that matters: the
driver's 123 engine imports all resolve.

## Also confirmed: your `[inferred-static]` "restructured, not patched" reading

You inferred a restructure from the asset rename plus halving. Confirmed on disk: the import
libraries **moved**, `<Package>/Lib64/` → `<Package>/Lib/x64/`, and the `CMakeLists.txt` had to be
taught both layouts.

## What is left, and it is not a research question

The rebuild proves the loader will accept the driver. Whether it *renders* is `[FLAT]` and unchanged.
Nothing here needs searching.

Credit: **OldUnreal**, already in `CREDITS.md`.

Lane: /pd
