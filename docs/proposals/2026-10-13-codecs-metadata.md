# Proposal: `craft-codecs-metadata` — shared container metadata fidelity

**Window:** 2026-10-13, 7-hour look-back (2026-10-13T13:33:58Z – 20:33:58Z).
**Status:** evidence-only proposal. No code is extracted in this phase; humans decide.
**Target crate:** `craft-codecs-metadata` (new crate under `crates/`, named
per the `craft-<name>` convention), carrying the metadata layer of the image
container codecs — EXIF directories, ICC profile handling and physical
resolution — independent of any app's document model.

## Why a shared crate

Four merges in this window by four different authors each fixed a place where
`storytold/photocraft`'s `crates/codecs` lost or mangled container metadata:
EXIF directories in TIFF, colour tables in indexed PNG, whole-ppi resolution
in PNG, and ICC profiles on CMYK composites. Every Craft app opens the same
containers, and every one will meet the same lossy-metadata bugs; three of
the four fixes carry no app vocabulary at all.

This promotes the previous run's deferred weak-hoist item (TIFF EXIF,
photocraft#4876) with the corroboration it asked for, and it sits next to the
already-open `craft-fontdb`/`craft-text-shaping` work (PR #4) in covering the
suite's `fonts`/`io`/`codecs` shared-surface candidates.

## Evidence (merged pull requests)

All four merged inside this run's window (2026-10-13, 16:07–17:24 UTC).

### storytold/photocraft#4876 — TIFF: write and read EXIF as the file's own Exif and GPS directories
- **Author:** switch87, merged 2026-10-13T17:24:28Z, merge commit `5064d80b1e2d`.
- **Files:** `crates/codecs/src/codecs/tiff.rs`, new
  `crates/codecs/src/codecs/tiff_exif.rs`, `crates/codecs/src/format.rs`,
  `crates/io/src/tiff_layers.rs`, tests under `crates/codecs/tests/`.
- **What it does:** TIFF is EXIF's own container: IFD0 points to the Exif
  directory (34665) and GPS directory (34853) and carries the descriptive
  tags itself. The TIFF codec now writes and reads EXIF that way, via a new
  reusable reader/builder (`tiff_exif.rs`, ~195 + ~168 test lines) that also
  handles a bare EXIF block (which is itself a TIFF stream).
- **Why generic:** EXIF-in-TIFF directory mechanics are container
  specifications, not app features. Any suite app writing or reading TIFF
  wants identical behaviour.

### storytold/photocraft#4817 — Palette PNGs open in Indexed Color mode with their colour table
- **Author:** marcoarnulfo, merged 2026-10-13T16:33:40Z, merge commit `e4f005d3dcad`.
- **Files:** `crates/codecs/src/codecs/png.rs`, `crates/codecs/src/lib.rs`,
  `crates/io/src/flat.rs`, tests under `crates/codecs/tests/`,
  `crates/io/tests/flat.rs`, `crates/codecs/README.md`.
- **What it does:** a palette PNG now opens in indexed-colour mode carrying
  its PLTE chunk's colour table instead of being flattened.
- **Why generic:** PLTE handling is a PNG container rule. The `io/flat.rs`
  touch is the thin app-side glue that stays behind in the extraction.

### storytold/photocraft#4857 — PNG: a whole-ppi pHYs reads back whole (300 ppi stays 300)
- **Author:** switch87, merged 2026-10-13T16:07:09Z, merge commit `7b0dbf182bfc`.
- **Files:** `crates/codecs/src/codecs/png.rs`,
  `crates/codecs/tests/resolution.rs`.
- **What it does:** PNG pHYs resolution is stored in pixels per metre, so a
  whole ppi value did not read back as the same whole number. The codec now
  stores whole ppi in the decoded resolution instead of an unrounded
  dpm conversion.
- **Why generic:** a metadata round-trip fix in the container codec itself;
  zero app vocabulary.

### storytold/photocraft#4948 — io: a tagged CMYK file's merged image reads through its own profile
- **Author:** timothydaly1, merged 2026-10-13T16:07:15Z, merge commit `f1233bb43a69`.
- **Files:** `crates/io/src/lib.rs`, `crates/io/tests/cmyk_composite_export.rs`.
- **What it does:** the merged image of a tagged CMYK file now reads through
  the file's own ICC profile rather than assuming the working profile.
- **Why generic:** honouring embedded ICC profiles on decode is shared-codec
  contract, not app behaviour.

## Scope of the proposed crate

Extract the metadata layer from `photocraft`'s `crates/codecs`: the TIFF EXIF
directory reader/builder (`codecs/tiff_exif.rs`), PNG chunk-level metadata
(PLTE, pHYs), and ICC-profile-aware decode entry points. Pixel codecs stay
where they are; the app-side `crates/io` glue remains in the app.

- `crates/codecs-metadata` → `craft-codecs-metadata`, never-crash lints per
  AGENTS.md, no `unsafe`, no app dependencies.
- Listed in `[workspace.dependencies]`; consumed by apps as a pinned `rev`.
- Gates: `cargo fmt --check`, `cargo clippy --workspace --all-targets -- -D
  warnings`, `cargo test --workspace`.

## NOTICE provenance draft

> The TIFF EXIF directory reader and builder in `crates/codecs-metadata` are
> extracted from:
> - PhotoCraft, `crates/codecs/src/codecs/tiff_exif.rs` and
>   `crates/codecs/src/codecs/tiff.rs`, revision `5064d80b1e2d` (PR #4876),
>   merged 2026-10-13.
>   https://github.com/storytold/photocraft
>   Copyright (c) 2026 ArtCraft Team and the PhotoCraft contributors.
> The PNG colour-table and physical-resolution metadata handling in
> `crates/codecs-metadata` are extracted from:
> - PhotoCraft, `crates/codecs/src/codecs/png.rs`, revisions `e4f005d3dcad`
>   (PR #4817) and `7b0dbf182bfc` (PR #4857), merged 2026-10-13.
>   https://github.com/storytold/photocraft
>   Copyright (c) 2026 ArtCraft Team and the PhotoCraft contributors.
> The ICC-profile-aware decode contract in `crates/codecs-metadata` draws on
> PhotoCraft, `crates/io/src/lib.rs`, revision `f1233bb43a69` (PR #4948),
> merged 2026-10-13, under the same licenses above. Pixel codec and
> application code are not copied.

## Relationship to earlier proposals

- Previous run's rollup (2026-10-13, 19:50 slot) deferred #4876 as a weak
  hoist pending a second same-theme merge; this window supplies three more
  (#4817, #4857, #4948), so the item is promoted here.
- Overlaps none of the open proposals #1–#6.
