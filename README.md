# distributed-agent-runtime

> Runtime for coordinating multiple autonomous LLM agents across distributed tasks. Battle-tested on real bug bounty pipelines (OpenClaw project) with findings up to Critical / $10k.

## Architecture

```
┌────────────────────────────────────────────────────────────┐
│                    Task Dispatcher                          │
│         (receives goal → decomposes → assigns)             │
├──────────────┬─────────────────┬────────────────────────── ┤
│  Strategic   │  Tactical       │  Operational              │
│  Agent       │  Agent          │  Agents (N)               │
│  (80B model) │  (14B model)    │  (7B models)              │
│  Planning    │  Coordination   │  Execution                │
├──────────────┴─────────────────┴───────────────────────────┤
│                   Tool Registry                             │
│  web_search │ code_exec │ file_ops │ api_calls │ custom    │
├────────────────────────────────────────────────────────────┤
│                   Memory Layer                             │
│   Short-term (context) │ Long-term (vector DB) │ Shared   │
└────────────────────────────────────────────────────────────┘
```

## Agent Roles

| Role | Model | Responsibility |
|------|-------|----------------|
| Strategist | 80B (Qwen2.5/DeepSeek) | Goal decomposition, quality control |
| Coordinator | 14B (DeepSeek-R1) | Task assignment, progress tracking |
| Executor | 7B (Mistral) | Tool calls, data collection |
| Critic | 14B | Output validation, retry logic |

## ACP Protocol

Custom **Agent Communication Protocol** for structured inter-agent messaging:

```json
{
  "protocol": "acp/1.0",
  "from": "strategist",
  "to": "executor_01",
  "type": "TASK_ASSIGN",
  "task_id": "recon_001",
  "payload": {
    "objective": "...",
    "tools_allowed": ["web_search", "api_call"],
    "timeout": 300,
    "quality_threshold": 0.85
  }
}
```

## Quick Start

```python
from distributed_agent_runtime import AgentRuntime, Task

runtime = AgentRuntime(
    strategist_model="qwen2.5:72b",
    coordinator_model="deepseek-r1:14b",
    executor_model="mistral:7b",
    max_parallel_agents=8,
)

task = Task(
    goal="Research target and generate comprehensive report",
    tools=["web_search", "code_exec"],
    max_iterations=50,
)

result = runtime.run(task)
```

## OpenClaw Integration

This runtime powers [OpenClaw](https://github.com/Dev-next-gen) — an autonomous bug bounty pipeline:
- 80B model handles reconnaissance strategy
- 14B model coordinates scanning phases
- Multiple 7B agents execute parallel scans
- Findings validated up to Critical severity / $10k

## Directory Structure

```
distributed-agent-runtime/
├── runtime/
│   ├── dispatcher.py
│   ├── agents/
│   │   ├── strategist.py
│   │   ├── coordinator.py
│   │   └── executor.py
│   ├── protocols/
│   │   └── acp.py
│   └── memory/
│       ├── short_term.py
│       └── long_term.py
├── tools/
├── tests/
└── examples/
```

## License

MIT — Léo Camus / [NextGen Labs](https://nextgen-labs.net)
