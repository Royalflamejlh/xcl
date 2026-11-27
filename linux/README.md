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
| #1860 | `bb2165b9` | ported (`port/1860-nvidia-dev` `b20a2395`) |
| #2229 | `98a629a3` | ported (`port/2229-nvidia-dev` `04fddf0a`); replaces `nvidia-dev`'s earlier copy (`41f1a85a`) with the head |
| #1975 | `72960386` | ported (`port/1975-nvidia-dev` `35852a5d`); its mesh barrier guard left out: `nvidia-dev` sends every multi-wave barrier to HLSL (`fb55e49e`) |
| #2070 | `d6bd4170` | ported (`port/2070-nvidia-dev` `3ae16650`) |
| #751 | `59f8e7c9` | ported by hand (`port/751-nvidia-dev` `eebbbf9f`): `nvidia-dev` already copied as #751 does (`copy_readable`) but still read title memory in place; `with_title_bytes` kept, `title_bytes` and `XCL_TEXTURE_SAFE_READ` removed |
| #2864 | `68cce725` | ported (`port/2864-nvidia-dev` `979fbb13`; only the README conflicted) |
| #1803 | `cb007514` | already in `nvidia-dev` |

Left out, with the reason:

| Pull request | Why |
|---|---|
| #2404, #2633 | `nvidia-dev` has its own form: an indirect dispatch of a split program waits for the whole form, built off the call path (#2893); it has no chained ExecuteIndirect |
| #2707 | it reworks `alloc_pool::title_reset` around #2404's chain recordings; `nvidia-dev` has its own `title_reset` (it returns unified memory per PC allocator, #1358) |

## vkd3d-proton

The same as `linux/stack`: stock vkd3d-proton `master` `71cae091` is the aim. Until the command-pool memory is fixed in
XCL, runs use vkd3d-proton v3.1 with dxil-spirv `694cc674` and the pool-release patch (`nvidia-dev` without #1860 needs
the newer dxil-spirv).

## How far it gets

On vkd3d-proton `master` `71cae091` unpatched (`e4b6b640`, before #2864, 1 run, 300 s, empty driver caches): the world load
at 75.7 s, presents to 110.3 s, then none while cold driver pipelines were built; one Xid 109 (a GPU context-switch timeout)
in that stretch, no device removal to the end; RSS flat at about 14 GB (no command-pool growth). `nvidia-dev` + #684 alone,
the same way on the patched v3.1 it needs: the world load at 58.5 s, presents to 98.2 s, the device removed at 243.7 s.
