# agent/lsp Design Documentation

## Goal
The `agent/lsp` directory implements the Language Server Protocol (LSP) integration for the Hermes Agent. Its primary purpose is to provide post-write semantic lints (diagnostics) when the agent edits codebase files (via the `write_file` / `patch` tools, in `tools/file_operations.py`).

Main tasks performed by this subsystem:
*   **Workspace gating:** LSP only activates when the file (or the agent's cwd) sits inside a Git worktree, so non-project folders (e.g. user-home Telegram gateway chats) never spawn daemons. Outside a worktree the caller falls back to its in-process syntax check.
*   **Server management:** Resolves a file to its language server, walks to the per-server project root (e.g. `pyproject.toml`, `Cargo.toml`, `go.mod`), spawns/re-uses one client per `(server_id, workspace_root)`, and lazily auto-installs missing binaries into a sandboxed staging dir (`<HERMES_HOME>/lsp/bin/`).
*   **JSON-RPC client:** Frames and exchanges JSON-RPC 2.0 over the server's stdin/stdout; tracks document sync (`didOpen`/`didChange`/`didSave`), merges push + pull diagnostics, and waits for fresh diagnostics with a timeout budget.
*   **Delta filtering:** Snapshots a diagnostics baseline immediately before an edit and subtracts it from post-edit diagnostics so the agent only sees newly-introduced errors. Baseline diagnostics are line-shifted through the edit diff so pre-existing errors that merely moved aren't reported as new. The baseline then rolls forward for the next edit.
*   **Resilience:** A per-process **broken-set** records `(server_id, workspace_root)` pairs that failed to spawn/initialize or timed out; those pairs are skipped instantly (no retry) until `hermes lsp restart` or process exit.
*   **Formatted reporting:** Emits compact, severity-filtered, length-bounded `<diagnostics>` XML blocks for the model.

## File Enumeration
*   [\_\_init\_\_.py](../../../agent/lsp/__init__.py): Package entry point. Exposes the process-wide `LSPService` singleton via `get_service()` (lazy create from config; returns `None` when disabled/inactive), `shutdown_service()`, and an `atexit` hook that tears down spawned servers on clean Python exit. (Note: the module docstring's example API names are illustrative; the real service API is `is_active` / `enabled_for` / `snapshot_baseline` / `get_diagnostics_sync`.)
*   [cli.py](../../../agent/lsp/cli.py): Handlers for the `hermes lsp` subcommands — `status`, `list`, `install <id>`, `install-all`, `restart`, `which <id>`. Also surfaces backend-tool gaps (e.g. bash-language-server installed but `shellcheck` missing).
*   [client.py](../../../agent/lsp/client.py): `LSPClient` — one async client per `(server_id, workspace_root)`. Owns the child process, drives the initialize handshake, does whole-document text sync (always sends a full replacement even when the server advertises incremental), stores push (`publishDiagnostics`) and pull (`textDocument/diagnostic`) diagnostics merged/deduped by content, retries `ContentModified` (-32801) with exponential backoff, and implements `wait_for_diagnostics` (push-counter + pull race with a time budget). Also holds `file_uri`/`uri_to_path` helpers and the spawn/SIGTERM/SIGKILL shutdown path.
*   [eventlog.py](../../../agent/lsp/eventlog.py): Structured logging on the `hermes.lint.lsp` logger. Steady-state events log at DEBUG (silent at default INFO); state transitions (`active for <root>`, `no project root`) and action-required failures (`server unavailable`, spawn-failed, timeout) are deduped to fire once per session via module-level sets.
*   [install.py](../../../agent/lsp/install.py): Auto-install of server binaries. `INSTALL_RECIPES` maps recipe keys to a strategy (`npm`/`go`/`pip`/`manual`), package, and bin name; `try_install` installs into `<HERMES_HOME>/lsp/bin/` (npm via `--prefix` + symlink, go via `GOBIN`, pip via `--target`), deduped per-package with a lock. `detect_status` and `hermes_lsp_bin_dir` support the CLI. Heavy servers (rust-analyzer, clangd, lua-language-server) are marked `manual`.
*   [manager.py](../../../agent/lsp/manager.py): `LSPService` — bridges the synchronous file_operations layer and the async clients. Runs one asyncio loop on a background daemon thread (`_BackgroundLoop`). Owns the client map, broken-set, last-used times, and per-file delta baseline. Public surface: `is_active`, `enabled_for`, `snapshot_baseline`, `get_diagnostics_sync` (open + save + wait + delta-filter + roll baseline forward), `get_status`, `shutdown`. `_mark_broken_for_file` blacklists and reaps a failed pair on timeout/error.
*   [protocol.py](../../../agent/lsp/protocol.py): JSON-RPC 2.0 framing — `Content-Length` encode/read over async streams, request/notification/response/error builders, `classify_message`, error-code constants, and the `LSPProtocolError` / `LSPRequestError` exceptions.
*   [range_shift.py](../../../agent/lsp/range_shift.py): Builds a piecewise-linear line-remap from `difflib.SequenceMatcher.get_opcodes()` (`build_line_shift`) and applies it to baseline diagnostics (`shift_diagnostic_range`, `shift_baseline`); diagnostics on deleted/replaced lines drop out. Preserves the "same error at a new location is genuinely new" signal.
*   [reporter.py](../../../agent/lsp/reporter.py): Formats diagnostics into `<diagnostics file=...>` blocks (1-indexed line/col), ERROR-only by default, capped at `MAX_PER_FILE` (20) entries and `MAX_TOTAL_CHARS` (4000) via `truncate`.
*   [servers.py](../../../agent/lsp/servers.py): The server registry. `ServerDef` (extension matching, `resolve_root`, `build_spawn`), `ServerContext` (install policy + user overrides), `SpawnSpec`, per-language root resolvers and spawn builders, the `SERVERS` list (~26 servers), `find_server_for_file`, `language_id_for`, and the `LANGUAGE_BY_EXT` map.
*   [workspace.py](../../../agent/lsp/workspace.py): Workspace + project-root resolution. `find_git_worktree` (the gate, cached), `resolve_workspace_for_file` (cwd-first, file-location fallback), `nearest_root` (marker walk with exclude semantics, e.g. typescript skips Deno projects), `is_inside_workspace`, `normalize_path`, `clear_cache`.

## Workflow
Runtime flow of a single edit: pre-edit baseline snapshot, the write, then post-edit diagnostics with line-shift + delta filtering.

```mermaid
sequenceDiagram
    autonumber
    participant FO as file_operations (sync)
    participant SVC as LSPService (bg loop)
    participant Client as LSPClient (async)
    participant Proc as Server subprocess

    Note over FO, Proc: Phase 1 — Baseline snapshot (pre-edit)
    FO->>SVC: snapshot_baseline(path)
    activate SVC
    SVC->>Client: _get_or_spawn(path)
    activate Client
    Note over Client, Proc: Spawn + initialize if not running (else reuse; broken pairs skipped)
    Client->>Proc: spawn + initialize
    Proc-->>Client: initialize result
    Client-->>SVC: client (or None)
    deactivate Client
    SVC->>Client: open_file() + wait_for_diagnostics()
    Client->>Proc: didOpen
    Proc-->>Client: publishDiagnostics (push)
    SVC->>SVC: store baseline[path]
    deactivate SVC

    Note over FO: Phase 2 — Edit
    FO->>FO: write edits to disk

    Note over FO, Proc: Phase 3 — Post-edit diagnostics + delta filter
    FO->>SVC: get_diagnostics_sync(path, line_shift=build_line_shift(pre, post))
    activate SVC
    SVC->>Client: open_file() + save_file() + wait_for_diagnostics()
    Client->>Proc: didChange + didSave
    par
        Proc-->>Client: publishDiagnostics (push)
    and
        Client->>Proc: textDocument/diagnostic (pull)
        Proc-->>Client: diagnostic items
    end
    Client-->>SVC: merged + deduped diagnostics
    SVC->>SVC: shift_baseline(baseline, line_shift)
    SVC->>SVC: drop diagnostics whose key is in shifted baseline
    SVC->>SVC: roll baseline forward to current state
    SVC-->>FO: delta diagnostics
    deactivate SVC
    FO->>FO: reporter.report_for_file(...)
```

## System Architecture
File relationships and the external caller:

```
                    +------------------------------------+
                    |  tools.file_operations (caller)    |
                    +-----------------+------------------+
                                      | get_service()
                                      v
                    +-----------------+------------------+
                    |   agent.lsp (__init__.py)          |
                    |   singleton + atexit shutdown      |
                    +-----------------+------------------+
                                      |
                                      v
                    +-----------------+------------------+
                    |   manager (LSPService)             |
                    |   bg asyncio loop, client map,     |
                    |   broken-set, delta baseline       |
                    +----+-------------------------+-----+
                         |                         |
            find_server  |                         | per (server_id, ws_root)
            resolve_root  v                        v
              +----------+----------+    +---------+-----------+
              |   servers           |    |   client (LSPClient)|
              | ServerDef registry, |    | sync, push+pull     |
              | spawn builders      |    | diagnostics, retry  |
              +----+-----------+----+    +----+-----------+----+
                   |           |              |           | JSON-RPC
        nearest_root|          | build_spawn  | (stdin/stdout)
                    v          v              v           v
        +-----------+--+  +----+---------+  +-+-----+  +--+-----------------+
        |  workspace   |  |   install    |  |protocol| |  Server subprocess |
        | git-worktree |  | auto-install |  | framer | | pyright/gopls/...  |
        | gate + roots |  | to lsp/bin   |  +--------+ +--------------------+
        +--------------+  +--------------+

        - - - - - - - - - - support modules - - - - - - - - - -
        [range_shift] line-shift baseline diagnostics via diff opcodes
        [reporter]    format final delta diagnostics for the model
        [eventlog]    deduped structured logging (hermes.lint.lsp)
        [cli]         `hermes lsp` subcommands (status/install/restart/which)
```
