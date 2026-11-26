# Blue-Shift engine (EA / Blue Shift Inc.)

Games on this page:

| Game | Serial | Region | Engine version |
| --- | --- | --- | --- |
| Marvel Nemesis: Rise of the Imperfects | SLUS-21281 | NTSC-U | V1 |
| Marvel Nemesis: Rise of the Imperfects | SLES-53585 | PAL | V1 |
| Marvel Nemesis: Rise of the Imperfects | SLKA-25311 | NTSC-K | V1 |
| Marvel Nemesis: Rise of the Imperfects (Demo) | SLUS-29156 | NTSC-U | V1 |
| ESPN Major League Baseball | SLUS-20794 | NTSC-U | V1 |
| World Series Baseball 2K3 | SLUS-20611 | NTSC-U | V0 |

Engine versions (V0/V1) are as given in PCSX2 issue #4484.

## Why it broke

- The engine needs the PS2 main CPU (EE) and the two vector units (VU0, VU1)
  to stay in very tight step with each other.
- Emulators run these parts with different timing than real hardware. The games
  show severe timing problems once gameplay starts.
- Older workarounds (EE cycle rate +3, MTVU off, Instant VU1 off, VU0 Kickstart)
  only helped partly.
- V0 (World Series Baseball 2K3) also uses a cache mode that PCSX2 does not
  implement, and crashes right away with the EE interpreter.
- Official PCSX2 closed the report as "won't fix".

Source: [PCSX2 issue #4484](https://github.com/PCSX2/pcsx2/issues/4484)

## What fixed it (Marvel Nemesis)

The fixes are by AITUS95 in pcsx2-reliquary (Oct 2026). They are GameDB-only
game fixes, switched on per game:

| Game fix | What it does | Problem it solves |
| --- | --- | --- |
| `VUCommunicationHack` | Keeps the exact instruction-pair boundaries where the EE starts VU programs and reads/writes shared VU registers. The fast recompiler (JIT) used to run past these points. | Displaced polygons. This is the main fix. |
| `VU1MaddiHack` | Precise VU1 MADDi math, without turning on full soft-float. | Floor texture flicker. |
| `GSZeroStrideHack` | Keeps the order of overlapping framebuffer writes that use zero stride. | Rendering errors from out-of-order writes. |
| `GSCoronaDepthHack` | Stops light and fog quads from writing depth, so they don't hide later layers. Narrowly matched to Marvel Nemesis materials. | Rectangle light artifacts and diagonal gaps in fog. |

How the main bug was found: the author built a slow, accurate reference using
the EE recompiler plus both VU interpreters with coordinated timing. Comparing
the fast JIT against that reference showed where the communication order was
being broken.

Note: with `VUCommunicationHack`, multi-threaded VU1 (MTVU) runs in a special
synchronized mode instead of fully async.

Reported speed after the fixes: one user went from 45–55% to 85–90% (Thread
Pinning off). The author measured about 99%, with brief dips. 100% is not
guaranteed on every PC.

Sources:
- [pcsx2-reliquary PR #1: VU communication boundaries](https://github.com/AITUS95/pcsx2-reliquary/pull/1)
- [pcsx2-reliquary PR #4: automatic fixes, performance](https://github.com/AITUS95/pcsx2-reliquary/pull/4)
- [pcsx2-reliquary PR #5: light and fog depth](https://github.com/AITUS95/pcsx2-reliquary/pull/5)

## Experiment in this fork: the baseball games

ESPN Major League Baseball uses the same V1 engine as Marvel Nemesis.
World Series Baseball 2K3 uses V0. This fork gives both games the first three
fixes (`VUCommunicationHack`, `VU1MaddiHack`, `GSZeroStrideHack`) to test if they
help. `GSCoronaDepthHack` was left off because it is matched to Marvel Nemesis
materials only.

Expectation before testing: ESPN MLB has a real chance. WSB 2K3 probably still
fails because of the missing cache mode.

## Test results

*Not tested yet. This section is filled in only from real tests.*

Test settings: release build v0.0.1 or newer, MTVU on, Thread Pinning off,
other settings default.

| Game | Serial | Build | Boots? | Plays a full match? | Speed % | Graphics OK? | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Marvel Nemesis | SLUS-21281 | | | | | | |
| Marvel Nemesis | SLES-53585 | | | | | | |
| Marvel Nemesis | SLKA-25311 | | | | | | |
| Marvel Nemesis (Demo) | SLUS-29156 | | | | | | |
| ESPN Major League Baseball | SLUS-20794 | v0.0.1 | Yes | Reaches gameplay | 53–59% (ParallelGS), 67% (Vulkan) | No: black jagged polygons near the camera | Menus and videos also slow (~53%). Thread Pinning on: no change. GPU only ~9% busy. Froze on exit. |
| World Series Baseball 2K3 | SLUS-20611 | | | | | | |

Test machine: Intel i7-12700K, NVIDIA RTX 3070, CachyOS Linux. Tested 2026-11-26. BIOS: SCPH-70012 (USA v2.00).

### Comparison with official PCSX2

Same game, same settings (default + Vulkan), official PCSX2 v2.8.2 (no Blue-Shift fixes):

| Build | Menus | Entering a game | Black polygons |
| --- | --- | --- | --- |
| Official PCSX2 v2.8.2 | Full speed (60 fps) | Froze before any 3D was shown | n/a (never reached 3D) |
| This fork v0.0.1 | ~53% | Reaches gameplay | Yes, near the camera |

Result: the Marvel Nemesis fixes are what let ESPN Major League Baseball reach gameplay. They cost a lot of speed, even in menus. The black polygons are a separate bug still to find.

## Notes for recomp projects

*To be written after testing.*
