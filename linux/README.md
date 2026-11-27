# XCL on Linux: the stack

The branch that gets XCL furthest on Linux (Wine + vkd3d-proton) as of the date below: `main` plus the open
pull requests that Linux needs, and the vkd3d-proton patches it still runs with. It moves as pull requests
merge or new ones help. Measured on NVIDIA (RTX 4060 Ti 16 GB, driver 615.71.09); AMD / RADV not tested.

Updated 2026-11-27.

## What it is

`main` `b49be18` (before #2555's launch pass) with these pull requests merged, in this order, at these heads:

| Pull request | Head | What it does on Linux |
|---|---|---|
| #684 | `e56dbc3f` | fixes the #662 crash in `xg::copy_from` |
| #751 | `59f8e7c9` | reads outside the coherent window |
| #1599 | `5b1518fc` | DirectStorage requests failing at the world load |
| #1860 | `bb2165b9` | dxil-spirv crashing on XCL's direct DXIL |
| #2082 | `cab89ed5` | dxil-spirv writing SPIR-V `spirv-val` rejects |
| #2404 | `5b9d4ff6` | split chained ExecuteIndirect |
| #2633 | `db34184f` | Dispatch-only signatures for those chains |
| #2707 | `dff6edc7` | releases recordings in place of #2404's `free_aside` sweep (no device loss) |
| #1803 | `cb007514` | heap addresses the world's shaders read |
| #2229 | `d2dee320` | the constant data that left the world flat grey |
| #1975 | `72960386` | (with the two above) the world in place of grey |
| #2753 | `e07d03bf` | 3-D LUT pages: no white overexposure |
| #2551 | `3567b2ef` | XCL's pipeline blob cache |
| #2070 | `8cb43460` | 64 MB stacks for offline pipeline builds (a dxil-spirv stack overflow) |

Merge conflicts were resolved by keeping both sides; see each merge commit.

## vkd3d-proton

`v3.1-90-g30b0eb9b`, built with:

- `vkd3d-proton-xcl-relaxed.patch` (here): accepts XCL's replaced vtables (#175). #2477 (with #867) is the
  XCL-side fix that removes the need for it on assert-checked builds.
- `vkd3d-proton-pool-reset-release.patch` (here): command pools release their memory at
  `ID3D12CommandAllocator::Reset`. Without it the title ran out of RAM at about 2 minutes (#2404, 7189698499).
- a local dxil-spirv patch (not here).

## Running it

`VKD3D_CONFIG=ignore_shared_fence` (the title's shared fences never complete under Wine) and
`XCL_DIRECT_DXIL=1 XCL_HUGE_PSO_WANTED_CONCURRENCY=6 XCL_DXC_THREADS=4 XCL_PSO_DISK_CACHE=1`.
Warm the driver's shader cache first (an offline `xcl shaders --precompile --only cs --pipelines` pass): on a
cold cache the title spends minutes with no picture while the driver builds pipelines.

## How far it gets

On the RTX 4060 Ti: the world load at about 53 s, the intro to about 520 s (colour banding), the movement
hint at about 590 s. Frames after it were black, and 2 of 3 runs of 900 s stopped late (685 and 793 s); one
ran to the end. The black is the tile-grid draw traced on #505. It peaks around 34 GB of system RAM. On an
RTX 4070 12 GB the same tree stopped drawing at the world load. Reports: #175, #505, #2404, #2899.
