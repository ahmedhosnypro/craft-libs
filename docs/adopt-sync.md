# Craft-libs adoption sync ledger

Tracks the `craft-*` git pins in the Craft Apps against this
repository's default branch (`main`). Each row records the pin each app
carries, the current target HEAD, whether the pin lags, and the merged
pull request that is evidence for the row.

Window: 7h look-back ending 2026-10-14 02:51 UTC (6-hour cadence with
overlap for idempotence and missed-slot healing). Swept via
`craft-helpers prs` across all 17 repos in the adoption scan list;
98 merged pull requests observed, 66 touching watch-paths.

Current target HEAD at scan time:
`e9546e191dd62f2bc1d61eca45dbdc2f1b3c1d8e` (2026-10-13 05:38 UTC).

## Pin matrix (observed on each repo's default branch)

| App | Crate | Pinned rev | Rev date | Lags HEAD by | Evidence merge |
|---|---|---|---|---|---|
| storytold/pdfcraft | craft-print | `d991bf6792f8ade99c51358bf3c1d902c8cd96b5` | 2026-10-12 | 7 commits / 1 merge (666c358) | [pdfcraft#1132](https://github.com/storytold/pdfcraft/pull/1132) |
| storytold/pdfcraft | craft-print | — (bumped to `f03ec27ab2c405969428512178f746183cd55ca0`) | 2026-10-12 | 4 commits / 1 merge (e9546e1) | [pdfcraft#1132](https://github.com/storytold/pdfcraft/pull/1132) |
| storytold/pdfcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits | [pdfcraft#1132](https://github.com/storytold/pdfcraft/pull/1132) |
| storytold/photocraft | craft-print | `d991bf6792f8ade99c51358bf3c1d902c8cd96b5` | 2026-10-12 | 7 commits / 1 merge (666c358) | pin sweep |
| storytold/photocraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits | pin sweep |
| storytold/vectorcraft | craft-print | `d991bf6792f8ade99c51358bf3c1d902c8cd96b5` | 2026-10-12 | 7 commits / 1 merge (666c358) | pin sweep |
| storytold/vectorcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits | pin sweep |
| storytold/designcraft | craft-print | `d991bf6792f8ade99c51358bf3c1d902c8cd96b5` | 2026-10-12 | 7 commits / 1 merge (666c358) | pin sweep |
| storytold/designcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits | pin sweep |
| storytold/filmcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits | pin sweep |
| storytold/effectcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits | pin sweep |
| storytold/lightcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits | pin sweep |
| storytold/cadcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits | pin sweep |
| storytold/soundcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits | pin sweep |

deckcraft, gridcraft, wordcraft, artcraft, artcraft-services,
craft-repo-status-app, craft-launcher and craft-fonts carry no
`craft-*` git pins in their root `Cargo.toml`.

## What moved since the pins

- `crates/print` gained 7 commits after `d991bf67…` ("Report Windows
  driver duplex capability to print callers"), including merge
  666c358 — Windows print progress listing (crates/print/src/progress.rs,
  +110 lines) and bounded PDF parsing fixes.
- `crates/ui` gained 3 commits after `5c874d51…` (shared printer
  settings, async job feedback, accessible print control labels).
- 76 commits total between the craft-ui pin and HEAD.

## Evidence merges this window

- [pdfcraft#1132](https://github.com/storytold/pdfcraft/pull/1132) —
  "Pick up craft-print's native Windows printing" (merged 2026-10-14
  02:51 UTC window): bumped `craft-print` from `d991bf67…` to
  `f03ec27…` in `Cargo.toml` (1 file, +1/-1). Locked at `f03ec27…`,
  which is 4 commits behind current HEAD `e9546e1…` (craft-print
  progressed twice more after that bump).
- The pin sweep above (read-only `contents/Cargo.toml` API reads of
  each repo's default branch) is the cross-check the scan mission
  requires for rev-sync rows; the pdfcraft row is anchored by a merge
  in the window itself.

## Outstanding sync proposals

1. **pdfcraft → `e9546e191dd62f2bc1d61eca45dbdc2f1b3c1d8e`** — its
   `craft-print` pin (`f03ec27…`) predates two further craft-print
   commits (ebc4d8c, 4af92ad: caller-thread progress for native sheet
   processing). Smallest, most mechanical follow-up to #1132.
2. **photocraft, vectorcraft, designcraft → `e9546e1…` craft-print**
   (currently `d991bf67…`, missing the Windows progress-listing
   surface from merge 666c358).
3. **All eight pinned apps → `e9546e1…` craft-ui** (currently
   `5c874d51…`, missing shared printer settings + async job feedback
   from b700ae7).
