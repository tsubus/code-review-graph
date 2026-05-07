## Why

Oh My Pi (OMP) is a popular AI coding agent for the terminal (4.1k+ stars) with first-class support for **hooks** (event-interceptor TypeScript modules). Code-review-graph already supports Claude Code, Copilot, Cursor, Gemini CLI, Qoder, OpenCode, and others — but not OMP. Adding OMP support lets users benefit from graph-powered code review, exploration, and refactoring without manual configuration. Unlike static skills which are hit-or-miss, hooks transparently inject graph behavior by intercepting session and tool events.

## What Changes

- Add `omp` as a supported platform in `code_review_graph/skills.py`:
  - MCP server configuration entry in `PLATFORMS` (`.omp/mcp.json`)
  - Hook generator `install_omp_hooks()` that writes `.omp/hooks/code-review-graph.ts` — a TypeScript hook module that shows graph status on session start and auto-updates the graph on file edits
- Update `code_review_graph/cli.py`:
  - Add `omp` to `_PLATFORM_CHOICES`
  - Wire `install_omp_hooks()` into `_handle_init()` under `--platform omp` / `--platform all`
- Add tests covering OMP hook generation and MCP config installation
- Update documentation (`docs/USAGE.md`, `README.md`) to list OMP as a supported platform

## Capabilities

### New Capabilities
- `omp-platform-support`: MCP config installation and hook generation for Oh My Pi

### Modified Capabilities
- *(none — no existing spec-level behavior changes)*

## Impact

- `code_review_graph/skills.py`: new platform entry + hook installer function
- `code_review_graph/cli.py`: platform choice list + install handler branch
- `tests/test_skills.py`: new test cases for OMP hook generation
- `docs/USAGE.md` + `README.md`: platform list update
