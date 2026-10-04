> **Archived.** This repo moved to [RLASAF12/agent-failure-lab](https://github.com/RLASAF12/agent-failure-lab/tree/main/gridlock) (folder `gridlock/`, full history preserved). Archived 2026-10-04.

# GRIDLOCK — Agent Failure Series #18

![Series](https://img.shields.io/badge/Agent%20Failure%20Series-%2318-f97316?style=flat-square)
![Type](https://img.shields.io/badge/type-interactive%20demo-a78bfa?style=flat-square)
![Status](https://img.shields.io/badge/live-harelasaf.com-22c55e?style=flat-square)

> **What happens when your multi-agent pipeline deadlocks on circular dependencies — and the rescue agent joins the deadlock?**

## What It Is

An interactive single-file HTML simulator demonstrating the GRIDLOCK failure mode in multi-agent AI systems. Watch in real time as circular dependencies freeze a pipeline, recovery attempts make things worse, timeouts burn tokens with $0 work done, and false completion signals fire anyway.

4 live panels. Every failure mode runs as an animated, interactive demo.

## Why It Exists

Multi-agent orchestration frameworks (LangGraph, CrewAI, AutoGen) have no built-in cycle detection. When Agent A waits for Agent B and Agent B waits for Agent A, the system silently freezes. Spawning a coordinator to resolve the deadlock typically gets captured by the same dependency graph. The timeout handler eventually fires, marks the pipeline `status="complete"`, and the downstream publisher receives empty arrays.

5 confirmed production incidents documented (2026). This pattern is underreported because logs look normal — both agents show "WAITING" status, not "FAILED".

## Whats Inside

```
index.html          — self-contained simulator (no dependencies, no build step)
```

Open it in a browser. No install, no server, no API keys.

## The 4 Failure Modes

| Panel | Name | What It Shows |
|-------|------|---------------|
| 1 | Chicken-Egg Lock | Basic A↔B deadlock with normal-looking logs |
| 2 | Rescue Trap | Coordinator agent captured by the same dependency graph |
| 3 | Timeout Token Burn | Live cost counter — tokens consumed, work done: 0% |
| 4 | False Completion | Pipeline marked complete, blank page published, success notification sent |

## The Fix

```python
from graphlib import TopologicalSorter, CycleError

def validate_pipeline(agents: dict[str, list[str]]) -> None:
    graph = {agent: set(deps) for agent, deps in agents.items()}
    try:
        list(TopologicalSorter(graph).static_order())
    except CycleError as e:
        raise ValueError(f"Pipeline has circular dependency: {e}") from e
```

Run topological sort before the first API call. O(V+E). Catches cycles before any tokens burn.

## Live Demo

**[→ rlasaf12.github.io/gridlock/](https://rlasaf12.github.io/gridlock/)**

## Series

| # | Name | Failure Mode |
|---|------|-------------|
| 15 | RaceCondition | Concurrent writes corrupt shared state |
| 16 | StaleMind | Agent acts on cached context after state change |
| 17 | GhostApproval | HITL gate bypassed by timeout handler |
| **18** | **GRIDLOCK** | **Circular dependency deadlock + rescue trap** |

Built by [harelasaf.com](https://harelasaf.com) — AI Operator documenting what actually breaks in production.

