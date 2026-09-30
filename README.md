# spark-vllm

Components for serving models with [vLLM](https://github.com/vllm-project/vllm) on an NVIDIA
DGX Spark (GB10 Grace Blackwell, aarch64, 128 GB unified memory).

## Status

Initial setup. The repo currently holds only the development lifecycle; the serving components
haven't been added yet. Each will land as its own change through that lifecycle.

## Development

Changes follow the agent lifecycle from
[agent-lifecycle-template](https://github.com/kwcantrell/agent-lifecycle-template):
spec-driven, with review depth set by each change's risk tier and gates enforced by hooks and CI.

- Agents: [AGENTS.md](AGENTS.md)
- Humans: [docs/lifecycle.md](docs/lifecycle.md) (how it works and setup),
  [docs/security.md](docs/security.md) (threat model), [docs/decisions/](docs/decisions/) (why)
