# gateway/builtin_hooks Design Documentation

## Goal
The `gateway/builtin_hooks` directory is designed as an extension point for always-active, built-in gateway hooks in the Hermes messaging gateway. It is intended to house event handlers that respond to key lifecycle events in the gateway (e.g., startup, session lifecycle, agent steps, command executions) and are registered automatically without requiring discovery from the user's local directory.

Currently, this directory contains no active hook implementations and serves as a placeholder stub for future core gateway hooks.

## File Enumeration
* [__init__.py](file:///home/castincar/hermes-agent/gateway/builtin_hooks/__init__.py): Initializes the Python package. Contains package-level documentation indicating its role as the entry point for always-registered built-in hooks.

## Workflow
Built-in hooks are loaded during the initialization phase of the `HookRegistry` within `gateway/hooks.py`. The runtime registration and execution sequence is as follows:

```mermaid
sequenceDiagram
    participant GR as GatewayRunner
    participant HR as HookRegistry (gateway/hooks.py)
    participant BH as Built-in Hooks (gateway/builtin_hooks)
    participant UH as User Hooks (~/.hermes/hooks/)

    GR->>HR: discover_and_load()
    activate HR
    HR->>HR: _register_builtin_hooks()
    activate HR
    Note over HR, BH: Future built-in hooks will be imported<br/>and registered into HR._handlers
    deactivate HR
    
    HR->>UH: Scan and load HOOK.yaml / handler.py
    Note over HR, UH: User hooks are loaded dynamically
    
    HR-->>GR: Loading complete
    deactivate HR

    Note over GR, HR: Firing an event (e.g., "agent:start")
    GR->>HR: emit(event_type, context)
    activate HR
    HR->>BH: Fire built-in handlers (if registered)
    HR->>UH: Fire user-defined handlers
    HR-->>GR: Dispatch done
    deactivate HR
```

## System Architecture
The relationship between built-in hooks, the hook registry, and the rest of the gateway is outlined in the block diagram below:

```text
┌────────────────────────────────────────────────────────────────┐
│                       Hermes Gateway Core                      │
│                                                                │
│   ┌────────────────┐                                           │
│   │ GatewayRunner  ├───────────────────────────────┐           │
│   └───────┬────────┘                               │           │
│           │ Fires Events                           │           │
│           ▼                                        ▼           │
│   ┌────────────────┐                       ┌──────────────┐    │
│   │  HookRegistry  │                       │   AIAgent    │    │
│   │(gateway/hooks) │                       │(run_agent.py)│    │
│   └───────┬────────┘                       └──────────────┘    │
│           │                                                    │
│           ├──────────────────────────────┐                     │
│           │ Loads always-active          │ Loads dynamically   │
│           ▼                              ▼                     │
│   ┌──────────────────────┐       ┌───────────────┐             │
│   │ gateway/builtin_hooks│       │ ~/.hermes/    │             │
│   │  (Extension Point)   │       │    hooks/     │             │
│   └──────────────────────┘       └───────────────┘             │
└────────────────────────────────────────────────────────────────┘
```
