# Repository Guidelines

## Project Structure & Module Organization
- Core Python sources live under `src/kimi_cli` (CLI entrypoint, UI, tools, agents, prompts, utils, wire-up). Shared bits for packaging live in `src/kaos`.
- Tests: unit/integration specs in `tests/`; AI scenarios and scripts in `tests_ai/` (heavier, may hit live services). Docs and images live in `docs/`. Build outputs go to `build/` and `dist/` (keep clean in commits).

## Build, Test, and Development Commands
- `make prepare` — sync dependencies via `uv` using the locked versions in `uv.lock`.
- `uv run kimi` — launch the CLI locally from the source tree.
- `make format` — auto-fix style with Ruff, then format.
- `make check` — Ruff lint (no fixes), format check, and Pyright type check.
- `make test` — run the pytest suite in `tests/`.
- `make ai-test` — run `tests_ai` flows; expect longer runtimes and possible external calls.
- `make build` — package the standalone binary with PyInstaller.

## Coding Style & Naming Conventions
- Python 3.13; default line length 100. Prefer typed functions and early returns.
- Use Ruff format and lint rules (`ruff check`; keep imports ordered). Run `make format` before sending PRs.
- Naming: modules and functions `snake_case`, classes `PascalCase`, constants `UPPER_SNAKE`. Keep prompt/config files under `src/kimi_cli/prompts` and tooling under `src/kimi_cli/tools`.

## Testing Guidelines
- Framework: pytest (+pytest-asyncio). Place new specs alongside the feature in `tests/` with descriptive names like `test_cli_shell_mode.py`.
- Keep tests deterministic; avoid network calls unless in `tests_ai` and guard them with markers or fixtures.
- When changing prompts or agent wiring, add scenario coverage or update snapshots where applicable.

## Commit & Pull Request Guidelines
- Commit messages: short, present-tense summaries similar to existing history.
- Pull requests: include a concise description of behavior changes, test evidence (`make test` / `make ai-test` notes), and any docs updates. Link related issues. For user-facing output changes, add before/after screenshots or terminal transcripts when feasible.
