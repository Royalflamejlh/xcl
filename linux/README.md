# XCL on Linux: the stack on `nvidia-dev`

The branch `nvidia-dev` with the pull requests Linux needs, the counterpart of `linux/stack` (which is on `main`). It is
a test branch, not a pull request. Measured on NVIDIA (RTX 4060 Ti 16 GB, driver 615.71.09), Wine 11.21 Staging.

Updated 2026-11-27.

## What it is

`nvidia-dev` `d1be8103` with these pull requests. They are `main`-based, so the merged ones also bring the `main`
commits `nvidia-dev` lacks.

| Pull request | Head | How it went in |
|---|---|---|
| #684 | `e56dbc3f` | merged |
| #1599 | `5b1518fc` | merged |
| #2082 | `cab89ed5` | merged |
| #2753 | `e07d03bf` | merged |
| #2551 | `3567b2ef` | merged |
| #794 | `bee31d56` | merged |
| #1462 | `813615c9` | merged |
| #1368 | `92ef1316` | merged |
| #1479 | `70364190` | merged |
| #2762 | `496ef578` | merged (one `host.rs` conflict, both sides kept) |
| #867 | `021b2c33` | ported (its commit; conflicts in `real.rs`, `list_x.rs`, `vram.rs`, `sparse.rs`, see the commit) |
| #2477 | `5225ddcb` | ported (its commit; three `real.rs` conflicts, see the commit) |
| #1803 | `cb007514` | already in `nvidia-dev` |

Left out, with the reason:

| Pull request | Why |
|---|---|
| #2404, #2633 | `nvidia-dev` has its own form: an indirect dispatch of a split program waits for the whole form, built off the call path (#2893); it has no chained ExecuteIndirect |
| #2707 | it reworks `alloc_pool::title_reset` around #2404's chain recordings; `nvidia-dev` has its own `title_reset` (it returns unified memory per PC allocator, #1358) |

Still to come: #751 and #2864 (being ported), #1860, #2229, #1975, #2070 (shader-side ports).

## vkd3d-proton

The same as `linux/stack`: stock vkd3d-proton `master` `71cae091` is the aim. Until the command-pool memory is fixed in
XCL, runs use vkd3d-proton v3.1 with dxil-spirv `694cc674` and the pool-release patch (`nvidia-dev` without #1860 needs
the newer dxil-spirv).

## How far it gets

Not run yet.
