# 2026-09-04 (`/lm`, dev PC, FULLY AUTONOMOUS) — the deployed VRGoldDrv.dll is ABI-incompatible with this PC's engine and will not load; the dev-PC flat rows are blocked on a rebuild

**The renderer never initialized.** Pre-flight set `GameRenderDevice=VRGoldDrv.VRGoldRenderDevice`
in `Unreal.ini` (backup taken) and launched `System64\Unreal.exe`. The game came up on a Windows
loader error and never rendered a frame. Evidence:
`dev-archive/recon/2026-09-04-vrgolddrv-abi-mismatch-uproperty-postload/`.

---

## 1. The failure

```
The procedure entry point ?PostLoad@UProperty@@UEAAXXZ could not be located in the
dynamic link library D:\nonSteam\UnrealGold\System64\VRGoldDrv.dll.
```

`?PostLoad@UProperty@@UEAAXXZ` is the MSVC-mangled `UProperty::PostLoad` (x64, `public: virtual void
__cdecl`). `VRGoldDrv` imports it from the engine's `Core`, and the deployed `Core` does not export
it — so the DLL cannot load, and with no render device the engine cannot start. `[verified-live
2026-09-04, n=1]`

**It is a version/ABI mismatch, not a broken file.** `VRGoldDrv.dll` is 198,144 bytes, a well-formed
PE32+ x86-64 DLL (verified `MZ`/`PE` header). The symbol is simply from a different build of the
engine than the one deployed here.

Likely cause: the driver was built against a different 227k sub-version's headers/import libs than
the `Core` in `System64\`. The dossier records the deployed build as 227 sub ~11; `MACHINES.md`
lists the dev-PC SDK as v227k_12. OldUnreal's engine is a private repo, and `UProperty`'s vtable /
exported surface can differ between sub-versions, which is exactly the kind of skew that produces a
single missing mangled export.

## 2. Why this matters beyond today

The board's dev-PC rows read "the DLL is now deployed here (there was none before)" and treat the
launch as ready. **Deployed is not the same as loads.** Until the driver and the running `Core`
agree on `UProperty` (and whatever else differs), none of the dev-PC flat tests can run:

- the blue-sky BGRA→RGBA channel-swap check,
- the `VRGOLD STEREO 1` console M2 proof,
- the gamma A/B against stock.

All three need a live `VRGoldRenderDevice`, which does not exist while the DLL won't load.

⚠️ The same check applies to the home PC: the `[FLAT @home]` M2 proof will hit this identical error
unless the home PC's engine build matches whatever `VRGoldDrv` was compiled against. Confirm the
running `Core.dll` sub-version on each machine before assuming the driver loads there.

## 3. The fix (`[PD]`, no game needed to do)

Rebuild `VRGoldDrv` against the **exact** engine deployed on the target machine:
- read the running `Core.dll` / `Engine.dll` sub-version in `System64\` (file version, or the
  `227` sub it reports), and
- link the driver against that sub-version's SDK headers/import libs, not merely "227k".

Then re-set the renderer line and re-launch. Success is the game reaching its menu on VRGoldDrv
(the blue-sky check is the first frame's verdict).

## 4. State left clean

- `Unreal.ini` reverted to stock `GameRenderDevice=ICBINDx11Drv.ICBINDx11RenderDevice`, so an
  ordinary launch works. (The pre-flight backup was removed after reverting.)
- The wedged process was force-terminated — see §5.

## 5. Automation on Unreal Gold, scored (§5a)

1. **Menu → gameplay: BLOCKED** — the engine never got a render device; no menu ever appeared.
2. **Commands: N/A** — never reached a console.
3. **Character + camera: N/A.**
4. **Self-close: had to FORCE-TERMINATE.** The failure is a modal MessageBox that respawns on
   dismissal (the loader retries the missing symbol), and there is no game/menu behind it. Graceful
   routes were genuinely attempted and all failed: `WM_CLOSE` to the main window (blocked by the
   dialog's modal loop), `WM_CLOSE` to the `#32770` dialog (closed it, another took its place),
   `SendKeys`/`AppActivate` Enter, and a click at the OK location. Only `taskkill /F` on that one
   PID ended it. This is the sanctioned last resort for an unrecoverable state, recorded here for
   honesty; it is not the normal close path.

## 6. What is NOT established

- Anything about VRGoldDrv's actual rendering (stereo, channel order, gamma) — it never ran.
- Whether the home PC has the same mismatch (must be checked there).
- The exact sub-version delta between the driver and the deployed Core (read both to confirm).

## 7. Gate

The dev-PC `[FLAT]` rows are blocked. Next is `[PD]`: rebuild `VRGoldDrv` against the deployed
engine's exact sub-version, then re-attempt the launch. Nothing needs the headset.
