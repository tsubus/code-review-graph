## Context

Code-review-graph supports 13+ AI coding platforms via `code_review_graph/skills.py` and `code_review_graph/cli.py`. Each platform integration follows a predictable pattern:

1. **MCP config**: an entry in `PLATFORMS` dict with config path, key, detect lambda, format, and type flag
2. **Skills**: a generator function that writes platform-native skill files (e.g., `generate_skills` for Claude, `install_gemini_cli_skills` for Gemini CLI)
3. **Hooks**: optional event hooks for auto-update (Claude, Codex, Cursor, Gemini CLI, OpenCode)
4. **Instructions**: injection of "use graph first" guidance into platform instruction files (Claude MD, Copilot instructions, etc.)
5. **CLI wiring**: add platform to `_PLATFORM_CHOICES` and branch in `_handle_init`

Oh My Pi (OMP) is a terminal-based AI coding agent by @can1357. It has three integration surfaces relevant to code-review-graph:

- **MCP servers**: configured via `.omp/mcp.json` (workspace-level) with standard `mcpServers` object
- **Hooks**: TypeScript/JavaScript modules in `.omp/hooks/` that export a factory receiving `HookAPI`. They register event handlers (`session_start`, `tool_call`, `tool_result`, etc.) to intercept and mutate behavior transparently.
- **Custom tools**: TypeScript/JavaScript modules in `.omp/tools/` that export a factory receiving `CustomToolAPI` — model-callable functions.

The user explicitly requested hooks over skills because skills are passive prompt guidance that the model may ignore, whereas hooks provide reliable event interception. The user also prefers a single integration surface rather than multiple.

## Goals / Non-Goals

**Goals:**
- Enable `code-review-graph install --platform omp` to register the MCP server in `.omp/mcp.json`
- Generate `.omp/hooks/code-review-graph.ts` — a TypeScript hook module that shows graph status on `session_start` and auto-updates the graph on file edits (`tool_result` for `write`/`edit`)
- Ensure generated TypeScript follows OMP's module contract (default export factory, `pi.exec` for shelling out, proper error handling with try/catch)
- Add tests for hook generation and MCP config installation

**Non-Goals:**
- OMP skills — explicitly out of scope per user request; skills are unreliable
- OMP custom tools — out of scope per user request; prefer a single integration surface
- OMP TTSR rules — out of scope for initial integration
- OMP slash commands — out of scope; can be added later via `.omp/commands/`
- Global user-level OMP config — keep project-scoped like other platforms

## Decisions

### 1. MCP Config: `.omp/mcp.json` workspace-level object format
**Rationale**: Consistent with other project-scoped platforms (Claude Code's `.mcp.json`, Qoder's `.qoder/mcp.json`). `detect()` returns `True` unconditionally so `install --platform omp` always works.

### 2. Hook: `.omp/hooks/code-review-graph.ts` with session start + file edit events
**Rationale**: Mirrors the hook behavior already provided for Claude (`SessionStart`), Codex, Cursor, and OpenCode (`file.edited`). The hook:
- On `session_start`: runs `code-review-graph status --brief` and logs output
- On `tool_result` for `write`/`edit`: runs `code-review-graph update --skip-flows` asynchronously

This provides transparent auto-behavior without relying on the model to read and follow skill prompts.

**Alternative considered**: Custom tools — rejected because the user wants a single integration surface and hooks provide auto-update behavior that custom tools cannot.

### 3. TypeScript generation from Python strings
**Rationale**: Consistent with the existing OpenCode plugin (`_opencode_plugin_content` in `skills.py`) which already generates TypeScript source from a Python string. The generated TypeScript does not need compilation — OMP loads it directly via Bun.

### 4. Error handling: fail-safe with try/catch
**Rationale**: Per OMP hooks documentation, all handlers should wrap async work in try/catch. Errors are swallowed so they never break the editor session. This matches the existing OpenCode plugin pattern.

## Risks / Trade-offs

| Risk | Mitigation |
|---|---|
| OMP hook API changes | Keep module simple; follow documented contract closely; only the string template needs updating |
| `.omp/mcp.json` is not the canonical config path | Monitor OMP releases; update `PLATFORMS["omp"]["config_path"]` if needed |
| Hook errors breaking the session | All handlers wrap async work in try/catch and swallow errors (fail-safe pattern) |
| Model doesn't use graph MCP tools without skill prompts | MCP tools are explicitly callable by the model; hooks handle auto-behavior; this is acceptable per user's preference for hooks over skills |

## Migration Plan

No migration needed — pure additive feature.

## Open Questions

- None resolved.
