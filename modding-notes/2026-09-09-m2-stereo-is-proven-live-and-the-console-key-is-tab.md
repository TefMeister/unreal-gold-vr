# M2 stereo is proven live — and the console key is Tab, not tilde

**2026-09-09, home PC (`RTX`), `/lm`, ONE launch. Game closed gracefully through its own console.**

Evidence: `dev-archive/recon/2026-09-09-vrgolddrv-m2-stereo-proven/` (8 captures, the full
`Unreal.log`, the harness, and the numbers).

## The headline

**`VRGoldDrv` loads, renders the game correctly, and `VRGOLD STEREO 1` produces genuine
side-by-side stereo with the HUD still full-width.** That is the M2 outcome the board has been
waiting on since 2026-09-02, and it was the *first* outcome on the row's own four-way list.

The whole sequence in one launch: attract demo renders → `open Vortex2` → live gameplay with HUD →
`VRGOLD STEREO 1` → two eyes → `IPD 20` → wider → `SWAPEYES 1` → halves exchange → `STEREO 0` →
back to mono, live, no relaunch. 80,156 frames, no driver warning or error in the log.
`[verified-live 2026-09-09, n=1 launch]`

## Why this is stereo and not a duplicated picture

The row warned that identical halves would mean a cbuffer layout bug, so it was checked with
numbers rather than by eye. Best horizontal alignment shift between the eyes, per depth band:

| band | IPD 2.85 (default) | IPD 20 |
| --- | --- | --- |
| upper (near ceiling beams) | +2 px | +23 px |
| middle (far wall) | +1 px | +8 px |
| lower (floor + near pillar) | +5 px | +38 px |

- **Never byte-identical** ⇒ not a duplicated image.
- **The shift varies by DEPTH** — near band +5 against mid +1 at the default, +38 against +8 at
  IPD 20 ⇒ real per-eye projection, not one frame offset twice.
- **It scales with IPD** — ~7x the separation gives ~7.6x the near-band shift ⇒ `StereoIPD`
  genuinely reaches the projection matrix.

`[verified-numerically 2026-09-09]` `SWAPEYES` checked the same way: afterwards the left half
matches the *previous right* half (1.16 mean abs diff) far better than its own previous left half
(6.22). `STEREO 0` returns to within 1.32 of the pre-stereo control.

⚠️ **NOT established:** anything about comfort, convergence, or whether it is *correct* stereo
rather than merely *present* stereo — the eye separation is right-handed and depth-varying, which
is what an import/export diff and a screenshot can show. Whether it fuses into a comfortable image
is a headset question, and the world scale is M3's job by design (`VRGoldDrv.cpp` says so).
⚠️ `n=1 launch`, one scene, one machine.

## ⭐ The thing that cost the most time, and will cost the next session nothing

**The console key on this install is `Tab`, not tilde.** `User.ini` line 49 is `Tab=Type` and line
168 is `Tilde=` — **unbound**. Pressing tilde did nothing at all.

That failed *silently and misleadingly*: with no console open, the typed command went to the game
as key bindings. `R=TeamTalk` (User.ini line 128) opened the chat prompt, and the remaining letters
landed in it, so the screen showed `TeamSay gold status` — which reads exactly like "the console
ate my command" rather than "the console never opened". Two rounds were spent on it.

This is the same family as `doom-2016-vr`'s scan-`0x29` keyboard-layout trap, and the general
lesson is worth stating once: **on UE1, read `User.ini` for the binding before assuming a key.**

## Two more automation facts about this engine

- **`Unreal.log` is 0 bytes while the game runs and complete the moment it exits.** The dossier
  already recorded the empty log as a dead end for reading the version banner; the useful half is
  that it is not empty, just *buffered*. **Every `VRGOLD` command and its reply is in there**
  (`Cmd:` / `Console:` line pairs), so the driver's output is fully readable — after the run, not
  during it. `VRGOLD STATUS` does answer; it simply prints nowhere on screen.
- **Capture with `BitBlt` from the screen DC**, as the standing primitive says. Confirmed fine here
  across menus and gameplay.

## Config changes made, and why

`System64\Unreal.ini`: `StartupFullscreen=True → False`, windowed viewport `1024x768 → 1280x720`.
Backup `Unreal.ini.bak-2026-09-09-pre-lm`. Two reasons, both deliberate: windowed makes capture
reliable, and `MACHINES.md` asks for a pinned **16:9** windowed size on this 21:9 machine so any
aspect-keyed number transfers to the dev PC. The engine wrote **no**
`[VRGoldDrv.VRGoldRenderDevice]` section back on exit, so stereo settings do not persist — every
launch starts mono at IPD 2.85.

## Automation scorecard (the five capabilities)

| capability | state |
| --- | --- |
| self-launch | ✅ proven — `Start-Process` on `System64\Unreal.exe`, window up in <12 s |
| menu → gameplay | ✅ proven — ESC from the attract demo, then console `open Vortex2` into a live level with HUD |
| commands | ✅ proven — **Tab** opens the console; all four `VRGOLD` commands accepted and answered |
| character + camera | ⚠️ **NOT exercised this session.** Scancode input is proven (it drove the console) but no walk or camera sweep was run |
| self-close | ✅ proven — console `exit`, graceful, full shutdown in the log, no `taskkill` |

Four of five. The gap is deliberate: the M2 question is answered by a still frame plus numbers, and
walking was not needed for it. It is the next session's cheap win, and it is now recorded in
`ai-game-control-profiles/profiles/unreal-gold.json`, which did not exist before today.
