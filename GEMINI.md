# Gemini Code Assistant Context

This document provides context for the Gemini Code Assistant to understand the Kimi CLI project.

## Project Overview

Kimi CLI is a command-line agent that assists with software development tasks. It's a Python project that can be installed using `uv`. It can be used as a standalone CLI, integrated with Zsh, or with any ACP-compatible editor like Zed.

The project is structured as follows:

- `src/kimi_cli`: The main source code for the application.
- `tests`: The test suite for the project.
- `pyproject.toml`: The project's metadata and dependencies.
- `Makefile`: Contains common development commands.

### Architecture

The application's entry point is `src/kimi_cli/cli.py`, which uses the `typer` library to create the command-line interface. The core application logic is in `src/kimi_cli/app.py`, which initializes and manages the various components of the application.

The "soul" of the agent is the `KimiSoul` class in `src/kimi_cli/soul/kimisoul.py`. This class is responsible for managing the conversation context, interacting with the LLM, and executing tools.

Agents are defined in YAML files and loaded by the `load_agent` function in `src/kimi_cli/soul/agent.py`. Tools are dynamically loaded from Python modules, allowing for a flexible and extensible tool system.

## Building and Running

The following commands are used for building, running, and testing the project.

- **Prepare the development environment:**
  ```sh
  make prepare
  ```

- **Run Kimi CLI:**
  ```sh
  uv run kimi
  ```

- **Format code:**
  ```sh
  make format
  ```

- **Run linting and type checking:**
  ```sh
  make check
  ```

- **Run tests:**
  ```sh
  make test
  ```

- **Build the standalone executable:**
  ```sh
  make build
  ```

## Development Conventions

- **Coding Style:** The project uses `ruff` for linting and formatting. The configuration is in `pyproject.toml`.
- **Testing:** The project uses `pytest` for testing. Tests are located in the `tests` directory.
- **Dependencies:** The project uses `uv` for dependency management. Dependencies are listed in `pyproject.toml`.
