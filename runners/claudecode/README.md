# claudecode — IEF Runner Implementation

> **Part of [IEF-Runners](https://github.com/everwork-ai/IEF-Runners)** — the Hands layer of the Intelligent Employee Foundry.

## Overview

`claudecode` is the first runner implementation in the IEF ecosystem. It wraps the Claude Code CLI into a structured, programmable interface with provider switching, credential management, and multi-modal execution patterns.

## Three Invocation Patterns

```
──────────────────────────────────────────────────────────────────
 1. Task Mode  (one_shot / detached)
──────────────────────────────────────────────────────────────────
    start → wait → fetch

    Single task, single result. CC process runs once, writes
    durable artifacts, then exits.

──────────────────────────────────────────────────────────────────
 2. Session Chain Mode  (continue)
──────────────────────────────────────────────────────────────────
    start → wait → fetch → continue → wait → fetch → ...

    Each turn is a separate CC invocation linked via --resume.
    Crash-resilient: any completed run can be continued.

──────────────────────────────────────────────────────────────────
 3. Live Session Mode  (LongRunSession)
──────────────────────────────────────────────────────────────────
    session-start → session-send / session-capture → session-stop

    CC process stays alive. Inject follow-up prompts at any time
    via bidirectional streaming.
```

## Quick Start

```bash
# Task Mode — run a single coding task
python -m claude_worker.worker start \
  --kind coding \
  --prompt "Add error handling to all API calls" \
  --provider z-ai

# Session Chain — continue a completed run
python -m claude_worker.worker continue \
  --run-id <run-id> \
  --prompt "Now add unit tests"

# Live Session — interactive multi-turn
python -m claude_worker.worker session-start \
  --prompt "Refactor the auth module" \
  --provider z-ai
```

## Project Structure

```
code/services/api/
├── claude_worker/
│   └── worker.py              # Complete runtime
└── tests/
    └── test_claude_worker.py  # Test suite
```

## Provider Support

| Provider | Models | Thinking |
|---|---|---|
| z-ai | glm-5.1, glm-5, glm-4.7 | ✅ budget-controlled |
| qwen-bailian-coding | qwen3.6-plus, qwen3-coder-plus | ✅ default-on |
| anthropic | All Claude models | ✅ native |
| deepseek | deepseek-chat, deepseek-reasoner | ⚠️ unverified |

## Runner Interface Compliance

| Method | Status | Notes |
|---|---|---|
| `prepare(context_pack)` | ✅ | Loads config, resolves credentials |
| `execute(task_slice)` | ✅ | Delegates to `start` / `continue` / `session-start` |
| `report(status, artifacts, errors)` | ✅ | Returns `final.json`, `stdout.txt`, `exitcode.txt` |
| `resume(run_ref)` | ✅ | `--resume` flag + run-id lookup |
| `cancel(run_ref)` | ✅ | `--abort` flag for detached runs |

## IEF Integration

- Receives **Task** objects from IEF-Operations
- Produces **ArtifactRef** objects per IEF-Protocol v0
- Reads **ContextPack** from IEF-Knowledge
- Subject to **PolicyPack** from IEF-Governance

## Requirements

- Python 3.10+
- Claude Code CLI (`npm install -g @anthropic-ai/claude-code`)

## Running Tests

```bash
cd code/services/api
python -m pytest tests/test_claude_worker.py -v
```
