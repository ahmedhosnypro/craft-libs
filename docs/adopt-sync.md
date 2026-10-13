# Adopt-sync: app pins of craft-libs revisions

Tracking doc for the Craft Apps' git pins of shared craft-libs crates, and how far each pin
lags the shared library default branch. Apps adopt by pinned revision:

```toml
craft-<name> = { git = "https://github.com/storytold/craft-libs", rev = "<commit>" }
```

(see the target README, "Layout"). This page records the pins observed in each app's root
`Cargo.toml` `[workspace.dependencies]` and the drift against the library `main` branch, so the
next bump PR is a copy-paste job.

Verified 2026-10-13T01:33:17Z by the 2026-10-13 adoption scan. This refreshes the snapshot proposed in
PR #1 (`agent/adopt-2026-10-13-craft-libs-adoption`, verified 00:37 UTC): the pins are
unchanged, the drift grew, and no bump PR was merged in the 192-merge window
2026-10-12T18:14:56Z–2026-10-13T01:14:56Z scanned this run.

## Current drift

Library `main` at scan time: `666c358065261a2816b376fa3bb10538dbf53040`
(storytold/craft-libs, committed 2026-10-13T00:12:53Z). Pins point at the upstream
`storytold/craft-libs` repository, not this fork.

### craft-ui @ `5c874d51eeb3de70e735c8ce87a59a7dedd47675`

Pin merge: "UI.8: shape dock action labels with logical accessibility", 2026-10-10T20:08:31Z.
Upstream `main` is **65 commits ahead** of this pin (was 65 at 00:37 UTC; no bump merged since).

| App | Pin evidence (root `Cargo.toml`, read via contents API this run) |
|---|---|
| storytold/photocraft | `craft-ui = { git = "https://github.com/storytold/craft-libs", rev = "5c874d51eeb3de70e735c8ce87a59a7dedd47675", features = ["serde"] }` |
| storytold/vectorcraft | same rev, `features = ["serde"]` |
| storytold/designcraft | same rev, `version = "0.1.0"`, `features = ["serde"]` |
| storytold/pdfcraft | same rev, `version = "0.1.0"`, `features = ["serde"]` |
| storytold/filmcraft | same rev, `version = "0.1.0"`, `features = ["serde"]` |
| storytold/lightcraft | same rev |
| storytold/cadcraft | same rev, `version = "0.1.0"`, `features = ["serde"]` |
| storytold/soundcraft | same rev, `version = "0.1.0"`, `features = ["serde"]` |

### craft-print @ `d991bf6792f8ade99c51358bf3c1d902c8cd96b5`

Pin merge: "Report Windows driver duplex capability to print callers", 2026-10-11T23:04:26Z.
Upstream `main` is **47 commits ahead** of this pin. Between the pin and `main`: the in-process
Windows print backend ("Print on Windows in-process, without PowerShell"), shared printer
settings + asynchronous job feedback, accessible print-control labels, and the craft-quit
addition.

| App | Pin evidence |
|---|---|
| storytold/photocraft | root `Cargo.toml`: `craft-print = { git = "https://github.com/storytold/craft-libs", rev = "d991bf6792f8ade99c51358bf3c1d902c8cd96b5" }` |
| storytold/vectorcraft | same rev |
| storytold/designcraft | same rev, `version = "0.1.0"` |
| storytold/pdfcraft | same rev, `version = "0.1.0"` |
| storytold/deckcraft | no `craft-print` pin in root `Cargo.toml` — not yet adopting |

## Window evidence (2026-10-12T18:14:56Z–2026-10-13T01:14:56Z)

- 192 merges scanned across the suite (photocraft 101, vectorcraft 53, deckcraft 30,
  designcraft 6, pdfcraft 2; other 12 repos: 0). Title scan found **no bump / rev / pin
  pull requests**: the pins above were not advanced by any merge in the window.
- New adoption-gap signal: storytold/deckcraft#317 "Pictures: WMF and EMF draw (metafile
  reader ported from WordCraft)" (echelon, merged 2026-10-12T22:27:19Z) vendored a
  wordcraft crate into the app instead of adopting a shared one — 8 of 9 files in
  deckcraft `crates/metafile/src` are blob-SHA-identical to wordcraft's. Landing the
  craft-metafile proposal (PR #1) would make the next adoption a one-line `rev` pin
  instead of another verbatim fork.

## How to act on this page

Each row is a candidate bump PR in the *app* repo (out of scope here — this repo only
tracks). Bumping craft-print picks up the in-process Windows backend and async job
feedback; bumping craft-ui picks up the last 65 commits of shared UI work.

## Method

Window scan over the suite's merged pull requests via the REST pulls list
(`sort=updated`, GET params only); pins read from each app's root `Cargo.toml` via the
GitHub contents API; drift measured with
`repos/storytold/craft-libs/compare/<pin>...main`.
