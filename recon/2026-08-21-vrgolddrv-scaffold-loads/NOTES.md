# 2026-08-21 — VRGoldDrv scaffold loads into the engine (home PC)

First live proof of the from-scratch render device. `vrgold-loadtest.log` is the
engine log from a 20-second boot of `System64\Unreal.exe` (227k subversion 11)
with `GameRenderDevice=VRGoldDrv.VRGoldRenderDevice`.

Key lines:

```
Log: Bound to VRGoldDrv.dll
Init: VRGoldDrv: Init 1024x768 colorbytes=4 fullscreen=1
...
Init: Game engine initialized
Log: Startup time: 1.181502 seconds
ScriptLog: Creating root window: UMenu.UMenuRootWindow
```

The engine bound our DLL, accepted the device, initialized audio and the game
engine, and built the menu root window with no crash. The device draws nothing
at this stage, by design.

Build: standalone CMake (VS 2022 Build Tools, x64 Release) against the official
OldUnreal 227k SDK (`OldUnreal-UnrealPatch227k-SDK.zip` from the Unreal-testing
v227k_12 release). Source lives in `unreal-gold-vr-staging/VRGoldDrv/`.

Incidental findings:

- The fresh install's `System64\Unreal.ini` was a 32-byte stub (only a `[URL]`
  section) — the same gutted-ini trap documented in the dossier §11. Repaired by
  copying `Default.ini` over it and setting `FirstRun=227`; the stub is kept as
  `Unreal.ini.stub-bak` next to it.
- The installed game reports 227 subversion 11; the SDK came from the 227k_12
  release. Binding and init worked regardless — no observed incompatibility at
  this stage.
- A `LOG=file.log` command-line argument is also parsed as the launch URL by
  227 (harmless "Can't find host LOG=..." warnings in the log); pass a map name
  first if this matters later.
- After the test, the INI was restored to stock
  `ICBINDx11Drv.ICBINDx11RenderDevice` and a 15-second sanity boot confirmed
  the game runs normally (DX11 shaders compile, no criticals).
