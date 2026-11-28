# XCL on Linux: the stack

The branch that gets XCL furthest on Linux (Wine + vkd3d-proton): `main` plus the open pull requests Linux needs.
The aim is to run on stock vkd3d-proton: `master` `71cae091` (2026-11-25) with its own dxil-spirv `b540c8a2`, no
patch, and the next release once it ships. Every fix goes in XCL, nothing in the layers. It moves as pull requests merge or new ones help. Measured on NVIDIA (RTX 4060 Ti 16 GB, driver
615.71.09); AMD / RADV not tested.

Updated 2026-11-27.

## What it is

`main` `b49be18` with these pull requests merged, in this order, at these heads. #2477 (with #867) sits on `main` `f9a0cb80`,
so its merge brings `main` up to `f9a0cb80`.
`main` `753e8b1a` (2026-11-27, with #3057) is merged on top. Run its launch pass with `XCL_PREPARE_WORKERS=8` until #3085 is in a run here.
`main` `bb9a6b4b` (2026-11-28, with #2959, #2872, #3085, #3086 and #1809) is merged on top of that. #2959 replaced the chained
`ExecuteIndirect` of #2404 and #2633 and the allocator reclaim of #2707, so those three leave the table. A split
program's `ExecuteIndirect` now runs its pieces only once its indirect set (pieces and whole form on the CBV root, built on
one background thread) is ready; until then it runs the whole form, built on the title's thread (#2404 ran the pieces
without it). #751's commit has been in `main` since #2844 (`fa671a2f`), so it leaves the table too.

| Pull request | Head | What it does on Linux |
|---|---|---|
| #684 | `e56dbc3f` | fixes the #662 crash in `xg::copy_from` |
| #1599 | `5b1518fc` | DirectStorage requests failing at the world load |
| #1860 | `bb2165b9` | dxil-spirv crashing on XCL's direct DXIL |
| #2082 | `cab89ed5` | dxil-spirv writing SPIR-V `spirv-val` rejects |
| #1803 | `cb007514` | heap addresses the world's shaders read |
| #2229 | `98a629a3` | the constant data that left the world flat grey |
| #1975 | `72960386` | (with the two above) the world in place of grey (its mesh barrier guard is left out: `main`'s wider one, 2ca91276, covers it) |
| #2753 | `e07d03bf` | 3-D LUT pages: no white overexposure |
| #2551 | `3567b2ef` | XCL's pipeline blob cache (the precompile builds every split program's whole form; `main` builds it only for those an earlier run used through `ExecuteIndirect`) |
| #2070 | `d6bd4170` | 64 MB stacks for offline pipeline builds (a dxil-spirv stack overflow) |
| #1368 | `92ef1316` | pipelines built with `NodeMask` 0, so vkd3d-proton takes the offline pass's blobs (#2453) |
| #2762 | `496ef578` | one texture per view-by-address, not one per padding pattern (#2743) |
| #2864 | `68cce725` | `XCL_DSTORAGE_CACHE`, an opt-in cache for DirectStorage reads |
| #1479 | `70364190` | under Wine the final `report.json` is written at the title's exit (#1461) |
| #688 | `760f742b` | a private heap per DXC compiler: the launch pass under Wine is no longer held on one shared heap |
| (ours) `arch/window-title-commit` | `ad88ffc6` | the build's commit in XCL's window title, log and `report.json`; not a pull request yet |
| #867 | `021b2c33` | keeps vkd3d-proton's native vtables and wraps the title's fences (#175) |
| #2477 | `5225ddcb` | patches the assert-checked vtables in place under Wine (#175) |
| #794 | `bee31d56` | hides the title's own 1920x1080 window, which Wine does not cloak (it showed white behind XCL's) |
| #1462 | `813615c9` | the e2e title's cloak check accepts that hidden window |
| #3154 | `bacb6f04` | compute programs with a static sampler get direct DXIL again: `main`'s DispatchX prologue (`d9403543`) sent 567 of the title's 1,636 to HLSL |
| (ours) | `c49ab7c9` | the same for a compute program's scratch (offered on #3154) |
| (ours, stack only; #3189) | `b7f0f86f` | multidimensional compute groups take direct DXIL again (the direct path has the console-linear reorder, `lanes.rs`); their HLSL builds reached 43.8 and 45.9 GB in single launch-pass workers under Wine. #3189 is the same change for `main` |
| #3090 | `b3e67617` | keeps barriers carrying console hint flags (cherry-picked from its `nvidia-dev` branch, with `nvidia-dev`'s `runner::device` for its test) |
| #3091 | `64aa28f0` | derived pipelines keep the shared bytecode (cherry-picked from its `nvidia-dev` branch) |
| #3092 | `d40a1540` | a 2 MiB grain for allocator page size two (cherry-picked from its `nvidia-dev` branch, with `nvidia-dev`'s `Uma::block_len` for its test) |

Merge conflicts were resolved by keeping both sides, except where `main` already does what the pull request did; see each merge commit.

## vkd3d-proton

Not there yet: the results below were measured on vkd3d-proton `v3.1-90-g30b0eb9b` with three local patches.
Each one is being replaced by a change in XCL:

| Patch | What it covers | Replaced by |
|---|---|---|
| relaxed vtable checks | vkd3d-proton refuses objects whose vtable XCL replaced (#175) | #867 + #2477, merged here: on stock `master` no refusal or assertion |
| command pools release at `Reset` | without it the title ran out of RAM at about 2 minutes (#2404); 48.9 GB by 130 s on this stack | an XCL change, being measured |
| a local dxil-spirv patch | programs dxil-spirv crashed on | `master`'s dxil-spirv `b540c8a2` converts all of XCL's direct programs; v3.1's (`8b48f0e0`) crashes on most of them |

## Running it

`VKD3D_CONFIG=ignore_shared_fence` (the title's shared fences never complete under Wine) and
`XCL_DIRECT_DXIL=1 XCL_HUGE_PSO_WANTED_CONCURRENCY=6 XCL_DXC_THREADS=4 XCL_PSO_DISK_CACHE=1`. `XCL_PREPARE=0` leaves out
the launch pass `main` runs by default since #2555; the runs below had none.
Warm the driver's shader cache first (an offline `xcl shaders --precompile --only cs --pipelines` pass): on a
cold cache the title spends minutes with no picture while the driver builds pipelines.

## How far it gets

On vkd3d-proton `v3.1-90-g30b0eb9b` with the local patches (`8df733bd`, 1 run, caches seeded from an offline pass): the world
load at 68.0 s, black from 532 s (#505's draw), the movement hint over it at 603–651 s, presents to 851 s; then a
compute pipeline build started at 843 s had not finished at 901 s.

On stock vkd3d-proton `master` (1 run, `XCL_PREPARE=0 XCL_SKIP_BINK=1`): the world load at 57.5 s and presents to
90 s, when the run was stopped at 30 GB of RSS: the command-pool memory above, still to be fixed in XCL.

Measured before #867 and #2477 came in, with the vkd3d-proton patches. On the RTX 4060 Ti: the world load at about
53 s, the intro to about 520 s (colour banding), then black frames from about 529 s, with the movement hint at about
590 s drawn over them. 2 of 3 runs of 900 s stopped late (685 and 793 s) on a compute pipeline build that never
finished; one ran to the end. The black is the tile-grid draw traced on #505: with that draw's writes turned off the
scene showed (#505, 7223345779). It peaks around 34 GB of system RAM. On an RTX 4070 12 GB the same tree stopped
drawing at the world load. Reports: #175, #505, #2404, #2899.
