# Proposal: `craft-text-shaping` — shared script-aware font fallback and text classification

**Status:** proposal (evidence only; no code extraction in this phase — humans decide)
**Target crate name:** `craft-text-shaping` (`crates/text-shaping`, named per AGENTS.md)
**Date:** 2026-10-13 (window 2026-10-13T07:21:55Z → 2026-10-13T11:32:17Z UTC, 7h look-back)

## Why

Inside one 7-hour window, four apps landed convergent, generic text/fallback logic in their
own crates. Each change answers a question every Craft App that renders text must answer:
"which family should I substitute when the document's family has no glyph for this script?"
None of the diffs reads as app-domain: no document model, no timeline/deck/DWG semantics —
script detection, family selection, and glyph policy.

This is distinct from the open `craft-fontdb` proposal (docs/proposals/2026-10-13-fontdb.md,
PR #1): fontdb owns *discovery* (OS folders, aliases, fontconfig, missing-font lists). This
proposal covers what happens *after* discovery — script classification, fallback family
choice, and bidi/paragraph classification — and it shifts that boundary: fontdb's "What
stays in each app" section explicitly leaves fallback chains app-side. Four apps now
maintain that layer four times.

## Evidence (merged PRs, this window)

| PR | Merge (UTC) | What landed | Files |
|---|---|---|---|
| [storytold/effectcraft#1015](https://github.com/storytold/effectcraft/pull/1015) | 09:28:28 | Chinese fallback families follow the locale's script | `crates/text/src/fonts.rs` (+52/−16) |
| [storytold/effectcraft#1126](https://github.com/storytold/effectcraft/pull/1126) | 09:28:27 | Fallback tries a script's usual families before faces loaded earlier | `crates/text/src/fonts.rs` (+54/−25), `crates/text/tests/script_fallback.rs` (+36 new) |
| [storytold/soundcraft#417](https://github.com/storytold/soundcraft/pull/417) | 10:53:47 | Hangul fallback so Korean names render in the UI | `crates/ui-egui/src/fonts.rs` (+78/−11) |
| [storytold/cadcraft#861](https://github.com/storytold/cadcraft/pull/861) | 08:38:04 | CJK interface fonts in every language; big-font cells in SHX text | `crates/fonts/src/lib.rs` (+79/−9), `crates/fonts/src/ttf.rs` (+16/−6), `crates/ui-egui/src/theme.rs` (+66/−13) |
| [storytold/deckcraft#379](https://github.com/storytold/deckcraft/pull/379) | 10:48:44 | Bidi, tabs, underlines, caps, numbering, vertical, emoji — measured vs PowerPoint | `crates/text/src/bidi.rs` (+178/−1), `crates/text/src/lib.rs` (+627/−85), `crates/text/src/tests.rs` (+438/−3), `crates/fonts/src/fallback.rs` (+86/−16), `crates/fonts/src/fontdb.rs` (+105 new) |

All merges are in the master merge commit list; all file stats via
`gh api repos/storytold/<repo>/pulls/<n>/files`.

## What the shared crate would own

- Script classification (Hangul/Chinese/CJK/Latin/Arabic/…) from a codepoint run.
- Fallback family choice: per-script preferred families, then faces already loaded
  (effectcraft#1126's ordering rule), with per-app script→family tables staying app-side
  (each app's table is a policy choice).
- Bidi paragraph/run classification (deckcraft#379's `bidi.rs`).
- A `no_unresolved` diagnostic: any rendering path that gave up must return a diagnostic
  instead of a panic (per craftrules "never crash").

## What stays in each app

- Per-script family *tables* (soundcraft's Hangul pick vs effectcraft's Chinese pick are
  product choices; the shared crate takes them as config).
- App-specific rendering: cadcraft's SHX big-font cells, deckcraft's PowerPoint parity
  fixtures, soundcraft's UI theming integration.
- Discovery stays with the open craft-fontdb proposal; this crate consumes a discovered
  set, it does not enumerate folders.

## File inventory (evidence)

| Source | Files |
|---|---|
| storytold/effectcraft#1015 | `crates/text/src/fonts.rs` |
| storytold/effectcraft#1126 | `crates/text/src/fonts.rs`, `crates/text/tests/script_fallback.rs` |
| storytold/soundcraft#417 | `crates/ui-egui/src/fonts.rs` |
| storytold/cadcraft#861 | `crates/fonts/src/lib.rs`, `crates/fonts/src/ttf.rs`, `crates/ui-egui/src/theme.rs` |
| storytold/deckcraft#379 | `crates/text/src/bidi.rs`, `crates/text/src/lib.rs`, `crates/text/src/tests.rs`, `crates/fonts/src/fallback.rs`, `crates/fonts/src/fontdb.rs` |

## NOTICE provenance draft

```
The script classification, fallback family selection and bidi classification in
crates/text-shaping is adapted from:
- EffectCraft, crates/text/src/fonts.rs, revisions
  ec86cdb584d22c3944ccb3a9db3ab154794cde25 (PR #1015) and
  6be321153cc3448c123403cfb123ec0b84175717 (PR #1126).
  https://github.com/storytold/effectcraft
  Copyright (c) 2026 ArtCraft Team and the EffectCraft contributors.
- SoundCraft, crates/ui-egui/src/fonts.rs, revision
  55eff096ac02d7aab919fed54b48320236ad4754 (PR #417).
  https://github.com/storytold/soundcraft
  Copyright (c) 2026 ArtCraft Team and the SoundCraft contributors.
- CadCraft, crates/fonts/src/lib.rs and crates/fonts/src/ttf.rs, revision
  a10e43cb2ab55121aef3c5761ada4a74a6a6077c (PR #861).
  https://github.com/storytold/cadcraft
  Copyright (c) 2026 ArtCraft Team and the CadCraft contributors.
- DeckCraft, crates/text/src/bidi.rs, crates/fonts/src/fallback.rs and
  crates/text/src/lib.rs, revision
  525ec7b1a9769c0f73fac66ca564027c3f436229 (PR #379).
  https://github.com/storytold/deckcraft
  Copyright (c) 2026 ArtCraft Team and the DeckCraft contributors.
All sources listed above are licensed MIT OR Apache-2.0.
```

Pattern precedent: the NOTICE already records this style of per-revision attribution for
`crates/print` and `crates/ui` (see NOTICE "Source provenance").

## Architecture compliance

Per the target's AGENTS.md: no app code, no app dependency; the crate is useful to more
than one app (four demonstrated in one window); a new crate under `crates/<name>` named
`craft-<name>`, inheriting workspace fields and lints, registered in
`[workspace.dependencies]`. No universal renderer: the crate classifies and picks
families; each app keeps its own renderer, shaping engine and fonts. No i18n extraction:
locale catalogs (e.g. cadcraft's `crates/ui-egui/src/i18n/*.tsv`) stay in the apps; the
crate reads codepoints, not locales. Gate commands before merging: `cargo fmt --check`,
`cargo clippy --workspace --all-targets -- -D warnings`, `cargo test --workspace`.
