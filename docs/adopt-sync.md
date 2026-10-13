# Adopt-sync — pin freshness across the Craft suite

Tracks how far each app's pinned `craft-libs` revisions lag the shared
libraries' default branch. Apps depend on crates by git rev (see the
[README](../README.md#layout)); a stale pin means an app ships without fixes
that already landed in the shared crates. Created by the lib-adoption scan on
2026-10-13 (window 15:53–22:53 UTC) and refreshed when a scan finds drift or a
bump merge.

## Suite-wide pin state (checked 2026-10-13 ~23:00 UTC)

Compared against `main` at `e9546e191dd62f2bc1d61eca45dbdc2f1b3c1d8e`.
Every repository that pins a craft-libs crate is behind `main`:

| App | Crate | Pinned rev | Commits behind | Newer work missed |
| --- | --- | --- | --- | --- |
| photocraft | craft-print | `d991bf6792f8ade99c51358bf3c1d902c8cd96b5` | 58 | native Windows in-process printing, printer settings, craft-atomic, craft-quit, craft-cms, craft-settings |
| photocraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 76 | everything above plus the print progress work |
| vectorcraft | craft-print | `d991bf67…` | 58 | same as photocraft/print |
| vectorcraft | craft-ui | `5c874d51…` | 76 | same as photocraft/ui |
| designcraft | craft-print | `d991bf67…` | 58 | same as photocraft/print |
| designcraft | craft-ui | `5c874d51…` | 76 | same as photocraft/ui |
| pdfcraft | craft-print | `f03ec27ab2c405969428512178f746183cd55ca0` | 14 | caller-thread progress for native sheet processing, craft-cms, craft-atomic permission policy |
| pdfcraft | craft-ui | `5c874d51…` | 76 | same as photocraft/ui |
| wordcraft | craft-ui | `d27646a5f7c8b4fc85d9dd3213b8c88b58e0088d` | 55 | native Windows printing, craft-atomic, craft-quit, craft-cms, craft-settings |
| gridcraft | craft-ui | `d27646a5…` | 55 | same as wordcraft |
| cadcraft | craft-ui | `5c874d51…` | 76 | same as photocraft/ui |
| filmcraft | craft-ui | `5c874d51…` | 76 | same as photocraft/ui |
| effectcraft | craft-ui | `5c874d51…` | 76 | same as photocraft/ui |
| lightcraft | craft-ui | `5c874d51…` | 76 | same as photocraft/ui |
| soundcraft | craft-ui | `5c874d51…` | 76 | same as photocraft/ui |

`deckcraft`, `craft-launcher` were checked and pin no craft-libs crates;
`craft-fonts` is a source repository, not a consumer. Pins were read from each
app's root `Cargo.toml` (wordcraft/gridcraft: `crates/ui-egui/Cargo.toml`) via
the GitHub contents API. "Behind" counts are `…rev...main` ahead-by values from
the GitHub compare API. `craft-cms` and `craft-quit` have no pinners at all yet:
they exist only at revisions no app depends on.

## In-window evidence: the suite does bump, on demand

[storytold/pdfcraft#1132](https://github.com/storytold/pdfcraft/pull/1132)
("Pick up craft-print's native Windows printing", merged 2026-10-13T20:46:24Z,
author jcl-bit) moved pdfcraft's `craft-print` pin in the root `Cargo.toml`:

- `d991bf6792f8ade99c51358bf3c1d902c8cd96b5` →
  `f03ec27ab2c405969428512178f746183cd55ca0`

with `Cargo.lock` regenerated (+1073/−77), `crates/print/README.md` updated and
`crates/ui-egui/src/open_worker_tests.rs` (+60) exercising the new behaviour.
This is the mechanical bump pattern every stale row above needs.

Two in-window merges did **not** bump pins, and are the counter-evidence this
doc tracks: vectorcraft's identically titled
[#1132](https://github.com/storytold/vectorcraft/pull/1132) ("Pick up
craft-print's native Windows printing") touched `Cargo.lock`, `Cargo.toml`,
ROADMAP and `crates/print/README.md` but no `rev =` line — vectorcraft's print
pin stayed at `d991bf67…` (58 behind), so the pickup is incomplete there — and
[storytold/craft-launcher#313](https://github.com/storytold/craft-launcher/pull/313)
touched only app-level manifests. No bump PR by the apps for `craft-ui` was
seen in the window: its pin has now drifted 76 commits.

## Recommendation

Per-app bump PRs, smallest first: one manifest line per crate (the pdfcraft
pattern above) — three `craft-print` bumps at `d991bf67…` (photocraft,
vectorcraft, designcraft), one `craft-print` refresh at `f03ec27…` (pdfcraft),
and nine `craft-ui` bumps at `5c874d51…`/`d27646a5…` — then, as consumers are
ready, first adoptions of `craft-atomic`, `craft-quit`, `craft-cms` and
`craft-settings`, which today have zero pins. This scan cannot open pull
requests in the app repositories; it records the drift here and names the
evidence.
