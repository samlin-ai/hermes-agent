# hermes_cli/proxy Design Documentation

## Goal
The goal of the `hermes_cli/proxy` directory is to implement a local, OpenAI-compatible HTTP proxy server that forwards client requests to upstream AI models using the user's active, authenticated OAuth provider subscriptions (such as Nous Portal or xAI Grok).

By acting as a credential-attaching forwarder, it enables external applications (e.g., Open WebUI, LibreChat) to query a local endpoint (`127.0.0.1:8645` by default) without needing static API keys. The proxy strips incoming client authorization headers, attaches the resolved upstream bearer token, forwards requests to the upstream API, handles client-transparent retry logic (e.g., rotating credentials on rate limits or authorization failures), and streams the response back.

## File Enumeration
* [__init__.py](file:///home/castincar/hermes-agent/hermes_cli/proxy/__init__.py): Packages the module and exposes the base `UpstreamAdapter` class for defining upstream providers.
* [cli.py](file:///home/castincar/hermes-agent/hermes_cli/proxy/cli.py): Implements the CLI interface for the `hermes proxy` command group, defining subcommands to start the proxy (`start`), inspect adapter statuses (`status`), and list available upstream providers (`providers`).
* [server.py](file:///home/castincar/hermes-agent/hermes_cli/proxy/server.py): Implements the core aiohttp-based HTTP proxy server that routes traffic, filters request/response headers, manages token attachments, interacts with the chosen adapter to rotate credentials on 401/429 responses, and streams responses chunk-by-chunk.
* [adapters/](file:///home/castincar/hermes-agent/hermes_cli/proxy/adapters): Subdirectory containing vendor-specific token resolvers, token validation, and retry handlers. For details, see [adapters/DESIGN.md](file:///home/castincar/hermes-agent/hermes_cli/proxy/adapters/DESIGN.md).

## Workflow
The sequence diagram below shows how the CLI initializes the server and how an incoming HTTP request is proxied to the upstream provider using the active adapter's credentials.

```mermaid
sequenceDiagram
    participant User as User / CLI Command
    participant CLI as cli.py (cmd_proxy_start)
    participant Server as server.py (run_server)
    participant Adapter as Upstream Adapter (adapters/*)
    participant Client as Client Application (e.g., Open WebUI)
    participant Upstream as Upstream API (Nous/xAI)

    User->>CLI: hermes proxy start --provider <name>
    CLI->>Adapter: is_authenticated()
    Adapter-->>CLI: True
    CLI->>Server: run_server(adapter, host, port)
    activate Server
    Server-->>CLI: Listening on http://host:port/v1
    
    Client->>Server: POST /v1/chat/completions (with arbitrary bearer token)
    Server->>Adapter: check path /chat/completions in allowed_paths
    alt path allowed
        Server->>Adapter: get_credential()
        Adapter-->>Server: UpstreamCredential(bearer, base_url)
        Server->>Upstream: Forwarded request with Bearer token
        alt Upstream returns 401 or 429
            Upstream-->>Server: Error (401/429)
            Server->>Adapter: get_retry_credential(failed_credential, status_code)
            Adapter-->>Server: Retry UpstreamCredential (or None)
            alt Retry credential exists
                Server->>Upstream: Retry request with new Bearer token
                Upstream-->>Server: Success response
            end
        else Upstream returns 200 OK
            Upstream-->>Server: Success response
        end
        Server-->>Client: Streamed chunked response (SSE)
    else path not allowed
        Server-->>Client: 404 Path not allowed
    end
    deactivate Server
```

## System Architecture
The block diagram below describes the relationship between the proxy command interface, the HTTP forwarding server, the upstream adapters, and the credential stores.

```
+-------------------------------------------------------------+
|                     hermes CLI / Core                       |
|           (e.g., hermes command line entrypoint)            |
+------------------------------+------------------------------+
                               |
                               | invokes subcommand
                               v
+-------------------------------------------------------------+
|                      hermes_cli/proxy/                      |
|                                                             |
|   +------------------+             +--------------------+   |
|   |      cli.py      |------------>|     server.py      |   |
|   | (CLI Subcommands)|             |  (aiohttp Server)  |   |
|   +--------+---------+             +---------+----------+   |
|            |                                 |              |
|            | resolves                        | uses         |
|            v                                 v              |
|   +-----------------------------------------------------+   |
|   |                     adapters/                       |   |
|   |                                                     |   |
|   |         +---------------------------------+         |   |
|   |         |             base.py             |         |   |
|   |         | (UpstreamAdapter Interface)     |         |   |
|   |         +--------+------------------+-----+         |   |
|   |                  ^                  ^               |   |
|   |                  | inherits         | inherits      |   |
|   |         +--------+---------+  +-----+----------+    |   |
|   |         |  nous_portal.py  |  |     xai.py     |    |   |
|   |         +------------------+  +----------------+    |   |
|   +-----------------------------------------------------+   |
+-------------------------------------------------------------+
                               |
             interacts with    v
        +-------------------------------------------+
        | Local Authentication Stores               |
        | (~/.hermes/auth.json / Credential Pool)   |
        +-------------------------------------------+
```
