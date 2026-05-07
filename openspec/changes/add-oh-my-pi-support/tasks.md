## 1. MCP Config & Platform Registry

- [x] 1.1 Add `omp` entry to `PLATFORMS` dict in `code_review_graph/skills.py` with `.omp/mcp.json` path, `mcpServers` key, object format, and `needs_type=True`
- [x] 1.2 Add `omp` to `_PLATFORM_CHOICES` in `code_review_graph/cli.py`

## 2. OMP Hook Generation

- [x] 2.1 Create `_omp_hook_content() -> str` in `code_review_graph/skills.py` returning TypeScript source for `.omp/hooks/code-review-graph.ts`
- [x] 2.2 Register `session_start` handler that runs `code-review-graph status --brief` and logs output
- [x] 2.3 Register `tool_result` handler that triggers `code-review-graph update --skip-flows` on `write`/`edit` tool calls, with error swallowing
- [x] 2.4 Create `install_omp_hooks(repo_root: Path) -> Path` that writes `.omp/hooks/code-review-graph.ts`
- [x] 2.5 Wire `install_omp_hooks()` into `_handle_init()` in `code_review_graph/cli.py`

## 3. Tests

- [x] 3.1 Add test in `tests/test_skills.py` verifying `install_omp_hooks` creates `.omp/hooks/code-review-graph.ts` with default export and event handler registrations
- [x] 3.2 Add test in `tests/test_skills.py` verifying OMP platform MCP config installation writes correct `.omp/mcp.json` structure
- [x] 3.3 Run `pytest tests/test_skills.py -v` and fix any failures

## 4. Documentation

- [x] 4.1 Add `omp` to the platform list in `docs/USAGE.md`
- [x] 4.2 Add `omp` to the platform list in `README.md`
- [x] 4.3 Verify `python -m code_review_graph install --help` shows `omp` in platform choices
