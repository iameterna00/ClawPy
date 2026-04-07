# ClawPy

A lightweight Python implementation of a production-grade LLM agent loop, based on the architecture of [claw-code](https://github.com/ultraworkers/claw-code)'s `query.ts`.

## What it does

ClawPy implements a `while True` async agent loop that:

- Calls the xAI Grok API (OpenAI-compatible)
- Executes tools when the model requests them
- Feeds tool results back into context and loops
- Compresses context automatically when it gets too long
- Tracks token budget across turns
- Checks stop hooks to know when to exit

## Tools

- `web_search` — Real-time web search via Tavily API
- `execute_python` — Run Python code locally via subprocess
- `execute_local` — Run any local .exe or shell command

## Architecture

Mirrors the core design of `query.ts`:

| query.ts | ClawPy |
|---|---|
| `queryLoop()` async generator | `agent_loop()` async generator |
| `autocompact` | `compress_context()` |
| `createBudgetTracker` | `TokenBudget` class |
| `handleStopHooks` | `check_stop_hooks()` |
| `runTools` | `run_tool()` |
| `maxTurns` guard | `max_turns` parameter |

## Setup

```bash
pip install openai httpx
```

```bash
export GROK_API_KEY=your_grok_api_key
export TAVILY_API_KEY=your_tavily_api_key
```

```bash
python agent.py
```

## Usage

```python
from agent import agent_loop

async for event in agent_loop("Search for the latest AI news and summarize it"):
    if event["type"] == "text":
        print(event["content"])
    elif event["type"] == "tool_call":
        print(f"Calling tool: {event['name']}")
    elif event["type"] == "done":
        print(f"Done: {event['reason']}")
```

## Event Types

| Event | Description |
|---|---|
| `text` | Assistant response text |
| `tool_call` | Tool being called with args |
| `tool_result` | Result returned from tool |
| `done` | Loop finished with reason |
| `error` | Something went wrong |

## Stack

- xAI Grok API (OpenAI-compatible)
- Tavily Search API
- Python asyncio + subprocess
