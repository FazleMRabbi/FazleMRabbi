# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Setup

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -e ".[dev]"
```

## Commands

```bash
# Lint and format
ruff check src/ tests/
ruff format src/ tests/
ruff format --check src/ tests/   # check-only (used in CI)

# Type checking
mypy src/

# Tests (most tests run without Windows/Power BI Desktop)
pytest -m "not e2e"              # skip tests requiring running Power BI Desktop
pytest -x -q                     # stop on first failure
pytest --cov=pbi_cli             # with coverage
pytest tests/test_commands/test_measure.py  # single test file
pytest -k "test_measure_list"    # single test by name
```

E2E tests (`-m e2e`) require a live Power BI Desktop instance on Windows and are excluded from CI.

## Architecture

pbi-cli has two independent layers that share the CLI framework but nothing else:

### Semantic Model Layer (requires live Power BI Desktop)

Data flow: `commands/` → `core/session.py` → `core/tom_backend.py` / `core/adomd_backend.py` → .NET CLR via pythonnet

- `core/dotnet_loader.py`: Lazy CLR bootstrap. Loads `pbi_cli/dlls/` (bundled Microsoft.AnalysisServices DLLs) via pythonnet/clr-loader. Nothing .NET-related imports until first connection.
- `core/session.py`: Holds the active `Session` dataclass (TOM Server + ADOMD connection). `get_session_for_command()` reconnects from the saved store in one-shot mode; REPL mode reuses the session.
- `core/connection_store.py`: Persists named connections to `~/.pbi-cli/connections.json`.
- `core/tom_backend.py`: All TOM write/read operations. Takes `.NET` TOM objects, returns plain Python dicts. This is the single .NET interop point for semantic model mutations.
- `core/adomd_backend.py`: DAX query execution via ADOMD.

### Report Layer (no connection needed)

Data flow: `commands/` → `core/report_backend.py` / `core/visual_backend.py` / etc. → JSON files on disk

- Operates entirely on PBIR (Enhanced Report Format) `.Report/definition/` JSON files.
- `core/pbir_path.py`: Auto-detects the report path by walking up from CWD or finding a sibling `.pbip` file.
- `core/report_backend.py`, `core/visual_backend.py`, `core/filter_backend.py`, etc.: Pure functions taking `Path` → returning dicts, mirroring the tom_backend pattern.
- `utils/desktop_sync.py`: After report-layer writes, optionally closes Power BI Desktop (with save), re-applies PBIR changes, and reopens. Requires `pywin32` (Windows only). Silently skipped if unavailable.

### CLI Framework

- `main.py`: Defines `PbiContext` (json_output, connection, repl_mode flags) and the root `cli` group. All commands are registered lazily in `_register_commands()`.
- `main_pbi_cli.py`: A separate entry point (`pbi-cli` binary) for skill management only; `pbi` is the main binary.
- `commands/_helpers.py`: `run_command()` wraps every backend call with error handling and automatic Desktop sync detection. Write operations (status in `_WRITE_STATUSES`) from report-layer commands trigger `desktop_sync` automatically.
- `core/output.py`: Dual-mode output. JSON mode (`--json`) prints to stdout; Rich human-readable output uses stderr for status messages to keep stdout clean for agent parsing.

### Skills System

- `src/pbi_cli/skills/`: Bundled SKILL.md files installed to `~/.claude/skills/` via `pbi-cli skills install`.
- `core/claude_integration.py`: Appends/removes the pbi-cli trigger block in `~/.claude/CLAUDE.md`.

## Adding a New Command Group

1. Create `src/pbi_cli/commands/your_cmd.py` with a `@click.group()`.
2. Use `run_command(ctx, backend_fn, **kwargs)` from `_helpers.py` — this handles errors, output formatting, and auto-sync.
3. For semantic model commands, call `get_session_for_command(ctx)` to obtain the active `Session`.
4. For report-layer commands, resolve the path with `resolve_report_path()` and pass a `definition_path` kwarg (this triggers the auto-sync check).
5. Register the group in `main.py::_register_commands()`.
6. Add tests in `tests/test_commands/test_your_cmd.py` using the `patch_session` and `tmp_connections` fixtures from `conftest.py`.

## Test Fixtures

`tests/conftest.py` provides mock TOM objects (`MockMeasure`, `MockColumn`, `MockCollection`, etc.) that mirror the .NET API surface (CamelCase properties). The `patch_session` fixture replaces the real `.NET` session with a mock, enabling non-Windows CI testing. The `tmp_connections` fixture provides an isolated `~/.pbi-cli/connections.json` via `tmp_path`.