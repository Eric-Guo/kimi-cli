# CODEBUDDY.md This file provides guidance to CodeBuddy Code when working with code in this repository.

## Development commands

- Prereqs: Python 3.13+, uv installed (pyproject requires >=3.13).
- Setup env: `make prepare`
- Run CLI (dev):
  - `uv run kimi`
  - `uv run kimi --help`
- Code quality:
  - Format: `make format`
  - Lint + types: `make check`
- Tests:
  - All: `make test`
  - AI tests: `make ai-test`
  - Single file: `pytest tests/test_specific.py`
- Build binary: `make build`
- Discover targets: `make help`

## Key CLI options (commonly used)
- `--agent-file PATH`: Use a custom agent spec.
- `--model, -m NAME`: Select model (overrides config default).
- `--yolo` (aka `--yes`, `-y`, `--auto-approve`): Auto-approve actions.
- `--acp`: Run as Agent Client Protocol server.
- `--wire`: Run Wire server (experimental).
- `--print`: Non-interactive print mode (implies `--yolo`).
- `--mcp-config-file PATH` (repeatable) or `--mcp-config JSON`: Load MCP tools.

## Big-picture architecture

Request flow (high level):
- CLI entrypoint parses flags and selects UI mode in src/kimi_cli/cli.py:36.
- Orchestrator builds runtime and agent in src/kimi_cli/app.py:25 (KimiCLI.create).
- Runtime collects builtin args (cwd listing, AGENTS.md, timestamp) in src/kimi_cli/soul/runtime.py:72.
- Agent spec (YAML) is loaded and toolset assembled in src/kimi_cli/soul/agent.py:28, system prompt templated in src/kimi_cli/soul/agent.py:79 using src/kimi_cli/agents/default/system.md.
- Core loop runs in the "soul" (LLM + tools controller) in src/kimi_cli/soul/kimisoul.py:55, step loop at src/kimi_cli/soul/kimisoul.py:158.
- UI adapters run the session: shell/print/acp/wire in src/kimi_cli/app.py:136, 179, 196, 203.

Key components
- CLI (Typer): Defines `kimi` command, validates exclusive UI flags, session lifecycle, JSON parsing for MCP (src/kimi_cli/cli.py:36-311).
- KimiCLI orchestrator: Creates LLM, Runtime, loads Agent, restores Context (src/kimi_cli/app.py:25-102).
- Runtime: Bundles config, LLM, session, builtin prompt args, approval and DenwaRenji (src/kimi_cli/soul/runtime.py:62-96). Loads AGENTS.md from repo root if present (src/kimi_cli/soul/runtime.py:29-39).
- AgentSpec + Agent loader:
  - Spec resolver and defaults in src/kimi_cli/agentspec.py:51-75 (default file at src/kimi_cli/agentspec.py:14).
  - Loads/templating system prompt and tools; optional MCP tools (src/kimi_cli/soul/agent.py:28-76, 140-159).
- Soul (KimiSoul):
  - Maintains context, retries transient API errors, compacts when near token limit, integrates tool results (src/kimi_cli/soul/kimisoul.py:146-279, 291-337).
  - Thinking support gated by model capabilities (src/kimi_cli/soul/kimisoul.py:121-138).
- Session & Context:
  - Sessions stored under ~/.kimi (hash per work dir) (src/kimi_cli/metadata.py:25-28; src/kimi_cli/session.py:9-53, 55-80, 82-104).
  - Context persisted to a jsonl history file with checkpoints and token counts (src/kimi_cli/soul/context.py:14-61, 74-143).
- Config & LLM:
  - Config file at ~/.kimi/config.json; created on first load (src/kimi_cli/config.py:95-107, 110-156).
  - Env var overrides for KIMI_* and OPENAI_* (src/kimi_cli/llm.py:30-68).
  - LLM provider abstraction via kosong; thinking auto-enabled for certain models (src/kimi_cli/llm.py:71-139, 142-150).
- UI modes:
  - Shell, Print, ACP, WireOverStdio invoked by KimiCLI (src/kimi_cli/app.py:136-208).

Tool system and safety model
- File read:
  - Requires absolute path; size-limited (MAX_LINES=1000, MAX_BYTES=100KB) (src/kimi_cli/tools/file/read.py:61-71, 86-135).
- File write/edit:
  - Absolute paths only; must be within working directory; requires approval (src/kimi_cli/tools/file/write.py:37-52, 92-101; src/kimi_cli/tools/file/replace.py:40-55, 95-101).
- Glob search:
  - Validates pattern; rejects leading `**`; confines search to working dir (src/kimi_cli/tools/file/glob.py:45-61, 63-77, 79-143).
- Grep:
  - Uses ripgrep; auto-installs a bundled binary if needed (src/kimi_cli/tools/file/grep.py:111-135, 161-235, 237-303).
- Bash/CMD:
  - Streams output; requires approval; timeout enforced (src/kimi_cli/tools/bash/__init__.py:46-77, 80-114).
- Approval/yolo:
  - All mutating operations and shell commands flow through Approval; `--yolo` auto-approves (wired by Runtime in src/kimi_cli/soul/runtime.py:94-96).

Security considerations (important for agents)
- Not sandboxed; operations must stay in the working directory. Tools explicitly guard against path traversal and out-of-repo writes (see file write/edit/glob references above).
- Many tools require absolute paths; prefer using provided tool interfaces rather than ad-hoc shell to avoid bypassing checks.
- Print mode implies `--yolo`; avoid using it unless non-interactive automation is intended (src/kimi_cli/cli.py:281-299).

Testing and quality gates
- Tests live under `tests/` (pytest) and optional AI tests under `tests_ai/` (Makefile `ai-test`).
- Linting/formatting via ruff; type checking via pyright (`make check`).

Configuration summary
- Default config path: `~/.kimi/config.json` (auto-created). Models/providers resolved from config or `--model`; environment variables can override at runtime (KIMI_BASE_URL, KIMI_API_KEY, KIMI_MODEL_NAME, etc.) (src/kimi_cli/config.py:95-156; src/kimi_cli/llm.py:30-68).

Notes for future agents
- Prefer tools over raw shell for file/search operations to inherit safety, pagination, and formatting.
- If AGENTS.md exists at repo root, Runtime injects its contents into the system prompt automatically (src/kimi_cli/soul/runtime.py:29-39).
