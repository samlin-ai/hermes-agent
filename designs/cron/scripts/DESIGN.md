# cron/scripts Design Documentation

## Goal
The `cron/scripts` directory contains executable helper scripts designed to run as part of the cron subsystem or as standalone command-line components. The primary goal of this directory is to house proactive background monitoring, ingestion, and filtering logic (such as the proactive urgency-monitor pattern). These scripts fetch or receive candidate items, evaluate them against configured criteria using a cheap/fast auxiliary LLM, and filter out below-threshold noise so that only actionable notifications or summaries are surfaced to the user.

## File Enumeration
- [__init__.py](../../../cron/scripts/__init__.py): Package initializer for `cron.scripts`. Scripts here are runnable via `python3 -m cron.scripts.<name>`.
- [classify_items.py](../../../cron/scripts/classify_items.py): CLI utility implementing the urgency-monitor pattern. Reads candidate items as JSON (a list, or `{"items": [...]}`, or a single object) from stdin or `--input-file`. It builds one batch prompt (showing each item's salient fields — title/subject/summary/text/body/from/sender/url) and makes a single `call_llm(task="monitor", temperature=0, max_tokens=1024)` request. The classifier returns a JSON array of `{index, score (0-10), reason}` objects (one per item); the script parses them locally (tolerating markdown fences) and emits only items scoring at or above `--threshold` (default 7), in `--format` `text` (default, Markdown blocks) or `json`. Empty input or no item above threshold → empty stdout, exit 0 (silent, so a wrapping cron job stays quiet). Failures are loud, not silent: invalid input JSON → exit 2, auxiliary-client import failure → exit 3, classifier call failure → exit 4.

## Workflow
```mermaid
sequenceDiagram
    participant Caller as Cron/CLI Caller
    participant CI as classify_items.py
    participant AC as agent.auxiliary_client

    Caller->>CI: Invokes script with JSON items, --threshold, & --criteria
    Note over CI: Load and parse items from stdin or file
    alt Items is empty
        CI-->>Caller: Silent exit 0 (prints nothing)
    else Items exist
        CI->>AC: call_llm(task="monitor", messages, temperature=0)
        AC-->>CI: Returns chat completion response
        Note over CI: Parse score and reasons from JSON response
        Note over CI: Filter items where score >= threshold
        alt Filtered items is empty
            CI-->>Caller: Silent exit 0 (prints nothing)
        else Filtered items exist
            Note over CI: Format outputs (text or JSON)
            CI-->>Caller: Print items to stdout (exit 0)
        end
    end
```

## System Architecture
```
               ┌─────────────────────────────────┐
               │    Cron Job / CLI Invocation    │
               └────────────────┬────────────────┘
                                │
                                │ reads candidates
                                │ via stdin/file
                                ▼
 ┌─────────────────────────────────────────────────────────────┐
 │ cron/scripts                                                │
 │                                                             │
 │   ┌─────────────────────────────────────────────────────┐   │
 │   │                  classify_items.py                  │   │
 │   │                                                     │   │
 │   │  - Parses JSON input                                │   │
 │   │  - Scores candidates via auxiliary client           │   │
 │   │  - Outputs items above target threshold             │   │
 │   └──────────────────────────┬──────────────────────────┘   │
 └──────────────────────────────┼──────────────────────────────┘
                                │
                                │ calls call_llm(task="monitor")
                                ▼
 ┌─────────────────────────────────────────────────────────────┐
 │ agent/auxiliary_client.py                                   │
 │                                                             │
 │  - Communicates with configured monitor LLM provider        │
 └─────────────────────────────────────────────────────────────┘
```
