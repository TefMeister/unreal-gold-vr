# The home PC now has the right SDK, and the loader will accept the driver there too

**2026-09-09, home PC (`RTX`), no launch.**

## The row said "the home PC needs the same rebuild first". It was right, and the failure was reproduced before fixing it.

`check-abi.sh` was pointed at the home PC's **installed** driver and its **own** `System64`:

```
imports: 207 symbol(s) across 6 DLL(s)
  Core.dll    100 imported, 2846 exported, 1 MISSING
      ?PostLoad@UProperty@@UEAAXXZ
  Engine.dll   23 imported, 4770 exported, 0 MISSING
```

That is the 2026-09-04 loader error exactly, one symbol wide, on a machine where nobody had
checked. `[verified-numerically 2026-09-09]` The deployed DLL was 202,752 B — the old v227k_12
build — so the `[FLAT @home]` M2 stereo test could not have run there whatever else was true. It
would have died at load with a message about a symbol, not rendered anything wrong, which is the
kind of failure that costs a headset session to discover.

## Which SDK, settled the same way the dev PC settled it

Not by dates. By whether the symbol is present:

| SDK on this machine | `?PostLoad@UProperty@@UEAAXXZ` in `Core.lib` | verdict |
| --- | --- | --- |
| `C:\Users\TD3KX\ue1-sdk-227k` (`Core/Lib64/Core.lib`) | present | v227k_12 |
| `C:\Users\TD3KX\ue1-sdk-227k_15` (`Core/Lib/x64/Core.lib`) | absent | v227k_15 |

`[verified-numerically 2026-09-09]` The library path itself is a second, independent tell — the SDK
moved `<Package>/Lib64/` to `<Package>/Lib/x64/` between the two, which is why `CMakeLists.txt`
probes both. Both SDKs are kept; they are not interchangeable.

Installed from `OldUnreal-UnrealPatch227k-SDK-Windows.zip` (26.6 MB) on the OldUnreal
`Unreal-testing` **v227k_15** release.

## Result

Rebuilt with `cmake -B build -A x64 -DUE1_SDK_ROOT=C:/Users/TD3KX/ue1-sdk-227k_15`, Release:

- `VRGoldDrv.dll` `01d7ccb7c30e…`, **198,144 B — the same size as the dev PC's build**;
- `check-abi.sh` against this machine's `System64`: **0 missing**, across `Core.dll` (100) and
  `Engine.dll` (23);
- `/Brepro` confirmed working here too — two builds, **0 differing bytes**;
- deployed, with `VRGoldDrv.dll.bak-2026-09-09-pre-v227k15` beside it.

⚠️ The **hash** is not the dev PC's `304f2d2ce96d`. `/Brepro` makes a build reproducible for a given
toolset, and the two machines are on different MSVC builds. Same source, same size, same import
set — this is the one project in the estate where a cross-machine hash comparison is not expected
to hold, and the size plus the import-set match is the check that does.

## What this does and does not establish

**Does:** the Windows loader will not reject the driver on the home PC for a missing entry point.
**Does not:** that it renders. A signature change behind an unchanged mangled name is invisible to
an import/export diff, and no launch has happened. The `[FLAT @home]` M2 stereo row is now
genuinely runnable there; it is still unrun.

⚠️ **Path note for anyone scripting this:** Unreal Gold is at `C:\nonSteam\UnrealGold` on the home
PC, not `D:\nonSteam` as on the dev PC. `check-abi.sh` defaults its `SYS` argument to the dev PC's
path, so it must be passed explicitly on the home machine or it silently checks nothing useful.
