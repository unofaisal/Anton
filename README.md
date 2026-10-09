# Anton

Anton is a lightweight multi-agent orchestration project written in Go. It loads agent definitions and skills from the filesystem, delegates complex work through a planner, and executes tool-backed tasks using an LLM provider.

## Overview

The system is built around a small coordination loop:

- a `manager` agent acts as the top-level coordinator
- sub-agents can be specialized for research, coding, writing, and analysis
- skills define reusable tool capabilities and instructions
- a `submit_plan` tool lets the manager break work into parallel or dependent tasks

This makes Anton a simple framework for building hierarchical AI agent workflows without a large runtime or heavy dependency graph.

## Why it exists

Anton is designed to explore the idea of structured, tool-using agents:

- agent definitions live in `agent-definitions/`
- skills and tool metadata live in `skills/`
- runtime orchestration code lives in `internal/`
- the main entrypoint boots the system and executes a task

## Project structure

```text
.
├── agent-definitions/
│   ├── analyst/
│   ├── coder/
│   ├── coder-backend/
│   ├── coder-frontend/
│   ├── manager/
│   ├── researcher/
│   └── writer/
├── cmd/
│   └── pipeline/
│       └── main.go
├── internal/
│   ├── agent/
│   ├── llm/
│   ├── memory/
│   ├── registry/
│   ├── runner/
│   └── tool/
├── skills/
│   └── delegation/
│       ├── SKILL.md
│       └── scripts/
├── .gitignore
├── go.mod
├── go.sum
├── hello.txt
├── mlp.py
└── README.md
```

## Architecture

### Agent definitions

Each directory under `agent-definitions/` contains an `AGENT.md` file. These define:

- the role of the agent
- the skills it uses
- the allowed sub-agents it can delegate to
- the instructions it follows

### Skills

A skill directory contains:

- `SKILL.md` with the skill's rules and guidance
- a `scripts/` folder with tool schemas and executable scripts

When the registry loads the project, it scans these directories and connects tool metadata to agent capabilities.

### Runtime flow

The runtime flow is roughly:

1. Load agents and skills from disk
2. Build a LLM client connected to OpenRouter
3. Start the `manager` agent
4. If a task is complex, the manager can call `submit_plan`
5. Sub-agents run with their allowed tools and instructions
6. Final results are aggregated back to the user

## Requirements

- Go 1.24.3 or newer
- An API key from OpenRouter or another compatible provider
- A working internet connection for model access

## Setup

1. Clone the repository:

```bash
git clone https://github.com/unofaisal/Anton.git
cd Anton
```

2. Install dependencies:

```bash
go mod tidy
```

3. Set your LLM API key:

```bash
export OPENROUTER_API_KEY="your_key_here"
```

4. Run the example app:

```bash
go run ./cmd/pipeline
```

## Usage

The current main entrypoint starts with the `manager` agent and asks it to inspect the current directory:

```go
brain := agent.NewBrain(client, "openai/gpt-oss-120b:free", agents, 0)
res, err := brain.Execute(context.Background(), "manager", "Check for the files in the current dir and summarize them.")
```

You can customize the prompt or swap in a different model by editing `cmd/pipeline/main.go`.

## Environment variables

The project currently expects the following environment variable:

- `OPENROUTER_API_KEY` — required for LLM requests

The file also contains a commented-out alternative for Hugging Face, which indicates the project was designed to be flexible across model providers.

## Notes

- `.env` is ignored in the repository configuration, but tracked files are still not automatically removed from Git history.
- `hello.txt` appears to be a placeholder or scratch file and may be removed later as the project evolves.
- The `mlp.py` file exists in the repo root and may be unrelated to the Go runtime; it should be reviewed if it is not meant to be a project artifact.

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

## Contributing

Contributions are welcome. The repo is currently structured as an experimentation/prototype for agent orchestration, so improvements in usability, documentation, and runtime stability are especially valuable.

## Status

Anton is a working prototype for hierarchical multi-agent task execution. It is best understood as an experimental orchestration framework rather than a production application framework yet.

---

If you'd like to extend Anton, a good next step would be to add:

- stronger validation for tool args
- clearer task result aggregation
- a command-line interface for custom prompts
- a richer set of built-in tools and skills
- CI checks for Go build/test validation

