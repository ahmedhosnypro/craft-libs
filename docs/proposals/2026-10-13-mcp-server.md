# Proposal: `craft-mcp-server` — shared MCP server plumbing and tool registration

**Status:** proposal (evidence only; no code extraction in this phase — humans decide)
**Target crate name:** `craft-mcp-server` (`crates/mcp-server`, named per AGENTS.md)
**Date:** 2026-10-13 (window 2026-10-13T07:21:55Z → 2026-10-13T11:32:17Z UTC, 7h look-back)

## Why

Two apps built and hardened Model Context Protocol servers in the same window, with the
same file taxonomy and the same hardening needs — and soundcraft is visibly re-deriving
the structure cadcraft already extracted. The generic layer (JSON-RPC framing, protocol
handshake, tool registration/selection, file-access sandboxing) is identical across apps;
only the tool bodies differ. That is exactly the hoist bar: the plumbing is useful to more
than one app and depends on no app.

## Evidence (merged PRs, this window)

| PR | Merge (UTC) | What landed | Files |
|---|---|---|---|
| [storytold/soundcraft#309](https://github.com/storytold/soundcraft/pull/309) | 08:43:45 | MCP file-access confinement: `--automation-read-root/--automation-write-root` | `apps/soundcraft-cli/src/automation_args.rs` (+90 new), `apps/soundcraft-cli/src/main.rs` (+60/−4), `crates/engine/src/file_access.rs` (+487 new), `crates/automation/src/lib.rs` (+19/−4), `docs/mcp.md` (+68 new) |
| [storytold/cadcraft#756](https://github.com/storytold/cadcraft/pull/756) | 08:34:15 | `--lazy`/`--launch` for MCP clients that start before the app | `apps/cadcraft-cli/src/main.rs` (+28/−12), `crates/mcp/src/backend.rs` (+59/−2), `docs/mcp.md` (+9) |
| [storytold/cadcraft#757](https://github.com/storytold/cadcraft/pull/757) | 09:34:35 | Per-tool registration: `render_area`, `find_text` | `crates/engine/src/cmd/inquiry.rs` (+75 new), `crates/mcp/src/backend.rs` (+21/−2), `crates/mcp/src/tools.rs` (+9/−2), `crates/ui-egui/src/control.rs` (+22/−2), `docs/mcp.md` |
| [storytold/soundcraft#215](https://github.com/storytold/soundcraft/pull/215) | 10:30:18 | Control server: bounded request lines and concurrent connections | `apps/soundcraft/src/control_server.rs` (+160/−11) |
| [storytold/soundcraft#271](https://github.com/storytold/soundcraft/pull/271) | 08:26:33 | Control-surface fix: `engine.commands` filter applied to UI commands | `crates/ui-egui/src/control.rs` (+6/−1) |

All merges are in the master merge commit list; all file stats via
`gh api repos/storytold/<repo>/pulls/<n>/files`.

## What the shared crate would own

- JSON-RPC framing and protocol handshake (`initialize`, method dispatch, `ping`,
  capabilities — cadcraft#852's `control.methods`/`ping`).
- Tool registry: named tools with JSON schema, selection/enablement, per-tool dispatch
  (cadcraft's `crates/mcp/src/tools.rs` + `backend.rs` split).
- File-access sandbox: read/write root enforcement with path canonicalization and
  refusal diagnostics (soundcraft#309's `file_access.rs` semantics).
- Server lifecycle: `--lazy`/`--launch` semantics for clients that start before the app
  (cadcraft#756).
- Hardened transport defaults: bounded request lines, bounded concurrency
  (soundcraft#215).

## What stays in each app

- Tool *bodies*: every app's tools call its own engine (cadcraft's `inquiry.rs` tools read
  DWG entities; soundcraft's touch automation/session state). Bodies stay app-side; the
  registry, framing, sandbox and lifecycle are shared.
- App docs (each `docs/mcp.md` keeps its tool catalog) and i18n of tool strings.
- The apps' own `ui-egui` control surfaces (their `control.rs` wiring).

## File inventory (evidence)

| Source | Files |
|---|---|
| storytold/soundcraft#309 | `crates/engine/src/file_access.rs` (+487), `apps/soundcraft-cli/src/automation_args.rs` (+90), `apps/soundcraft-cli/src/main.rs`, `crates/automation/src/lib.rs`, `docs/mcp.md` |
| storytold/cadcraft#756 | `crates/mcp/src/backend.rs`, `apps/cadcraft-cli/src/main.rs`, `docs/mcp.md` |
| storytold/cadcraft#757 | `crates/mcp/src/backend.rs`, `crates/mcp/src/tools.rs`, `crates/engine/src/cmd/inquiry.rs`, `crates/ui-egui/src/control.rs`, `docs/mcp.md` |
| storytold/soundcraft#215 | `apps/soundcraft/src/control_server.rs` (+160/−11) |
| storytold/soundcraft#271 | `crates/ui-egui/src/control.rs` (+6/−1) |

## NOTICE provenance draft

```
The MCP server framing, tool registry, file-access sandbox and server lifecycle in
crates/mcp-server is adapted from:
- SoundCraft, crates/engine/src/file_access.rs and
  apps/soundcraft-cli/src/automation_args.rs, revision
  5495378d7a4f89e64985a8e03dc809fd3e7c44e1 (PR #309).
  https://github.com/storytold/soundcraft
  Copyright (c) 2026 ArtCraft Team and the SoundCraft contributors.
- CadCraft, crates/mcp/src/backend.rs, crates/mcp/src/tools.rs and
  apps/cadcraft-cli/src/main.rs, revisions
  483a9304ca96bbbdd1c76c0affb7077cba462eea (PR #756) and
  8abc0909f3f29776355e9138c1556acf344d4f8f (PR #757).
  https://github.com/storytold/cadcraft
  Copyright (c) 2026 ArtCraft Team and the CadCraft contributors.
All sources listed above are licensed MIT OR Apache-2.0.
```

Pattern precedent: the NOTICE already records this style of per-revision attribution for
`crates/print` and `crates/ui` (see NOTICE "Source provenance").

## Architecture compliance

Per the target's AGENTS.md: no app code in the shared crate — tool bodies stay in each
app's engine; the crate depends on no app and is useful to more than one app (two
demonstrated, same window). New crate under `crates/<name>`, named `craft-<name>`,
inheriting workspace fields and lints, registered in `[workspace.dependencies]`.
Scope policy respected: this is protocol plumbing, not a universal renderer, not i18n
extraction, not font binaries. Gate commands before merging: `cargo fmt --check`,
`cargo clippy --workspace --all-targets -- -D warnings`, `cargo test --workspace`.
