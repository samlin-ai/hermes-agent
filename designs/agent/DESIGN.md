# agent Design Documentation

## Goal
The `agent` directory is the core execution and intelligence engine of the Hermes Agent. It coordinates model initialization, request/response cycle execution, credential lifecycle management, prompt construction, context optimization (such as compaction/compression), tool dispatching/execution, external memory sync, and telemetry tracking.

The main tasks performed by this subsystem include:
*   **Conversation Orchestration:** Managing the main turn execution loop, tracking retry and budget limits, and finalizing session states.
*   **Context & Prompt Engineering:** Assembling system prompts dynamically (incorporating active tools, local files, system context, and user-provided skills) and automatically compressing prompt context when approaching model window limits to protect cache properties.
*   **Multi-Provider Translation:** Normalizing raw parameters and response payloads across diverse LLMs (Anthropic, Gemini, Bedrock, Codex, OpenAI-compatible) via adapters and transports.
*   **Tool Execution & Guardrails:** Running tool calls sequentially or concurrently, validating paths against workspace scopes, enforcing security boundaries, and mapping diagnostics via Language Server Protocol (LSP) post-write checks.
*   **Credential Lifecycle & Vault Integration:** Dynamically pulling API secrets from external managers and rotating credential keys across multi-slot pools to gracefully recover from rate limits.
*   **Skill & Memory Management:** Loading platform-specific custom skills and syncing context with semantic memory engines.

---

## File Enumeration

### Subdirectories (with existing Design Documents)
*   [lsp](file:///home/castincar/hermes-agent/designs/agent/lsp/DESIGN.md): Language Server Protocol (LSP) integration. Provides workspace-aware diagnostics, automatic server bootstrapping, differential diagnostic filtering (delta matching), and remapping of line numbers on modified code files.
*   [secret_sources](file:///home/castincar/hermes-agent/designs/agent/secret_sources/DESIGN.md): Integrates external secret sources (like Bitwarden Secrets Manager) to securely fetch API keys at startup, bypassing the need for plaintext `.env` storage.
*   [transports](file:///home/castincar/hermes-agent/designs/agent/transports/DESIGN.md): Standardized communication interface abstracting provider-specific schemas. Handles translation, token-counting, and houses the Codex App-Server JSON-RPC execution session and MCP bridge.

### Python Modules
*   [__init__.py](file:///home/castincar/hermes-agent/agent/__init__.py): Exposes package entry points and triggers module-level preloads.
*   [account_usage.py](file:///home/castincar/hermes-agent/agent/account_usage.py): Tracks token counts, request totals, and cost metrics for active LLM accounts.
*   [agent_init.py](file:///home/castincar/hermes-agent/agent/agent_init.py): Implements agent instantiation and parses configurations, endpoints, timeouts, and guardrails to build an `AIAgent`.
*   [agent_runtime_helpers.py](file:///home/castincar/hermes-agent/agent/agent_runtime_helpers.py): Houses utility functions for trajectory serialization, JSON argument repair, user-message merging, and failover transport recovery.
*   [anthropic_adapter.py](file:///home/castincar/hermes-agent/agent/anthropic_adapter.py): Translates messages and tool schemas specifically for Anthropic's Claude API format, managing cache breakpoints and system/thinking flags.
*   [async_utils.py](file:///home/castincar/hermes-agent/agent/async_utils.py): Provides async wrappers for calling blocking synchronous code inside thread pools.
*   [auxiliary_client.py](file:///home/castincar/hermes-agent/agent/auxiliary_client.py): Provides a stateless helper client to invoke fast/cheap background model calls for summary, review, or curation.
*   [azure_identity_adapter.py](file:///home/castincar/hermes-agent/agent/azure_identity_adapter.py): Authenticates and fetches active session tokens using Azure Identity SDKs for Azure-based model endpoints.
*   [background_review.py](file:///home/castincar/hermes-agent/agent/background_review.py): Executes background reviews of session history to identify memory updates or skill opportunities.
*   [bedrock_adapter.py](file:///home/castincar/hermes-agent/agent/bedrock_adapter.py): Translates schemas and requests for AWS Bedrock Converse API endpoints.
*   [browser_provider.py](file:///home/castincar/hermes-agent/agent/browser_provider.py): Implements a high-level browsing automation provider.
*   [browser_registry.py](file:///home/castincar/hermes-agent/agent/browser_registry.py): Registry managing active browser provider instances.
*   [chat_completion_helpers.py](file:///home/castincar/hermes-agent/agent/chat_completion_helpers.py): Provides formatting, token estimation, and request handlers for default OpenAI-compatible completion routes.
*   [codex_responses_adapter.py](file:///home/castincar/hermes-agent/agent/codex_responses_adapter.py): Formats request structures for the OpenAI Codex Responses API.
*   [codex_runtime.py](file:///home/castincar/hermes-agent/agent/codex_runtime.py): Manages Codex workspace directories, parameters, and environment initialization.
*   [coding_context.py](file:///home/castincar/hermes-agent/agent/coding_context.py): Scans active project trees, identifying target files and logical project boundaries.
*   [context_compressor.py](file:///home/castincar/hermes-agent/agent/context_compressor.py): Orchestrates automatic history compression, updating context snapshots and summarizing older turns when limits are approached.
*   [context_engine.py](file:///home/castincar/hermes-agent/agent/context_engine.py): Discovers and parses active files (like `.cursorrules`, `AGENTS.md`) to append to the system prompt.
*   [context_references.py](file:///home/castincar/hermes-agent/agent/context_references.py): Tracks workspace and source file paths referenced within the current conversation window.
*   [conversation_compression.py](file:///home/castincar/hermes-agent/agent/conversation_compression.py): Computes token metrics and performs basic text/message trimming.
*   [conversation_loop.py](file:///home/castincar/hermes-agent/agent/conversation_loop.py): Implements the primary turn execution cycle (`run_conversation()`), coordinating prompts, API retry structures, error failovers, and tool runs.
*   [copilot_acp_client.py](file:///home/castincar/hermes-agent/agent/copilot_acp_client.py): Standard client for communicating with the GitHub Copilot Agent Coprocessor.
*   [credential_persistence.py](file:///home/castincar/hermes-agent/agent/credential_persistence.py): Manages persistent storage, reading, and decryption of tokens and keys.
*   [credential_pool.py](file:///home/castincar/hermes-agent/agent/credential_pool.py): Maintains a registry of API keys, rotating them when encountering rate limits or validation errors.
*   [credential_sources.py](file:///home/castincar/hermes-agent/agent/credential_sources.py): Resolves configurations, environment values, and vault references to parse available API credentials.
*   [credits_tracker.py](file:///home/castincar/hermes-agent/agent/credits_tracker.py): Monitors real-time financial and token expenditures against a hard configured budget.
*   [curator.py](file:///home/castincar/hermes-agent/agent/curator.py): Evaluates user-created skills dynamically and moves them between active/stale/archived lifecycle states.
*   [curator_backup.py](file:///home/castincar/hermes-agent/agent/curator_backup.py): Handles local backups of skills prior to curation sweeps.
*   [display.py](file:///home/castincar/hermes-agent/agent/display.py): Controls interactive console animations, status feeds, and console formatting rules.
*   [error_classifier.py](file:///home/castincar/hermes-agent/agent/error_classifier.py): Classifies connection and API errors to drive fallback/backoff logic.
*   [errors.py](file:///home/castincar/hermes-agent/agent/errors.py): Defines custom exceptions shared across the core agent codebase.
*   [file_safety.py](file:///home/castincar/hermes-agent/agent/file_safety.py): Scans write targets, enforcing workspace restrictions to avoid destructive outside writes.
*   [gemini_cloudcode_adapter.py](file:///home/castincar/hermes-agent/agent/gemini_cloudcode_adapter.py): Request formatter and adapter for Google Cloud Code Gemini endpoints.
*   [gemini_native_adapter.py](file:///home/castincar/hermes-agent/agent/gemini_native_adapter.py): Adapter mapping standard formats into Google Gemini's native API models.
*   [gemini_schema.py](file:///home/castincar/hermes-agent/agent/gemini_schema.py): Maps parameters and structured outputs into Gemini schemas.
*   [google_code_assist.py](file:///home/castincar/hermes-agent/agent/google_code_assist.py): Handles integrations with Google Cloud Code Assist APIs.
*   [google_oauth.py](file:///home/castincar/hermes-agent/agent/google_oauth.py): Controls credential acquisition and lifecycle for Google Google OAuth endpoints.
*   [i18n.py](file:///home/castincar/hermes-agent/agent/i18n.py): Manages internationalization, resolving strings by locale mappings.
*   [image_gen_provider.py](file:///home/castincar/hermes-agent/agent/image_gen_provider.py): Provides logic to invoke image generation models.
*   [image_gen_registry.py](file:///home/castincar/hermes-agent/agent/image_gen_registry.py): Registry for mapping active image generation providers.
*   [image_routing.py](file:///home/castincar/hermes-agent/agent/image_routing.py): Routes image requests across active image providers.
*   [insights.py](file:///home/castincar/hermes-agent/agent/insights.py): Evaluates task trajectories to distill structured execution summaries.
*   [iteration_budget.py](file:///home/castincar/hermes-agent/agent/iteration_budget.py): Implements limits on tool-calling iteration counts.
*   [jiter_preload.py](file:///home/castincar/hermes-agent/agent/jiter_preload.py): Pre-imports performance JSON parser modules.
*   [lmstudio_reasoning.py](file:///home/castincar/hermes-agent/agent/lmstudio_reasoning.py): Sanitizes and extracts reasoning tokens from LM Studio endpoints.
*   [manual_compression_feedback.py](file:///home/castincar/hermes-agent/agent/manual_compression_feedback.py): Manages state and outputs when users trigger manual context compactions.
*   [markdown_tables.py](file:///home/castincar/hermes-agent/agent/markdown_tables.py): Formats list/dict data structures into clean Markdown tables.
*   [memory_manager.py](file:///home/castincar/hermes-agent/agent/memory_manager.py): Integrates dynamic semantic memory plugins, injecting memory summaries into prompts.
*   [memory_provider.py](file:///home/castincar/hermes-agent/agent/memory_provider.py): Abstract base class definition for memory connectors.
*   [message_sanitization.py](file:///home/castincar/hermes-agent/agent/message_sanitization.py): Validates, repairs, and sanitizes payload structures (stripping non-ASCII and surrogates).
*   [model_metadata.py](file:///home/castincar/hermes-agent/agent/model_metadata.py): Maintains context sizing tables, provider prefixes, and token count calculations.
*   [models_dev.py](file:///home/castincar/hermes-agent/agent/models_dev.py): Integrates with `models.dev` API to query details, capabilities, and token costs.
*   [moonshot_schema.py](file:///home/castincar/hermes-agent/agent/moonshot_schema.py): Adapts requests and schemas for Moonshot AI models.
*   [nous_rate_guard.py](file:///home/castincar/hermes-agent/agent/nous_rate_guard.py): Rate-limiting scheduler and queue manager for Nous API routes.
*   [onboarding.py](file:///home/castincar/hermes-agent/agent/onboarding.py): Manages profile initialization prompts for first-time users.
*   [plugin_llm.py](file:///home/castincar/hermes-agent/agent/plugin_llm.py): Handles custom user plugin loading as active model provider wrappers.
*   [portal_tags.py](file:///home/castincar/hermes-agent/agent/portal_tags.py): Parses and cleans XML portal tag blocks from tool outputs.
*   [process_bootstrap.py](file:///home/castincar/hermes-agent/agent/process_bootstrap.py): Ensures safe stdin/stdout streams for spawned processes.
*   [prompt_builder.py](file:///home/castincar/hermes-agent/agent/prompt_builder.py): Combines core system instructions, skills, active file paths, and environment settings.
*   [prompt_caching.py](file:///home/castincar/hermes-agent/agent/prompt_caching.py): Applies Anthropic cache checkpoints to system blocks.
*   [rate_limit_tracker.py](file:///home/castincar/hermes-agent/agent/rate_limit_tracker.py): Tracks API endpoint failures and cooldown periods.
*   [redact.py](file:///home/castincar/hermes-agent/agent/redact.py): Scrubs credentials and keys from text buffers and logs.
*   [retry_utils.py](file:///home/castincar/hermes-agent/agent/retry_utils.py): Provides helper utilities for backoff calculations.
*   [runtime_cwd.py](file:///home/castincar/hermes-agent/agent/runtime_cwd.py): Standardizes resolving the agent's current workspace directory.
*   [shell_hooks.py](file:///home/castincar/hermes-agent/agent/shell_hooks.py): Exposes pre-run and post-run shell hooks to track directory shifts.
*   [skill_bundles.py](file:///home/castincar/hermes-agent/agent/skill_bundles.py): Represents groupings of pre-packaged custom skills.
*   [skill_commands.py](file:///home/castincar/hermes-agent/agent/skill_commands.py): Exposes slash commands to install, search, and enable skills.
*   [skill_preprocessing.py](file:///home/castincar/hermes-agent/agent/skill_preprocessing.py): Validates environmental parameters prior to running skills.
*   [skill_utils.py](file:///home/castincar/hermes-agent/agent/skill_utils.py): Parses skill frontmatter and constructs metadata indexes.
*   [ssl_guard.py](file:///home/castincar/hermes-agent/agent/ssl_guard.py): Configures SSL contexts to support corporate proxy interceptors.
*   [stream_diag.py](file:///home/castincar/hermes-agent/agent/stream_diag.py): Logs diagnostic metrics for long-lived streaming connections.
*   [subdirectory_hints.py](file:///home/castincar/hermes-agent/agent/subdirectory_hints.py): Tracks working directory paths to guide contextual suggestions.
*   [system_prompt.py](file:///home/castincar/hermes-agent/agent/system_prompt.py): Holds static identity instructions.
*   [think_scrubber.py](file:///home/castincar/hermes-agent/agent/think_scrubber.py): Isolates and removes inline thought/reasoning tags from final logs.
*   [title_generator.py](file:///home/castincar/hermes-agent/agent/title_generator.py): Auto-generates summary titles from conversation sessions.
*   [tool_dispatch_helpers.py](file:///home/castincar/hermes-agent/agent/tool_dispatch_helpers.py): Provides helpers to parse, serialize, and format tool responses.
*   [tool_executor.py](file:///home/castincar/hermes-agent/agent/tool_executor.py): Executes API tool calls sequentially or in concurrent worker pools.
*   [tool_guardrails.py](file:///home/castincar/hermes-agent/agent/tool_guardrails.py): Restricts and sanitizes execution parameters.
*   [tool_result_classification.py](file:///home/castincar/hermes-agent/agent/tool_result_classification.py): Categorizes results into error or success flags.
*   [trajectory.py](file:///home/castincar/hermes-agent/agent/trajectory.py): Records structured trace logs of steps taken during a turn.
*   [transcription_provider.py](file:///home/castincar/hermes-agent/agent/transcription_provider.py): Adapts audio to text queries.
*   [transcription_registry.py](file:///home/castincar/hermes-agent/agent/transcription_registry.py): Registry mapping transcription engines.
*   [tts_provider.py](file:///home/castincar/hermes-agent/agent/tts_provider.py): Converts text outputs to audio streams.
*   [tts_registry.py](file:///home/castincar/hermes-agent/agent/tts_registry.py): Registry mapping text-to-speech engines.
*   [turn_context.py](file:///home/castincar/hermes-agent/agent/turn_context.py): Stores current tracking states for the active turn.
*   [turn_finalizer.py](file:///home/castincar/hermes-agent/agent/turn_finalizer.py): Flushes logs and clears state metrics when a turn completes.
*   [turn_retry_state.py](file:///home/castincar/hermes-agent/agent/turn_retry_state.py): Tracks attempt iterations for single turn retries.
*   [usage_pricing.py](file:///home/castincar/hermes-agent/agent/usage_pricing.py): Computes session costs based on input/output pricing parameters.
*   [video_gen_provider.py](file:///home/castincar/hermes-agent/agent/video_gen_provider.py): Adapter interface for video models.
*   [video_gen_registry.py](file:///home/castincar/hermes-agent/agent/video_gen_registry.py): Registry mapping video engines.
*   [web_search_provider.py](file:///home/castincar/hermes-agent/agent/web_search_provider.py): Integrates search engines into unified queries.
*   [web_search_registry.py](file:///home/castincar/hermes-agent/agent/web_search_registry.py): Registry for active web search engines.

---

## Workflow

The sequence diagram below displays the runtime flow during a single user conversation turn:

```mermaid
sequenceDiagram
    autonumber
    participant App as Caller (cli.py / gateway)
    participant Loop as conversation_loop (run_conversation)
    participant PB as prompt_builder / system_prompt
    participant Comp as context_compressor
    participant Pool as credential_pool
    participant Trans as transports / adapters
    participant Exec as tool_executor
    participant LSP as agent/lsp (Diagnostics)

    App->>Loop: run_conversation(user_message)
    activate Loop
    Loop->>PB: build_system_prompt()
    PB-->>Loop: system_prompt
    
    opt Context limit reached
        Loop->>Comp: compress_context(history)
        Comp-->>Loop: compacted_history
    end
    
    Loop->>Pool: get_active_credential()
    Pool-->>Loop: credential
    
    Loop->>Trans: chat completion / request
    Trans-->>Loop: model_response (text / tool_calls)

    alt Has tool calls
        Loop->>Exec: execute_tool_calls(tool_calls)
        activate Exec
        opt File edit tool (e.g. write_file)
            Exec->>LSP: snapshot_baseline()
            Exec->>Exec: perform edit
            Exec->>LSP: get_diagnostics_sync()
            LSP-->>Exec: delta_diagnostics
        end
        Exec-->>Loop: tool_results
        deactivate Exec
        Note over Loop, Trans: Loop back to send tool results to model
    end

    Loop->>Loop: turn_finalizer() & update costs
    Loop-->>App: final_response
    deactivate Loop
```

---

## System Architecture

The following ASCII block diagram illustrates how the components within `agent/` relate to each other and to the external system:

```
                       +----------------------------------------+
                       |              Hermes Core               |
                       |    (cli.py / gateway / run_agent.py)   |
                       +-------------------+--------------------+
                                           |
                                           v
                       +-------------------+--------------------+
                       |          agent/conversation_loop       |
                       |             (Orchestrator)             |
                       +-----+-------------+-------------+------+
                             |             |             |
        +--------------------+             |             +--------------------+
        v                                  v                                  v
+-------+--------------+           +-------+--------------+           +-------+--------------+
|   Prompt & Context   |           |    Model Transports  |           |   Tools & Execution  |
|                      |           |                      |           |                      |
|  - prompt_builder    |           |  - agent/transports  |           |  - tool_executor     |
|  - system_prompt     |           |  - provider adapters |           |  - tool_guardrails   |
|  - turn_context      |           |    (anthropic,       |           |  - file_safety       |
|  - context_          |           |     gemini, bedrock) |           |  - agent/lsp         |
|    compressor        |           |  - credential_pool   |           |                      |
|  - memory_manager    |           |  - secret_sources    |           |                      |
+----------------------+           +----------------------+           +----------------------+
```
