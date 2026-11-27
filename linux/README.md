# XCL on Linux: the stack on `amd-dev`

`amd-dev` plus the open pull requests Linux needs, as `linux/stack` carries them on `main`. **AMD is not tested here:**
it is built and tested on the CPU and on NVIDIA cards only; nobody has run it on an AMD card or on RADV.

Updated 2026-11-27. Not run on the title yet. It builds, and its lib suite was run under Wine on the CPU (lavapipe, one
test at a time, 1 run each) on vkd3d-proton `master` `71cae091`, unpatched: 1,713 passed, 22 failed, 28 ignored. 21 of
the 22 fail on `amd-dev` `7c795f5e` + #684 + #794 too (on a vkd3d-proton whose vtable checks are relaxed, which it needs); the
other one, `depth_uav`'s `gpu_d24s8_and_d16s8_enhanced_barrier_twin_sync_passes_equal_depth_test`, calls the runtime's
`Reset` on an adopted list itself. That `amd-dev` tree on the unpatched vkd3d-proton failed 51 tests and then stopped in a
texture-upload test. The card tests of the ported pull requests ran on native Windows (RTX 2070 SUPER 8 GB, driver
621.01) at `9c2edf76`: 33 passed and 2 failed, the two `amd-dev` fails by itself there
(`offline_compute_pipeline_is_reused_by_a_second_process` and its split twin).

## What it is

`amd-dev` `7c795f5e` with these on top. `amd-dev` does not hold `main` (36 commits behind today), so a pull request made
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
| #867 | `021b2c33` | cherry-picked; `amd-dev`'s class tables are RwLocks, its own code stays around each guarded PC call, and the fences it added since (sparse's `rb_done_fence` and per-queue fences, vram's retire queue, `console_heap`'s queue fences) are raw ones too |
| #2477 | `5225ddcb` | cherry-picked on #867; `make_class` keeps `amd-dev`'s `find_matching_class`, and a class patched in place has no copied snapshot |
| #2707 | `1a3b6981` | in part: of its six commits, `78799549` is in `amd-dev` already and `f796b9e6` is there as `5568d655` (`backing_allocator`); `068671d5` (the reset guard) and `dff6edc7` are cherry-picked, clean; `59cc732a` and `1a3b6981` are left out: `amd-dev` holds a released allocator's busy backings its own way (`pending_releases`), and the last builds on the other's orphan list |
| #1975 | `72960386` | cherry-picked from its port `port/1975-amd-dev` `62379faf` |
| #2070 | `d6bd4170` | cherry-picked from its port `port/2070-amd-dev` `a88477cb`: `WORKER_STACK` is 16 MB there, `amd-dev`'s own worker stack |
| #2229 | `98a629a3` | cherry-picked from its port `port/2229-amd-dev` `0331600f`: it replaces `amd-dev`'s earlier copy of this pull request (`6e83f828`) with the head, `kernel.rs` kept to `amd-dev`'s API |
| #2872 | `2dad32e6` | cherry-picked, clean |
| #688 | `760f742b` | cherry-picked; `amd-dev`'s imports kept beside the new ones; its library link step still makes its linker and utils on the shared heap, only the compilers get a private one |

One commit of ours on top of the #867 port, `de0dcbce`: `amd-dev`'s `ExecuteCommandLists` hook sends XCL's own submits (sparse
uploads and readbacks, texture uploads) to the PC before the place where #867 restores the lists' native vtables, so it
restores them there too. Without it the suite above ends in a page fault inside vkd3d-proton's `ExecuteCommandLists`.

Already in `amd-dev`: #1803 (its head `cb007514` is an ancestor).

Left out, because `amd-dev` has the fix in a form of its own:

| Pull request | What `amd-dev` has |
|---|---|
| #751 | `6d7dcbb9` "read non-UMA title_bytes safely via read_committed": the same condition (borrow inside the coherent window, copy elsewhere) |
| #1599 | `40f499a7` defers the half-full auto-submit for heap page-array batches, and `a0442743` reads a physical-page list again when its request runs |
| #2762 | `c093211e` "a placed view's padding does not make a second texture" |
| #2404 | its own chained ExecuteIndirect: `77453544`, `fb81714d`, `6a0bb5b4` and later (`real/dispatch_chain.rs`) |
| #2936 (`cae11a79`) | `c5951d21` "detach retired segments and reclaim idle aside allocators": its `alloc_pool::retire_recording` has the same body as #2936's `park_list` and is called at #2936's two places in `title_list.rs`; the third, `kickoff_x`'s reclaimed segments, has that call here through #2707's `dff6edc7`. #2936 itself conflicts at all three places |

Not ported: #2633 (split programs' pieces read their slab from a root CBV, less GPU time for each chained
ExecuteIndirect). It is written on #2404's chain code, which `amd-dev` replaces with its own (33 conflicts), so it would
be a rewrite there and needs a card to check.

With #867 + #2477 in, it is meant for stock vkd3d-proton as `linux/stack` is (`master` `71cae091`); not run on it yet.
What was checked on the CPU, at `9b670905`: every compute program the launch pass translates goes through that
vkd3d-proton's dxil-spirv (`b540c8a2`) and its output passes `spirv-val`: 1,613 of 1,613 as a run gets them (the 1,073
ray-tracing programs `XCL_RT_GATE` holds back left out), 2,682 of 2,682 with `XCL_RT_GATE=0`, and 1,613 of 1,613 unsplit;
the same through `788f3e22` (vkd3d-proton `256f7510`). 1 pass each.

The launch pass under Wine, on the CPU (8 DXC threads, an empty cache each pass, 3 passes each in turn, beside other
jobs on the machine): the first 182 pixel shaders of the pack took 110, 116 and 127 s on `amd-dev` `7c795f5e` + #684 + #794,
and 22, 26 and 27 s on this branch, which has #688 and #2872.

## Running it

As `linux/stack` (its `linux/README.md`): `VKD3D_CONFIG=ignore_shared_fence` and the same `XCL_` switches. Since
`926433d8` the direct translator is `amd-dev`'s default for compute, so `XCL_DIRECT_DXIL=1` is not needed.
