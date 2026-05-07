## Why

The Oh My Pi (OMP) platform support added in the last commit uses project-level paths (`.omp/mcp.json` and `.omp/hooks/`) that mirror other platforms like Claude Code and Cursor. However, OMP has a peculiar structure where its global user-level config lives under a nested `agent/` directory: `~/.omp/agent/mcp.json` for MCP servers and `~/.omp/agent/hooks/` for hooks. The project-level structure is different — `.omp/mcp.json` is flat while hooks still belong under `.omp/agent/hooks/`. The current implementation places hooks at `.omp/hooks/`, which OMP does not load. Additionally, `detect` returns `True` unconditionally instead of checking whether OMP is actually installed.

Because `code-review-graph` is a user-level tool (its MCP server can serve any repo via `cwd`), it should integrate with OMP at the global level like Windsurf, Zed, and Continue — not pollute every project with `.omp/` directories.

## What Changes

- Switch OMP MCP config from project-level `.omp/mcp.json` to global `~/.omp/agent/mcp.json`
- Switch OMP hooks from project-level `.omp/hooks/` to global `~/.omp/agent/hooks/`
- Fix `PLATFORMS["omp"]["detect"]` from `lambda: True` to `lambda: (Path.home() / ".omp").exists()`
- Update `install_omp_hooks()` to write to `~/.omp/agent/hooks/` (global, ignoring `repo_root`)
- Add JSONC comment/trailing-comma stripping for OMP config reads (`.omp/agent/mcp.json` may contain comments)
- Update all tests to assert global paths and detect behavior
- Update documentation (README, USAGE) to show correct global config location

## Capabilities

### New Capabilities
- `omp-global-config`: Correctly integrate with Oh My Pi's global user-level configuration under `~/.omp/agent/`

### Modified Capabilities
- (none — this is a pure bugfix with no spec-level behavior changes)

## Impact

- `code_review_graph/skills.py`: `PLATFORMS["omp"]` config path, detect lambda, `install_omp_hooks()` path
- `code_review_graph/cli.py`: No changes (wiring already exists)
- `tests/test_skills.py`: All OMP path assertions and detect tests
- `README.md`, `docs/USAGE.md`: OMP config file path in supported platforms table
