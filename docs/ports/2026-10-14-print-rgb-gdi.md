# Port note: craft-print RGB/GDI surface (source 2026-10-14)

**Status:** port note (evidence only; no code moves in this phase — humans decide)
**Source change:** `craft-print` `crates/print`, unmerged branch tip `3d37da389a3618ab9b76109da11416e9ca4ea719` (fork point `666c358`, 7 commits)
**Date:** 2026-10-14 (window 2026-10-13T22:01:45Z → 2026-10-14T05:38:31Z UTC, 7h look-back)

## What changed in the source

photocraft merged the shared-surface change upstream
([photocraft#4599](https://github.com/storytold/photocraft/pull/4599),
"Share native Windows driver settings and RGB printing through
craft-print", merge `369704ac`) and pinned the resulting rev. The rev
is **not on this repository's `main`** — see
`docs/adopt-sync.md` (this run's ledger refresh): `main..3d37da38` is
7 commits, all `crates/print`:

- `ccb1eae` — refuse fitting that would discard annotations
  (`sheets.rs` +9, README, 28 test lines).
- `8fb8956` — **`craft_print::rgb`**: "safe driver settings, printable
  geometry, RGB pages and caller-composed vector strokes" via the new
  git dependency `craft-winprint-gdi`
  (`storytold/winprint.rs` @ `c32b8e71…`); the README's color policy is
  explicit (app-managed/unmanaged RGB → `ICM_DONE_OUTSIDEDC` around
  `StartPage`; driver-managed RGB → `ICM_OFF`; vendor auto-correction
  is a driver setting and requires qualification; other platforms
  return `rgb::PrintError::Unsupported`).
- `6ba4d0b` — refuse annotation visibility the native renderer cannot
  print correctly; `3929c3d` — refuse explicit print options dropped by
  Windows drivers (`ticket.rs` +66); `d873514` — README documentation
  of the refusals; `3d37da38` — merge.

This repository's `main` holds the **old** copy-paste recipe: PDF-only
Windows spooling (in-box `Windows.Data.Pdf` → XPS PrintTicket), CUPS on
macOS/Linux, no `rgb` module (verified at `e9546e1`).

## Which apps need the port, and their current recipe state

| App | Current recipe state (read this window) | Port needed |
|---|---|---|
| **photocraft** | Done — pinned `3d37da38` (photocraft#4599); uses the `rgb` surface via `crates/ui-egui` (`craft-print = { workspace = true }`) | none |
| **vectorcraft** | `craft-print = { …, rev = "d991bf67…" }` — pre-`rgb`, 7 print commits behind the fork point too | bump to the merged rev after the branch lands on `main`; no app-side API migration observed in the window |
| **designcraft** | `craft-print = { version = "0.1.0", …, rev = "d991bf67…" }` | same as vectorcraft |
| **pdfcraft** | `craft-print = { version = "0.1.0", …, rev = "f03ec27…" }` (bumped by [pdfcraft#1132](https://github.com/storytold/pdfcraft/pull/1132) in the prior window) | two-print-commit bump after the branch lands; the `rgb` surface is opt-in, so existing PDF-spool calls keep compiling |

The `rgb` surface is additive: `pub mod rgb { pub use craft_winprint_gdi::*; }`
inside `crates/print/src/lib.rs`, with a separate dependency
(`craft-winprint-gdi`, MIT OR Apache-2.0, `unsafe` isolated upstream).
Apps that do not opt in keep the existing PDF/CUPS transport, so the
port is a pin bump plus (for apps that want native driver settings)
adoption of the `rgb` API in their print command layer.

## Order of operations (for humans)

1. Merge the diverged craft-print branch (`3d37da38`) into `main` —
   until then no rev carries both the progress-listing work (`666c358`,
   already on `main`) and the shared driver-settings/RGB surface.
2. Bump the three lagging pins (vectorcraft, designcraft, pdfcraft) per
   `docs/adopt-sync.md`.
3. Apps wanting native Windows driver settings adopt `craft_print::rgb`
   in their own print-command layer; ICC conversion, document layout,
   labels and print-mark composition stay in each app, exactly as the
   surface's README requires.

## Architecture compliance

This note is documentation only. It respects the target's AGENTS.md
scope policy: craft-print keeps PDF generation app-side ("PDF
generation stays in each app", README), and the note proposes no
universal renderer, no i18n extraction, no font binaries. storytold/*
repositories were read via the GitHub API only; all writes in this
pull request are on this fork.
