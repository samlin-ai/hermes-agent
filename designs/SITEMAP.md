# Design Artifacts Sitemap

This sitemap acts as a comprehensive, well-structured index of all system design documentation within the Hermes Agent codebase. All design documentation files (`DESIGN.md`) have been consolidated under the [designs](file:///home/castincar/hermes-agent/designs) directory mirroring the repository's layout.

---

## 1. Global & Orchestration

*   **Project Root Design**
    *   **File:** [designs/DESIGN.md](file:///home/castincar/hermes-agent/designs/DESIGN.md)
    *   **Code Directory:** [/home/castincar/hermes-agent/](file:///home/castincar/hermes-agent/)
    *   **Description:** The root orchestration hub of Hermes. Explains CLI, gateways, state persistence, LLM tool definitions, and main execution entry points.

---

## 2. Core Agent Intelligence

*   **Agent Core Engine**
    *   **File:** [designs/agent/DESIGN.md](file:///home/castincar/hermes-agent/designs/agent/DESIGN.md)
    *   **Code Directory:** [agent/](file:///home/castincar/hermes-agent/agent/)
    *   **Description:** The core execution and intelligence engine of the Hermes Agent. Handles context management, rate-limiting, and main conversation flow logic.
*   **Language Server Protocol (LSP)**
    *   **File:** [designs/agent/lsp/DESIGN.md](file:///home/castincar/hermes-agent/designs/agent/lsp/DESIGN.md)
    *   **Code Directory:** [agent/lsp/](file:///home/castincar/hermes-agent/agent/lsp/)
    *   **Description:** Language Server Protocol integration providing diagnostics, automatic server bootstrapping, and remapping.
*   **Secret Sources**
    *   **File:** [designs/agent/secret_sources/DESIGN.md](file:///home/castincar/hermes-agent/designs/agent/secret_sources/DESIGN.md)
    *   **Code Directory:** [agent/secret_sources/](file:///home/castincar/hermes-agent/agent/secret_sources/)
    *   **Description:** External secrets integration (e.g., Bitwarden) to securely fetch API keys at runtime.
*   **Transports & Providers**
    *   **File:** [designs/agent/transports/DESIGN.md](file:///home/castincar/hermes-agent/designs/agent/transports/DESIGN.md)
    *   **Code Directory:** [agent/transports/](file:///home/castincar/hermes-agent/agent/transports/)
    *   **Description:** Modular transport/adapter layer abstracting LLM providers (Gemini, Anthropic, Bedrock, etc.) and MCP gateway connectivity.

---

## 3. User Interfaces & Frontends

*   **Hermes CLI**
    *   **File:** [designs/hermes_cli/DESIGN.md](file:///home/castincar/hermes-agent/designs/hermes_cli/DESIGN.md)
    *   **Code Directory:** [hermes_cli/](file:///home/castincar/hermes-agent/hermes_cli/)
    *   **Description:** Exposes user-facing terminal interactions, argparse hierarchies, configuration wizards, and local HTTP proxy endpoints.
*   **CLI Subcommands**
    *   **File:** [designs/hermes_cli/subcommands/DESIGN.md](file:///home/castincar/hermes-agent/designs/hermes_cli/subcommands/DESIGN.md)
    *   **Code Directory:** [hermes_cli/subcommands/](file:///home/castincar/hermes-agent/hermes_cli/subcommands/)
    *   **Description:** Handles modular CLI subcommands processing options, inputs, and plugin hooks.
*   **Local OpenAI-Compatible Proxy**
    *   **File:** [designs/hermes_cli/proxy/DESIGN.md](file:///home/castincar/hermes-agent/designs/hermes_cli/proxy/DESIGN.md)
    *   **Code Directory:** [hermes_cli/proxy/](file:///home/castincar/hermes-agent/hermes_cli/proxy/)
    *   **Description:** A local proxy translating OpenAI API request schemas into internal Hermes tool actions.
*   **Proxy Vendor Adapters**
    *   **File:** [designs/hermes_cli/proxy/adapters/DESIGN.md](file:///home/castincar/hermes-agent/designs/hermes_cli/proxy/adapters/DESIGN.md)
    *   **Code Directory:** [hermes_cli/proxy/adapters/](file:///home/castincar/hermes-agent/hermes_cli/proxy/adapters/)
    *   **Description:** Unified, vendor-agnostic provider translation layer within the proxy service.
*   **Dashboard Authentication**
    *   **File:** [designs/hermes_cli/dashboard_auth/DESIGN.md](file:///home/castincar/hermes-agent/designs/hermes_cli/dashboard_auth/DESIGN.md)
    *   **Code Directory:** [hermes_cli/dashboard_auth/](file:///home/castincar/hermes-agent/hermes_cli/dashboard_auth/)
    *   **Description:** Modular authentication framework for web dashboards and local user login gates.
*   **TUI WebSocket Gateway**
    *   **File:** [designs/tui_gateway/DESIGN.md](file:///home/castincar/hermes-agent/designs/tui_gateway/DESIGN.md)
    *   **Code Directory:** [tui_gateway/](file:///home/castincar/hermes-agent/tui_gateway/)
    *   **Description:** Bidirectional WebSocket / stdio RPC broker backing the Ink-based terminal UI dashboard.
*   **Desktop App System**
    *   **File:** [designs/apps/desktop/DESIGN.md](file:///home/castincar/hermes-agent/designs/apps/desktop/DESIGN.md)
    *   **Code Directory:** [apps/desktop/](file:///home/castincar/hermes-agent/apps/desktop/)
    *   **Description:** Electron application layouts, theme structures, primitive rendering elements, and localized strings.

---

## 4. Messaging & Integration Gateway

*   **Gateway Interface**
    *   **File:** [designs/gateway/DESIGN.md](file:///home/castincar/hermes-agent/designs/gateway/DESIGN.md)
    *   **Code Directory:** [gateway/](file:///home/castincar/hermes-agent/gateway/)
    *   **Description:** The multi-platform integration gateway bridges the core agent to chat client protocols.
*   **Built-in Hooks**
    *   **File:** [designs/gateway/builtin_hooks/DESIGN.md](file:///home/castincar/hermes-agent/designs/gateway/builtin_hooks/DESIGN.md)
    *   **Code Directory:** [gateway/builtin_hooks/](file:///home/castincar/hermes-agent/gateway/builtin_hooks/)
    *   **Description:** Modular plugins and session intercepts acting automatically inside the gateway pipelines.
*   **Platform Adapters**
    *   **File:** [designs/gateway/platforms/DESIGN.md](file:///home/castincar/hermes-agent/designs/gateway/platforms/DESIGN.md)
    *   **Code Directory:** [gateway/platforms/](file:///home/castincar/hermes-agent/gateway/platforms/)
    *   **Description:** Platform abstraction handlers mapping messaging interactions (e.g., Slack, Telegram, Discord).
*   **QQBot Adapter**
    *   **File:** [designs/gateway/platforms/qqbot/DESIGN.md](file:///home/castincar/hermes-agent/designs/gateway/platforms/qqbot/DESIGN.md)
    *   **Code Directory:** [gateway/platforms/qqbot/](file:///home/castincar/hermes-agent/gateway/platforms/qqbot/)
    *   **Description:** Platform implementation translating QQ chat group messages and webhook events.

---

## 5. Capabilities, Tools & Sandboxes

*   **Tools Orchestration**
    *   **File:** [designs/tools/DESIGN.md](file:///home/castincar/hermes-agent/designs/tools/DESIGN.md)
    *   **Code Directory:** [tools/](file:///home/castincar/hermes-agent/tools/)
    *   **Description:** Extensibility layer managing path safety, action validation, and LLM-facing schemas.
*   **Target Environments & Sandboxes**
    *   **File:** [designs/tools/environments/DESIGN.md](file:///home/castincar/hermes-agent/designs/tools/environments/DESIGN.md)
    *   **Code Directory:** [tools/environments/](file:///home/castincar/hermes-agent/tools/environments/)
    *   **Description:** Sandbox controllers running tool processes locally or inside Docker, SSH, Modal, or Daytona.
*   **Computer Use (OSWorld navigation)**
    *   **File:** [designs/tools/computer_use/DESIGN.md](file:///home/castincar/hermes-agent/designs/tools/computer_use/DESIGN.md)
    *   **Code Directory:** [tools/computer_use/](file:///home/castincar/hermes-agent/tools/computer_use/)
    *   **Description:** Model-agnostic desktop UI interactions, screenshot overlays, and mouse/keyboard navigation.
*   **Agent Communication Protocol (ACP)**
    *   **File:** [designs/acp_adapter/DESIGN.md](file:///home/castincar/hermes-agent/designs/acp_adapter/DESIGN.md)
    *   **Code Directory:** [acp_adapter/](file:///home/castincar/hermes-agent/acp_adapter/)
    *   **Description:** Exposes standard stdio JSON-RPC adapter endpoint connecting Hermes with Zed, VS Code, and IDE plugins.

---

## 6. Automations & Scheduling

*   **Cron Scheduler**
    *   **File:** [designs/cron/DESIGN.md](file:///home/castincar/hermes-agent/designs/cron/DESIGN.md)
    *   **Code Directory:** [cron/](file:///home/castincar/hermes-agent/cron/)
    *   **Description:** Scheduled automation daemon running periodic queries, background scans, and command workflows.
*   **Cron Scripts**
    *   **File:** [designs/cron/scripts/DESIGN.md](file:///home/castincar/hermes-agent/designs/cron/scripts/DESIGN.md)
    *   **Code Directory:** [cron/scripts/](file:///home/castincar/hermes-agent/cron/scripts/)
    *   **Description:** Dedicated Python and shell utility scripts invoked by the scheduler loop.
