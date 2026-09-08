# 2026-09-08 — evidence for the VRGoldDrv ABI fix (v227k_12 → v227k_15)

`/pd`, dev PC. **No launch.** Everything here was produced from files on disk with
`llvm-objdump` / `llvm-nm`. Write-up:
`modding-notes/2026-09-08-the-abi-blocker-was-one-symbol-wide-and-vrgolddrv-is-rebuilt-against-v227k_15.md`

| file | what it is |
| --- | --- |
| `check-abi-before-rebuild.log` | the old v227k_12 driver: **1 MISSING**, `?PostLoad@UProperty@@UEAAXXZ` — the 2026-09-04 loader error, reproduced statically |
| `check-abi-after-rebuild.log` | the v227k_15 rebuild: **0 MISSING** across Core (100) and Engine (23) |
| `imports-old-v227k_12-build.tsv` | every symbol the old driver imported, per source DLL |
| `imports-new-v227k_15-build.tsv` | the same for the rebuild. `diff` the two: **exactly one symbol changes**, `PostLoad@UProperty` → `PostLoad@UField` |
| `sdk-v227k_12-Core.lib.symbols.txt` | v227k_12 import-library symbols — **contains** `?PostLoad@UProperty@@UEAAXXZ` |
| `sdk-v227k_15-Core.lib.symbols.txt` | v227k_15 — **does not**, matching the deployed engine. This is what identifies the deployed release |
| `deployed-Core.dll.exports.txt` | the deployed engine's export names |

All of these are **interface metadata we generated** (export-name dumps), not game content: no
engine code, no assets, no binaries.

The one thing none of it shows is whether the driver renders. It shows the loader will not reject
it, which is the failure that happened, and no more.
