# Proposal: `craft-gradient` — shared gradient stop/ramp math for the Craft Apps

**Status:** proposal (evidence only; no code extraction in this phase — humans decide)
**Target crate name:** `craft-gradient` (`crates/gradient`, named per AGENTS.md "New crates")
**Date:** 2026-10-13
**Window:** merges 2026-10-12T18:14:56Z–2026-10-13T01:14:56Z (7h look-back on the 6h cadence)

## Why

The same gradient stop/ramp semantics were re-derived in three apps inside one window.
The diffs are generic ramp math — no app-domain model appears in any of them — yet each
app carries its own copy on its own drift schedule.

## Evidence (merged pull requests in the window)

| Merge | Owner | What it changed | Files |
|---|---|---|---|
| [storytold/designcraft#638](https://github.com/storytold/designcraft/pull/638) (webdevtodayjason, merged 2026-10-12T23:20:54Z, merge commit `cc5b4a1f0f9dac7fc509191746ec048d6c8ffb1a`) | designcraft | `Gradient::reverse()` now moves each midpoint with its segment and mirrors it (`1.0 - midpoint`); previously it reversed offsets only | `crates/color/src/gradient.rs` (+20/−1) |
| [storytold/vectorcraft#1530](https://github.com/storytold/vectorcraft/pull/1530) (EricKuck, merged 2026-10-13T00:13:29Z, merge commit `40b02f479b789e9c02bb9a5248dc4bac2ee8dea8`) | vectorcraft | AI import: stops listed downwards are conditionally reversed before sorting, so stops sharing a ramp point keep their order "and each its midpoint, to the next stop" | `crates/eps/src/import/ai/paint.rs` (+5/−2), `crates/eps/src/import/ai/tests.rs` (+13/−0) |
| [storytold/deckcraft#273](https://github.com/storytold/deckcraft/pull/273) (echelon, merged 2026-10-12T21:36:00Z, merge commit `ceb6e3b990f9fe73be3e932754d633fdf73bf96c`) | deckcraft | Two-colour gradients ramp per PowerPoint instead of muddy linear sRGB; new ramp module | `crates/render/src/ramp.rs` (+128/−0), `crates/render/src/paint.rs`, `lib.rs`, `tests.rs` |
| [storytold/deckcraft#281](https://github.com/storytold/deckcraft/pull/281) (echelon, merged 2026-10-12T21:51:20Z) | deckcraft | Text gradient fills restart over every line, PowerPoint-style | deckcraft `crates/render` (policy surface) |
| [storytold/deckcraft#288](https://github.com/storytold/deckcraft/pull/288) (echelon, merged 2026-10-12T21:59:33Z) | deckcraft | Rectangle and outline path gradients | deckcraft `crates/render` |
| [storytold/photocraft#4337](https://github.com/storytold/photocraft/pull/4337) (xtellarvoid, merged 2026-10-13T01:14:17Z) | photocraft | GRD (Photoshop gradient preset) import into the gradient-map preset picker | app-side `crates/ui-egui/adjust_dialog.rs`, `adjust_editors.rs` + per-app i18n catalogs |

Same-window drift, concretely:

- designcraft#638 fixes midpoint handling in `reverse()`; vectorcraft#1530's patch comment
  encodes the same midpoint-to-next-stop invariant from the import side. Two apps, one
  invariant, one window, no shared code.
- deckcraft#273 adds a second ramp implementation (`ramp.rs`) alongside designcraft's
  `crates/color/src/gradient.rs` sampler.
- photocraft#4337 reads Photoshop `.grd` presets while deckcraft#281/#288 target
  PowerPoint's gradient model: three apps, three gradient dialects, zero shared code.

## What `craft-gradient` would own (boundary, per AGENTS.md)

Stop sort/normalize/reverse semantics, ramp interpolation (including the PowerPoint-style
two-colour ramp from deckcraft#273), preset interchange readers (Office gradient model,
Photoshop GRD per photocraft#4337).

NOT owned (app code stays in the apps): document model, render layering, UI editors, and
per-app localization. No universal renderer and no i18n extraction — the scope policy in
AGENTS.md wins; each app keeps its own draw path and calls the crate for math.

## Provenance (NOTICE "Source provenance" draft)

The gradient stop/ramp math in crates/gradient is adapted from:

- DesignCraft, `crates/color/src/gradient.rs`, revision
  `cc5b4a1f0f9dac7fc509191746ec048d6c8ffb1a` (storytold/designcraft#638).
  https://github.com/storytold/designcraft
  Copyright (c) 2026 ArtCraft Team and the DesignCraft contributors.
- VectorCraft, `crates/eps/src/import/ai/paint.rs`, revision
  `40b02f479b789e9c02bb9a5248dc4bac2ee8dea8` (storytold/vectorcraft#1530).
  https://github.com/storytold/vectorcraft
  Copyright (c) 2026 ArtCraft Team and the VectorCraft contributors.
- DeckCraft, `crates/render/src/ramp.rs`, revision
  `ceb6e3b990f9fe73be3e932754d633fdf73bf96c` (storytold/deckcraft#273).
  https://github.com/storytold/deckcraft
  Copyright (c) 2026 ArtCraft Team and the DeckCraft contributors.

## Integration

- Register in `[workspace.dependencies]` (AGENTS.md; README "Layout"); apps adopt by pinned
  `rev`, as they already do for `craft-ui` and `craft-print`.
- Gate before merging: `cargo fmt --check`, `cargo clippy --workspace --all-targets -- -D warnings`,
  `cargo test --workspace`; `lib.rs` carries the never-crash deny list.
