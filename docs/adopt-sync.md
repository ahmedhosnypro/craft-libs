# Adopt-sync: app pins of craft-libs revisions

Tracking doc for the Craft Apps' git pins of shared craft-libs crates, and how far each pin
lags the shared library default branch. Apps adopt by pinned revision:

```toml
craft-<name> = { git = "https://github.com/storytold/craft-libs", rev = "<commit>" }
```

(see the target README, "Layout"). This page records the pins observed in each app's root
`Cargo.toml` `[workspace.dependencies]` and the drift against the library `main` branch, so the
next bump PR is a copy-paste job. Verified 2026-10-13 00:37 UTC.

## Current drift

Target default branch HEAD at scan time: `666c358065261a2816b376fa3bb10538dbf53040`
(storytold/craft-libs `main`, committed 2026-10-13T00:12:53Z).

### craft-print @ `d991bf6792f8ade99c51358bf3c1d902c8cd96b5`

Pin merge: "Report Windows driver duplex capability to print callers", 2026-10-11T23:04:26Z.
Upstream `main` is **47 commits ahead**; 11 of them touch `crates/print`, including
"Print on Windows in-process, without PowerShell" (`754f4d70e06a`), the shared printer
settings + asynchronous job feedback block (`b700ae753f77`, `d27646a5f7c8`, `4cae45dbe8ef`)
and the bounded-PDF-parsing hardening (`85ef8b92ac3e`, `c22935270b7d`, `697b473c0b8b`,
`02d08fe9cc0f`).

| App | Pin evidence |
|---|---|
| storytold/photocraft | PR storytold/photocraft#3872 "Use craft-print for system printing" (merged 2026-10-12T21:00:10Z, merge commit `b3222ee8e7e6bcb4799c37f904c595f20f9baf5e`) adds the pin above in root `Cargo.toml` and wires `crates/engine/src/print_cmds.rs` (+69/−71) |
| storytold/vectorcraft | root `Cargo.toml` `[workspace.dependencies]`: `craft-print = { git = "https://github.com/storytold/craft-libs", rev = "d991bf6792f8ade99c51358bf3c1d902c8cd96b5" }` |
| storytold/designcraft | root `Cargo.toml`: same rev, `version = "0.1.0"` |
| storytold/pdfcraft | root `Cargo.toml`: same rev, `version = "0.1.0"`, comment "Shared system spooler; document composition remains in pdfcraft-print" |
| storytold/deckcraft | root `Cargo.toml` lists no `craft-print` git pin in the window's snapshot — not yet adopting |

### craft-ui @ `5c874d51eeb3de70e735c8ce87a59a7dedd47675`

Pin merge: "UI.8: shape dock action labels with logical accessibility", 2026-10-10T20:08:31Z.
Upstream `main` is **65 commits ahead** of this pin.

| App | Pin evidence |
|---|---|
| storytold/photocraft | root `Cargo.toml`: `craft-ui = { git = …, rev = "5c874d51…", features = ["serde"] }` |
| storytold/vectorcraft | root `Cargo.toml`: same rev + `features = ["serde"]` |
| storytold/designcraft | root `Cargo.toml`: same rev + `features = ["serde"]`, comment "keep this dependency above the engine boundary" |
| storytold/pdfcraft | root `Cargo.toml`: same rev + `features = ["serde"]`, comment "Shared frontend components; engines remain independent of egui" |

## How to act on this page

Each row is a candidate bump PR in the *app* repo (out of scope here — this repo only tracks).
The value of syncing: the in-process Windows print backend, printer settings and async job
feedback, and the craft-atomic/craft-quit additions all land between the pins and `main`.

## Method

Window scan 2026-10-12T18:37:23Z–2026-10-13T00:37:23Z over the suite's merged pull requests;
pins read from each app's root `Cargo.toml` via the GitHub contents API; drift measured with
`repos/storytold/craft-libs/compare/<pin>...main`. See `rollups/2026-10-13-00.md` for the full
window record.
