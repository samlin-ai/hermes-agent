# Design Artifacts Sitemap

This sitemap acts as a comprehensive, well-structured index of all system design documentation within the Hermes Agent codebase. All design documentation files (`DESIGN.md`) have been consolidated under the [designs](./) directory mirroring the repository's layout.

---

## 1. Global & Orchestration

*   **Project Root Design**
    *   **File:** [designs/DESIGN.md](DESIGN.md)
    *   **Code Directory:** [hermes-agent/](../)
    *   **Description:** The root orchestration hub of Hermes. Explains CLI, gateways, state persistence, LLM tool definitions, and main execution entry points.

---

## 2. Core Agent Intelligence

*   **Agent Core Engine**
    *   **File:** [designs/agent/DESIGN.md](agent/DESIGN.md)
    *   **Code Directory:** [agent/](../agent/)
    *   **Description:** The core execution and intelligence engine of the Hermes Agent. Handles context management, rate-limiting, and main conversation flow logic.
*   **Language Server Protocol (LSP)**
    *   **File:** [designs/agent/lsp/DESIGN.md](agent/lsp/DESIGN.md)
    *   **Code Directory:** [agent/lsp/](../agent/lsp/)
    *   **Description:** Language Server Protocol integration providing diagnostics, automatic server bootstrapping, and remapping.
*   **Secret Sources**
    *   **File:** [designs/agent/secret_sources/DESIGN.md](agent/secret_sources/DESIGN.md)
    *   **Code Directory:** [agent/secret_sources/](../agent/secret_sources/)
    *   **Description:** External secrets integration (e.g., Bitwarden) to securely fetch API keys at runtime.
*   **Transports & Providers**
    *   **File:** [designs/agent/transports/DESIGN.md](agent/transports/DESIGN.md)
    *   **Code Directory:** [agent/transports/](../agent/transports/)
    *   **Description:** Modular transport/adapter layer abstracting LLM providers (Gemini, Anthropic, Bedrock, etc.) and MCP gateway connectivity.

---

## 3. User Interfaces & Frontends

*   **Hermes CLI**
    *   **File:** [designs/hermes_cli/DESIGN.md](hermes_cli/DESIGN.md)
    *   **Code Directory:** [hermes_cli/](../hermes_cli/)
    *   **Description:** Exposes user-facing terminal interactions, argparse hierarchies, configuration wizards, and local HTTP proxy endpoints.
*   **CLI Subcommands**
    *   **File:** [designs/hermes_cli/subcommands/DESIGN.md](hermes_cli/subcommands/DESIGN.md)
    *   **Code Directory:** [hermes_cli/subcommands/](../hermes_cli/subcommands/)
    *   **Description:** Handles modular CLI subcommands processing options, inputs, and plugin hooks.
*   **Local OpenAI-Compatible Proxy**
    *   **File:** [designs/hermes_cli/proxy/DESIGN.md](hermes_cli/proxy/DESIGN.md)
    *   **Code Directory:** [hermes_cli/proxy/](../hermes_cli/proxy/)
    *   **Description:** A local proxy translating OpenAI API request schemas into internal Hermes tool actions.
*   **Proxy Vendor Adapters**
    *   **File:** [designs/hermes_cli/proxy/adapters/DESIGN.md](hermes_cli/proxy/adapters/DESIGN.md)
    *   **Code Directory:** [hermes_cli/proxy/adapters/](../hermes_cli/proxy/adapters/)
    *   **Description:** Unified, vendor-agnostic provider translation layer within the proxy service.
*   **Dashboard Authentication**
    *   **File:** [designs/hermes_cli/dashboard_auth/DESIGN.md](hermes_cli/dashboard_auth/DESIGN.md)
    *   **Code Directory:** [hermes_cli/dashboard_auth/](../hermes_cli/dashboard_auth/)
    *   **Description:** Modular authentication framework for web dashboards and local user login gates.
*   **TUI WebSocket Gateway**
    *   **File:** [designs/tui_gateway/DESIGN.md](tui_gateway/DESIGN.md)
    *   **Code Directory:** [tui_gateway/](../tui_gateway/)
    *   **Description:** Bidirectional WebSocket / stdio RPC broker backing the Ink-based terminal UI dashboard.
*   **Desktop App System**
    *   **File:** [designs/apps/desktop/DESIGN.md](apps/desktop/DESIGN.md)
    *   **Code Directory:** [apps/desktop/](../apps/desktop/)
    *   **Description:** Electron application layouts, theme structures, primitive rendering elements, and localized strings.

---

## 4. Messaging & Integration Gateway

*   **Gateway Interface**
    *   **File:** [designs/gateway/DESIGN.md](gateway/DESIGN.md)
    *   **Code Directory:** [gateway/](../gateway/)
    *   **Description:** The multi-platform integration gateway bridges the core agent to chat client protocols.
*   **Built-in Hooks**
    *   **File:** [designs/gateway/builtin_hooks/DESIGN.md](gateway/builtin_hooks/DESIGN.md)
    *   **Code Directory:** [gateway/builtin_hooks/](../gateway/builtin_hooks/)
    *   **Description:** Modular plugins and session intercepts acting automatically inside the gateway pipelines.
*   **Platform Adapters**
    *   **File:** [designs/gateway/platforms/DESIGN.md](gateway/platforms/DESIGN.md)
    *   **Code Directory:** [gateway/platforms/](../gateway/platforms/)
    *   **Description:** Platform abstraction handlers mapping messaging interactions (e.g., Slack, Telegram, Discord).
*   **QQBot Adapter**
    *   **File:** [designs/gateway/platforms/qqbot/DESIGN.md](gateway/platforms/qqbot/DESIGN.md)
    *   **Code Directory:** [gateway/platforms/qqbot/](../gateway/platforms/qqbot/)
    *   **Description:** Platform implementation translating QQ chat group messages and webhook events.

---

## 5. Capabilities, Tools & Sandboxes

*   **Tools Orchestration**
    *   **File:** [designs/tools/DESIGN.md](tools/DESIGN.md)
    *   **Code Directory:** [tools/](../tools/)
    *   **Description:** Extensibility layer managing path safety, action validation, and LLM-facing schemas.
*   **Target Environments & Sandboxes**
    *   **File:** [designs/tools/environments/DESIGN.md](tools/environments/DESIGN.md)
    *   **Code Directory:** [tools/environments/](../tools/environments/)
    *   **Description:** Sandbox controllers running tool processes locally or inside Docker, SSH, Modal, or Daytona.
*   **Computer Use (OSWorld navigation)**
    *   **File:** [designs/tools/computer_use/DESIGN.md](tools/computer_use/DESIGN.md)
    *   **Code Directory:** [tools/computer_use/](../tools/computer_use/)
    *   **Description:** Model-agnostic desktop UI interactions, screenshot overlays, and mouse/keyboard navigation.
*   **Agent Communication Protocol (ACP)**
    *   **File:** [designs/acp_adapter/DESIGN.md](acp_adapter/DESIGN.md)
    *   **Code Directory:** [acp_adapter/](../acp_adapter/)
    *   **Description:** Exposes standard stdio JSON-RPC adapter endpoint connecting Hermes with Zed, VS Code, and IDE plugins.

---

## 6. Automations & Scheduling

*   **Cron Scheduler**
    *   **File:** [designs/cron/DESIGN.md](cron/DESIGN.md)
    *   **Code Directory:** [cron/](../cron/)
    *   **Description:** Scheduled automation daemon running periodic queries, background scans, and command workflows.
*   **Cron Scripts**
    *   **File:** [designs/cron/scripts/DESIGN.md](cron/scripts/DESIGN.md)
    *   **Code Directory:** [cron/scripts/](../cron/scripts/)
    *   **Description:** Dedicated Python and shell utility scripts invoked by the scheduler loop.
