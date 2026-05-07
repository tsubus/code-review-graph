## Context

Oh My Pi (OMP) stores its user-level configuration in `~/.omp/` with a nested `agent/` subdirectory:

```
~/.omp/
├── agent/
│   ├── config.json          # Main agent settings
│   ├── mcp.json             # MCP server configuration
│   ├── models.yml           # Custom model definitions
│   ├── commands/            # Custom slash commands
│   ├── agents/              # Custom task agents
│   ├── extensions/          # Custom extensions
│   └── hooks/               # Custom hooks
├── plugins/                 # Installed MCP plugins
├── sessions/                # Saved conversation sessions
└── ...
```

The last commit implemented OMP support using project-level paths:
- `.omp/mcp.json` — this IS valid for project-level MCP, but `code-review-graph` should integrate globally
- `.omp/hooks/` — this is **wrong**; OMP never looks here. Hooks belong under `agent/hooks/`

OMP's project-level and global-level structures differ:
- **Project MCP**: `.omp/mcp.json` (flat, directly under `.omp/`)
- **Global MCP**: `~/.omp/agent/mcp.json` (nested under `agent/`)
- **Project hooks**: `.omp/agent/hooks/` (nested under `agent/`)
- **Global hooks**: `~/.omp/agent/hooks/` (nested under `agent/`)

Because `code-review-graph` is a user-level tool (one install serves all repos via `cwd`), the integration should use global paths — consistent with Windsurf, Zed, Continue, and other global-config platforms.

## Goals / Non-Goals

**Goals:**
- Make `code-review-graph install --platform omp` write to paths OMP actually reads
- Use global `~/.omp/agent/mcp.json` for MCP server registration
- Use global `~/.omp/agent/hooks/` for hook installation
- Only install when OMP is detected (`~/.omp/` exists)
- Preserve existing config and hooks on re-install (idempotent)

**Non-Goals:**
- Project-level OMP config (`.omp/mcp.json`) — out of scope; global is the canonical integration point for `code-review-graph`
- OMP skills, custom tools, TTSR rules, or slash commands
- Changing any other platform's behavior

## Decisions

### 1. Global config over project-level
**Decision**: Write to `~/.omp/agent/mcp.json` and `~/.omp/agent/hooks/` instead of project-level `.omp/mcp.json` and `.omp/hooks/`.

**Rationale**: `code-review-graph` is a user-level MCP server. Its `mcp` subcommand serves the graph for whatever repo it is invoked from. Writing per-project config would require re-running `install` in every repo, and each install would overwrite the previous `cwd`. Global config is the natural fit.

**Alternative considered**: Keeping project-level `.omp/mcp.json` and only fixing hooks to `.omp/agent/hooks/`. Rejected because it perpetuates the per-project installation burden and contradicts how other global-config platforms are handled.

### 2. Detect `~/.omp/` existence
**Decision**: Change `detect` from `lambda: True` to `lambda: (Path.home() / ".omp").exists()`.

**Rationale**: Unconditional detection causes `install --platform all` to always write OMP configs even when the user does not have OMP installed. This is inconsistent with every other platform.

### 3. JSONC compatibility for config reads
**Decision**: Apply the same JSONC stripping (single-line comments, trailing commas) already used for Zed.

**Rationale**: OMP's config loader may write comments or trailing commas into `~/.omp/agent/mcp.json` during its own `/mcp add` flows. Naive `json.loads` would crash. This is defensive and consistent with existing platform handling.

### 4. Keep existing hook behavior
**Decision**: Preserve the current TypeScript hook content (`session_start` + `tool_result` handlers).

**Rationale**: The hook logic is correct; only the install location is wrong.

## Risks / Trade-offs

- **[Risk]** User has project-level `.omp/mcp.json` that shadows the global config.
  → **Mitigation**: Document that `code-review-graph install --platform omp` writes global config. Users with project-level overrides already know how OMP precedence works.

- **[Risk]** Global config gets `cwd` from the first repo installed in; subsequent repos skip because `code-review-graph` already exists.
  → **Mitigation**: This is pre-existing behavior shared with Windsurf, Zed, Continue, etc. The server entry includes `cwd`, and users can re-run with `--force` or edit the JSON if they need to switch repos.

- **[Risk]** `~/.omp/agent/mcp.json` does not exist yet (fresh OMP install).
  → **Mitigation**: `install_platform_configs` already creates parent directories and writes fresh JSON.

## Migration Plan

No runtime migration needed. After this fix:
```bash
code-review-graph install --platform omp
```

Users who ran the buggy version will have stale `.omp/mcp.json` and `.omp/hooks/` files in any repo where they previously installed. These are harmless — OMP ignores them. No cleanup action required.

## Open Questions

- None.
