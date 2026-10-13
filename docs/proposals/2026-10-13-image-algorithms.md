# Proposal: `craft-image-algorithms` — shared FFT sharpening and depth-from-focus

- **Date:** 2026-10-13 (run window 2026-10-13T12:50:04Z – 19:50:04 UTC)
- **Status:** proposal only — evidence gathering; no code extracted yet
- **Target crate:** `craft-image-algorithms` in `crates/image-algorithms`, named per
  AGENTS.md. The repo-list calls the shared surface `algo`.
- **Evidence window:** 7-hour look-back on the 6-hour adoption cadence

## Summary

PhotoCraft merged two substantial, generic image-processing algorithms into its
app-local `crates/algo` in this window: FFT-based large-radius convolution
sharpening, and depth-from-focus map synthesis. Both are textbook, non-learned
image processing with no PhotoCraft domain model in the hot paths, and both
landed because no shared crate carries them.

## Evidence

- storytold/photocraft#2987 — "Speed up large-radius Unsharp Mask with guarded
  FFT convolution" (optimalbytee, merged 2026-10-13T17:23:57Z). 11 files,
  +1412/−18, including `crates/algo/src/sharpen.rs`, `sharpen/fft.rs`,
  `sharpen/fft_bench.rs`, `sharpen/reference.rs`, `sharpen/exact.rs`,
  `sharpen/tests.rs`; the FFT path (radius 64–1000, ≥65 536 px, `rustfft`)
  falls back to a streamed separable exact path preserving tap order.
- storytold/photocraft#2073 — "Auto-Blend Stack Images: a depth map by depth
  from focus, for Lens Blur" (ragtux, merged 2026-10-13T17:23:36Z). 3 files,
  +1101/−6, including `crates/algo/src/dff.rs` (focus-volume recipe: per-layer
  focus measure on luminance, depth assignment, map conditioning) feeding a
  Lens Blur depth map.

## Why this is generic, not app-domain

The algorithm bodies are pure image math operating on pixel planes: no
PhotoCraft layer model, no UI, no document semantics. The app-specific parts
(command wiring in `crates/engine`, dialog UX in `crates/ui-egui`) stay
app-side and are listed here only as the integration boundary. Any suite app
doing image filtering or focus stacking could reuse these verbatim.

## Proposed extraction (for human decision)

A `craft-image-algorithms` crate carrying, to start:

- guarded FFT convolution with exact-path fallback (from #2987, `rustfft`
  already a workspace-family dependency);
- depth-from-focus map synthesis over aligned planes (from #2073);
- reference and exact-path oracles plus property tests as shared test assets.

## File inventory (evidence merges)

| Merge | Files |
|---|---|
| photocraft#2987 | `crates/algo/src/lib.rs`, `crates/algo/src/sharpen.rs`, `crates/algo/src/sharpen/exact.rs`, `crates/algo/src/sharpen/fft.rs`, `crates/algo/src/sharpen/fft/tests.rs`, `crates/algo/src/sharpen/fft_bench.rs`, `crates/algo/src/sharpen/reference.rs`, `crates/algo/src/sharpen/tests.rs` (app-side wiring: `crates/ui-egui/examples/unsharp_preview_bench.rs`, `docs/*`) |
| photocraft#2073 | `crates/algo/src/dff.rs`, `crates/algo/src/lib.rs` (app-side wiring: `crates/engine/src/align_cmds.rs`) |

## NOTICE provenance draft

```markdown
The FFT sharpening and depth-from-focus algorithms in crates/image-algorithms
are adapted from:
- PhotoCraft, crates/algo/src/sharpen.rs and crates/algo/src/dff.rs,
  revision 50c4f72af71f2066045edf5045a4790071dc9abc.
  https://github.com/storytold/photocraft
  Copyright (c) 2026 ArtCraft Team and the PhotoCraft contributors.
```

(Revision from `commits?path=crates/algo` on storytold/photocraft, queried
2026-10-13.)

## Architecture compliance

- No universal renderer: pure pixel-plane math, no compositor, no scene graph.
- No i18n surface touched; no font or asset binaries.
- Never-crash lints inherited from the workspace per target AGENTS.md; the
  #2987 thresholds exist precisely to fall back instead of panicking on
  unsupported geometry.
