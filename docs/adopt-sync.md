# Craft-libs adoption sync ledger

Maintained by the six-hour adoption scans (`craft-libs-adoption`). Each
refresh rewrites this matrix from a read-only pin sweep of every suite
repository's default branch against this repository's `main`. One row
per app pin; the merged pull request that is evidence for the row is
cited.

Window: **7h look-back ending 2026-10-14 05:38 UTC** (6-hour cadence
with 1h overlap for idempotence and missed-slot healing), sweep
2026-10-13T22:01:45Z → 2026-10-14T05:38:31Z. 66 merged pull requests
collected via `craft-helpers prs` across all 17 scan repos; no repo hit
the per-repo cap, no truncation, no REST fallback needed.

This refresh supersedes the 2026-10-14 02:51 UTC ledger (PR #10): the
target HEAD is unchanged, but photocraft's `craft-print` row changed
this window (photocraft#4599) and the drift lines are re-measured.

Current target HEAD at scan time:
`e9546e191dd62f2bc1d61eca45dbdc2f1b3c1d8e` (2026-10-13 11:38 UTC).

## Pin matrix (observed this window on each repo's default branch)

| App | Crate | Pinned rev | Rev date | Status vs `main` | Evidence |
|---|---|---|---|---|---|
| storytold/photocraft | craft-print | `3d37da389a3618ab9b76109da11416e9ca4ea719` | 2026-10-13 | **diverged** — rev is NOT on `main`; it is the tip of an unmerged craft-print branch, 7 commits past the merge-base `666c358` (main has since advanced 11 commits on craft-cms/atomic) | [photocraft#4599](https://github.com/storytold/photocraft/pull/4599) |
| storytold/photocraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits behind (3 in `crates/ui`) | pin sweep |
| storytold/vectorcraft | craft-print | `d991bf6792f8ade99c51358bf3c1d902c8cd96b5` | 2026-10-12 | 58 commits behind (7 in `crates/print`, incl. the progress-listing merge `666c358`) | pin sweep |
| storytold/vectorcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits behind | pin sweep |
| storytold/designcraft | craft-print | `d991bf6792f8ade99c51358bf3c1d902c8cd96b5` | 2026-10-12 | 58 commits behind | pin sweep |
| storytold/pdfcraft | craft-print | `f03ec27ab2c405969428512178f746183cd55ca0` | 2026-10-12 | 14 commits behind (2 in `crates/print`: `ebc4d8c` caller-thread progress, `4af92ad` submitter callbacks) | pin sweep (bumped by [pdfcraft#1132](https://github.com/storytold/pdfcraft/pull/1132), prior window) |
| storytold/pdfcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits behind | pin sweep |
| storytold/lightcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits behind | pin sweep |
| storytold/filmcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits behind (pinned `version = "0.1.0"`, `features = ["serde"]`) | pin sweep |
| storytold/effectcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits behind (`features = ["serde"]`) | pin sweep |
| storytold/cadcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits behind (`features = ["serde"]`) | pin sweep |
| storytold/soundcraft | craft-ui | `5c874d51eeb3de70e735c8ce87a59a7dedd47675` | 2026-10-10 | 76 commits behind (`features = ["serde"]`) | pin sweep |

Repos carrying no `craft-*` git pins in their root `Cargo.toml`:
deckcraft, gridcraft, wordcraft, artcraft, artcraft-services,
craft-repo-status-app, craft-launcher, craft-fonts.

(The 2026-10-14 02:51 UTC ledger missed filmcraft, effectcraft, cadcraft
and soundcraft — their single-line `craft-ui = { version = …, rev = … }`
entries defeat a line-oriented grep. This matrix is complete: 8 apps pin
craft-ui, 4 pin craft-print.)

## What moved since the pins

- `crates/print` after `d991bf67…`: 7 commits, ending at `4af92ad`
  (`bridge native page processing to submitter callbacks`), including
  `666c358` — Windows print progress listing (`crates/print/src/progress.rs`).
- After `f03ec27…` (pdfcraft): 2 print commits (`ebc4d8c`, `4af92ad`).
- `crates/ui` after `5c874d51…`: 3 commits (shared printer settings,
  async job feedback, accessible print control labels).
- **Divergence:** photocraft#4599 pinned `3d37da38`, which is not on
  `main`. `main..3d37da38` is 7 commits: `ccb1eae` (refuse fitting that
  would discard annotations), `8fb8956` (`craft_print::rgb` — shared
  safe driver settings + RGB transport, via the new
  `craft-winprint-gdi` git dependency), `6ba4d0b`/`3929c3d`/`d873514`
  (refuse driver-dropped print options and annotation-visibility
  changes; README documentation), plus the merge itself. `main` and the
  branch fork at `666c358`; since then `main` advanced 11 commits
  (craft-cms extraction and hardening, `dd9e3f1`…`e9546e1`). No app can
  pin a rev that carries both.

## Evidence merges this window

- [photocraft#4599](https://github.com/storytold/photocraft/pull/4599) —
  "Share native Windows driver settings and RGB printing through
  craft-print" (merged 2026-10-14T04:38:25Z, merge
  `369704ac551e8e409800e423ff550c353d5522ef`): root `Cargo.toml` bumps
  the `craft-print` pin `d991bf67…` → `3d37da38…` (+1/−1);
  `crates/ui-egui/Cargo.toml` adds `craft-print = { workspace = true }`;
  21 `crates/ui-egui` files (+1,743/−49) wire the shared
  printer-settings surface. Companion merge
  [photocraft#3013](https://github.com/storytold/photocraft/pull/3013)
  (same minute) built the local GDI crate `crates/winprint`
  (`Cargo.toml` +26, `src/lib.rs` +284, `src/windows.rs` +474) whose
  upstreamed form is `craft-winprint-gdi`, pinned by `8fb8956` at rev
  `c32b8e717259d0a7589d301fa80122323b052012` of
  `storytold/winprint.rs`.
- The pin sweep above (read-only
  `contents/Cargo.toml` API reads of each repo's default branch) is the
  cross-check the scan mission requires for rev-sync rows.

## Outstanding sync proposals

1. **Land the diverged branch:** merge `3d37da38` (photocraft's pin)
   into `main` — until then `main` lacks code a shipping app already
   pins, and no common rev exists for the other print consumers.
2. **pdfcraft** → craft-print `e9546e1…` (or the post-merge HEAD):
   currently `f03ec27…`, missing `ebc4d8c` + `4af92ad`.
3. **vectorcraft, designcraft** → same: currently `d991bf67…`, missing
   7 print commits including the progress-listing surface (`666c358`).
4. **All eight craft-ui apps** (photocraft, vectorcraft, pdfcraft,
   lightcraft, filmcraft, effectcraft, cadcraft, soundcraft) →
   `e9546e1…`: currently `5c874d51…`, 76 commits behind.

Each of these is a one-line pin bump in the app's root `Cargo.toml`
plus `Cargo.lock` churn; apps should re-run their print/UI test slices
after bumping.

## Architecture compliance

This file is documentation only; it changes no crate. The sync actions
it proposes are app-side one-line pin bumps; nothing here violates the
AGENTS.md scope policy (no universal renderer, no i18n extraction, no
font binaries), and storytold/* repositories were read via the GitHub
API only. The scan writes only branches and pull requests on this
fork.
