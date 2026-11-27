# XCL on Linux: the stack

The branch that gets XCL furthest on Linux (Wine + vkd3d-proton): `main` plus the open pull requests Linux needs.
The aim is to run on stock vkd3d-proton: `master` `71cae091` (2026-11-25) with its own dxil-spirv `b540c8a2`, no
patch, and the next release once it ships. Every fix goes in XCL, nothing in the layers. It moves as pull requests merge or new ones help. Measured on NVIDIA (RTX 4060 Ti 16 GB, driver
615.71.09); AMD / RADV not tested.

Updated 2026-11-27.

## What it is

`main` `b49be18` with these pull requests merged, in this order, at these heads. #2477 (with #867) sits on `main` `f9a0cb80`,
so its merge brings `main` up to `f9a0cb80`.

| Pull request | Head | What it does on Linux |
|---|---|---|
| #684 | `e56dbc3f` | fixes the #662 crash in `xg::copy_from` |
| #751 | `59f8e7c9` | reads outside the coherent window |
| #1599 | `5b1518fc` | DirectStorage requests failing at the world load |
| #1860 | `bb2165b9` | dxil-spirv crashing on XCL's direct DXIL |
| #2082 | `cab89ed5` | dxil-spirv writing SPIR-V `spirv-val` rejects |
| #2404 | `5b9d4ff6` | split chained ExecuteIndirect (its newer head fixes the sweep #2707 replaces here) |
| #2633 | `19be6590` | Dispatch-only signatures for those chains |
| #2707 | `1a3b6981` | releases recordings in place of #2404's `free_aside` sweep (no device loss) |
| #1803 | `cb007514` | heap addresses the world's shaders read |
| #2229 | `98a629a3` | the constant data that left the world flat grey |
| #1975 | `72960386` | (with the two above) the world in place of grey |
| #2753 | `e07d03bf` | 3-D LUT pages: no white overexposure |
| #2551 | `3567b2ef` | XCL's pipeline blob cache |
| #2070 | `d6bd4170` | 64 MB stacks for offline pipeline builds (a dxil-spirv stack overflow) |
| #1368 | `92ef1316` | pipelines built with `NodeMask` 0, so vkd3d-proton takes the offline pass's blobs (#2453) |
| #867 | `021b2c33` | keeps vkd3d-proton's native vtables and wraps the title's fences (#175) |
| #2477 | `5225ddcb` | patches the assert-checked vtables in place under Wine (#175) |
| #794 | `bee31d56` | hides the title's own 1920x1080 window, which Wine does not cloak (it showed white behind XCL's) |
| #1462 | `813615c9` | the e2e title's cloak check accepts that hidden window |

Merge conflicts were resolved by keeping both sides; see each merge commit.

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
