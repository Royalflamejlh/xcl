# XCL on Linux: the stack on `amd-dev`

`amd-dev` plus the open pull requests Linux needs, as `linux/stack` carries them on `main`. **AMD is not tested here:**
the branch is built and run on NVIDIA only (RTX 4060 Ti 16 GB, driver 615.71.09, Wine + vkd3d-proton); nobody has run it
on an AMD card or on RADV.

Updated 2026-11-27. Not run on the title yet: it builds (lib and tests); under Wine on the CPU the ported pull requests'
lib tests pass (62 of 62), and the GPU tests of #1368 and #2551 were skipped there (no card).

## What it is

`amd-dev` `7c795f5e` with these on top. `amd-dev` does not hold `main` (34 commits behind today), so a pull request made
on `main` is merged where that brings no other commit along, and cherry-picked (its author kept) where it would.

| Pull request | Head | How it went in |
|---|---|---|
| #2082 | `cab89ed5` | merged |
| #794 | `bee31d56` | merged |
| #1462 | `813615c9` | merged |
| #1479 | `70364190` | merged |
| #1860 | `bb2165b9` | merged as its `amd-dev` copy `linux/direct-single-latch-amd-dev` `4f590fb1`: its own test's inner loop reads a second input, as `amd-dev`'s branch facts fold the original shape away |
| #1368 | `92ef1316` | cherry-picked, clean |
| #2864 | `68cce725` | cherry-picked, clean |
| #684 | `e56dbc3f` | cherry-picked; both sides' tests kept in `xg.rs` |
| #2551 | `3567b2ef` | cherry-picked; `make_one` takes the record on `amd-dev`, and the launch pass's own whole-form call stays |
| #2753 | `e07d03bf` | cherry-picked; `place()` on `amd-dev` keeps the XG texture it laid out, so the size comes from that one, with the same fall-back for a 3-D target |

Already in `amd-dev`: #1803 (its head `cb007514` is an ancestor).

Left out, because `amd-dev` has the fix in a form of its own:

| Pull request | What `amd-dev` has |
|---|---|
| #751 | `6d7dcbb9` "read non-UMA title_bytes safely via read_committed": the same condition (borrow inside the coherent window, copy elsewhere) |
| #1599 | `40f499a7` defers the half-full auto-submit for heap page-array batches, and `a0442743` reads a physical-page list again when its request runs |
| #2762 | `c093211e` "a placed view's padding does not make a second texture" |
| #2404 | its own chained ExecuteIndirect: `77453544`, `fb81714d`, `6a0bb5b4` and later (`real/dispatch_chain.rs`) |

Not ported yet (each conflicts with `amd-dev`'s own code and needs a port, not a merge):

| Pull request | Why it is wanted | State |
|---|---|---|
| #867 + #2477 | stock vkd3d-proton: native vtables kept, fences wrapped, assert-checked vtables patched in place | 11 hunks in `real.rs`, `sparse.rs`, `vram.rs` |
| #2707 | the allocator pool's reset races (no device loss on `linux/stack`) | its first commit is in `amd-dev` (`78799549`); the other five conflict in `alloc_pool.rs`, `bind.rs`, `real.rs` |
| #2633 | split programs' pieces read their slab from a root CBV | 33 hunks against `amd-dev`'s own chain code |
| #2229, #1975, #2070 | the shader-side fixes | ported separately |

Until #867 + #2477 are in, this branch needs a vkd3d-proton with the relaxed vtable checks, as `amd-dev` itself does.

## Running it

As `linux/stack` (its `linux/README.md`): `VKD3D_CONFIG=ignore_shared_fence` and the same `XCL_` switches. Since
`926433d8` the direct translator is `amd-dev`'s default for compute, so `XCL_DIRECT_DXIL=1` is not needed.
