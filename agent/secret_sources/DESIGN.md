# agent/secret_sources Design Documentation

## Goal
The `agent/secret_sources` directory manages the integration of external secret sources to securely supply environment-variable-shaped credentials at process startup. By retrieving secrets dynamically, Hermes avoids the need to store sensitive keys (e.g., LLM provider API keys) in plaintext in the user's `~/.hermes/.env` file. These integrations run non-destructively: they only populate environment variables that are not already defined, ensuring that local shell exports and `.env` file settings retain precedence.

## File Enumeration
* [`__init__.py`](./__init__.py)
  Package docstring only. Defines what a "secret source" is (an env-var-shaped credential supplier that runs after `~/.hermes/.env` loads and is non-destructive by default) and points at the one shipped source (`bitwarden`) and its CLI wizard (`hermes_cli.secrets_cli`).
* [`bitwarden.py`](./bitwarden.py)
  Bitwarden Secrets Manager integration via the `bws` CLI. Responsibilities:
  - **Binary management** — `find_bws()` resolves the managed copy (`<hermes_home>/bin/bws`) first, then system PATH; `install_bws()` downloads the pinned version (`_BWS_VERSION = 2.0.0`) for the current platform/arch/libc from GitHub Releases, verifies its SHA-256 against the published checksum file, guards against zip-slip (`_safe_extract_member`), and installs it atomically.
  - **Fetch** — `fetch_bitwarden_secrets()` runs `bws secret list <project_id> --output json` as a subprocess (`BWS_ACCESS_TOKEN`, optional `BWS_SERVER_URL` for region/self-host), parses the JSON list, and keeps only valid env-var names.
  - **Two-layer cache** — an in-process dict (`_CACHE`) and a disk-persisted JSON file (`<hermes_home>/cache/bws_cache.json`, mode 0600, secret values only, never the token), both sharing one TTL.
  - **Apply** — `apply_bitwarden_secrets()` is the env_loader entry point; it injects fetched keys into `os.environ` non-destructively (won't override existing vars unless `override_existing`, never clobbers the access-token env var) and never raises — failures come back as a `FetchResult` with `error` set.
  - **Test hook** — `_reset_cache_for_tests()` clears both cache layers.

## Workflow
The sequence diagram below shows how credentials are loaded and cached from Bitwarden Secrets Manager during process startup.

```mermaid
sequenceDiagram
    participant EL as hermes_cli/env_loader.py
    participant BW as agent/secret_sources/bitwarden.py
    participant C as Cache (In-Memory / Disk)
    participant BWS as bws (Bitwarden CLI Subprocess)
    participant ENV as os.environ

    EL->>BW: apply_bitwarden_secrets(...)
    activate BW
    Note over BW: read BWS_ACCESS_TOKEN + project_id;<br/>find_bws(install_if_missing)
    BW->>C: L1 in-process, then L2 disk (is_fresh?)
    alt Cache Hit (fresh)
        C-->>BW: Return cached secrets (promote L2->L1)
    else Cache Miss (stale or missing)
        BW->>BWS: bws secret list <project_id> --output json
        activate BWS
        BWS-->>BW: JSON secrets output
        deactivate BWS
        BW->>C: Write L1 + L2 (0600, values only)
    end
    loop each fetched key
        BW->>ENV: set if absent (skip token env / existing unless override)
    end
    BW-->>EL: FetchResult (applied, skipped, warnings, error)
    deactivate BW
```

## System Architecture
The relationships between the secrets module, CLI command layers, local storage, and the upstream Bitwarden endpoint are illustrated below.

```
+-----------------------------------------------------------------------------------+
|                                 Hermes Core / CLI                                 |
|                                                                                   |
|  +---------------------------+             +----------------------------------+   |
|  | hermes_cli/env_loader.py  |             |    hermes_cli/secrets_cli.py     |   |
|  +-------------+-------------+             +-----------------+----------------+   |
|                |                                             |                    |
|                | (Calls apply_bitwarden_secrets)             | (CLI Setup/Sync/   |
|                v                                             v Status/Install)    |
|  +-----------------------------------------------------------+----------------+   |
|  |                        agent/secret_sources/bitwarden.py                   |   |
|  +---------+--------------------+----------------------------+----------------+   |
|            |                    |                            |                    |
|            | (downloads binary) | (invokes)                  | (reads/writes)     |
|            v                    v                            v                    |
|    +---------------+    +---------------+      +----------------------------+     |
|    | GitHub        |    | bws binary    |      | Cache (two layers, 1 TTL): |     |
|    | Releases      |    | (Subprocess)  |      |  L1 _CACHE (in-process)    |     |
|    | (zip + sha256)|    +-------+-------+      |  L2 cache/bws_cache.json   |     |
|    +---------------+            |             |     (disk, 0600)           |     |
|                                 |             +----------------------------+     |
|                                 |                                                 |
|                                 | (HTTP API)                                      |
|                                 v                                                 |
|                      +--------------------+                                       |
|                      |  Bitwarden Cloud / |                                       |
|                      |  Self-hosted Vault |                                       |
|                      +--------------------+                                       |
+-----------------------------------------------------------------------------------+
```
