# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kimi CLI is an AI-powered command-line interface agent for software development tasks. It functions as both an interactive coding assistant and a shell, with support for multiple UI modes (shell, print, ACP, wire) and integration capabilities with editors like Zed.

## Development Commands

### Essential Commands
```bash
# Development setup
make prepare                   # Prepare development environment with frozen dependencies

# Running Kimi CLI
uv run kimi                    # Run Kimi CLI in development mode
uv run kimi --help             # Show all CLI options

# Code Quality
make format                    # Auto-format code with ruff
make check                     # Run linting (ruff) and type checking (pyright)

# Testing
make test                      # Run unit tests with pytest
make ai-test                   # Run AI-powered tests in tests_ai/
pytest tests/test_specific.py  # Run specific test file

# Building
make build                     # Build standalone executable with PyInstaller
```

### Key CLI Options
- `--agent-file`: Custom agent specification file
- `--model` or `-m`: Specify LLM model
- `--yolo`: Approve all actions without confirmation
- `--acp`: Enable Agent Client Protocol mode
- `--mcp-config-file`: Connect to MCP servers

## Architecture Overview

### Core Structure
The codebase follows a modular architecture with clear separation of concerns:

```
src/kimi_cli/
├── cli.py              # Click-based CLI entry point, handles command parsing
├── app.py              # Main KimiCLI class, orchestrates the application
├── soul/               # Core AI agent implementation
│   ├── kimisoul.py     # Main agent logic and LLM interaction
│   ├── agent.py        # Agent loading and management
│   ├── context.py      # Context management for conversations
│   └── runtime.py      # Runtime execution environment
├── tools/              # Tool system for agent capabilities
│   ├── bash.py         # Shell command execution with safety controls
│   ├── file.py         # File operations (read, write, patch)
│   ├── task.py         # Todo list and task management
│   └── mcp.py          # Model Context Protocol integration
├── agents/             # Agent configurations
│   └── default/        # Default agent with coding capabilities
└── session.py          # Session management and persistence
```

### Key Design Patterns

1. **Agent-Tool Architecture**: The agent (`KimiSoul`) uses a tool system where each capability is a separate tool class with standardized interfaces.

2. **Multi-Modal UI**: Supports different UI modes (shell, print, ACP, wire) through a unified interface, allowing integration with various environments.

3. **Session-Based**: Conversations are managed through sessions that can be persisted and resumed.

4. **Safety-First**: All potentially dangerous operations (file writes, shell commands) require user approval unless `--yolo` is used.

### Important Implementation Details

- **Python 3.13+ Required**: Uses modern Python features and type hints extensively
- **Async/Await**: Core components use asyncio for concurrent operations
- **Pydantic Models**: Heavy use of Pydantic for data validation and configuration
- **Rich Terminal Output**: Uses rich library for formatted terminal output
- **LLM Abstraction**: Uses kosong library for LLM provider abstraction

### Configuration System

- **Main Config**: Loaded from `~/.config/kimi-cli/config.yaml` or specified via `--config-file`
- **Agent Config**: YAML-based agent specifications in `src/kimi_cli/agents/`
- **Environment Variables**: LLM provider credentials via env vars (KOSONG_API_KEY, etc.)

### Testing Strategy

- **Unit Tests**: Comprehensive test suite in `tests/` using pytest
- **AI Tests**: AI-powered testing in `tests_ai/` for complex scenarios
- **Integration Tests**: Test tool integrations and agent behaviors
- **Type Checking**: Pyright for static type analysis
- **Linting**: Ruff for code formatting and linting

### Security Considerations

- **Working Directory Restrictions**: File operations limited to current working directory
- **Approval System**: Dangerous operations require explicit user approval
- **Command Timeouts**: Shell commands have execution timeouts
- **Non-sandboxed**: Direct system access with appropriate warnings to users
