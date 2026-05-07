## 1. Fix PLATFORMS Entry for Global Config

- [ ] 1.1 Update `PLATFORMS["omp"]["config_path"]` from `root / ".omp" / "mcp.json"` to `Path.home() / ".omp" / "agent" / "mcp.json"`
- [ ] 1.2 Update `PLATFORMS["omp"]["detect"]` from `lambda: True` to `lambda: (Path.home() / ".omp").exists()`
- [ ] 1.3 Add JSONC comment stripping for OMP config reads (same pattern as Zed: strip `//` comments and trailing commas before `json.loads`)

## 2. Fix Hook Install to Global Path

- [ ] 2.1 Update `install_omp_hooks()` to write to `Path.home() / ".omp" / "agent" / "hooks"` instead of `repo_root / ".omp" / "hooks"`
- [ ] 2.2 Update `_omp_hook_content()` docstring to reflect the corrected global install path
- [ ] 2.3 Verify `install_omp_hooks()` still accepts `repo_root` parameter for API compatibility but ignores it (matches `install_codex_hooks` pattern)

## 3. Update Tests

- [ ] 3.1 Update `TestInstallOmpHooks` assertions to expect `~/.omp/agent/hooks/code-review-graph.ts`
- [ ] 3.2 Update `TestInstallOmpHooks` parent directory assertions to expect `~/.omp/agent/hooks`
- [ ] 3.3 Update `test_install_omp_config` to expect `~/.omp/agent/mcp.json`
- [ ] 3.4 Update `test_install_omp_preserves_existing_servers` to write pre-existing config to `~/.omp/agent/mcp.json`
- [ ] 3.5 Update `test_install_omp_no_duplicate` to write pre-existing config to `~/.omp/agent/mcp.json`
- [ ] 3.6 Add test for `detect` behavior: when `~/.omp/` does not exist, `install --platform all` should NOT include Oh My Pi
- [ ] 3.7 Run `pytest tests/test_skills.py -k omp -v` and verify all tests pass

## 4. Update Documentation

- [ ] 4.1 Update README.md supported platforms table: Oh My Pi config from `.omp/mcp.json + .omp/hooks/code-review-graph.ts` to `~/.omp/agent/mcp.json + ~/.omp/agent/hooks/code-review-graph.ts`
- [ ] 4.2 Update docs/USAGE.md supported platforms table with the same corrected paths

## 5. Verification

- [ ] 5.1 Run full test suite: `pytest tests/test_skills.py -v`
- [ ] 5.2 Run mypy: `mypy code_review_graph/skills.py`
- [ ] 5.3 Verify `code-review-graph install --platform omp --dry-run` shows `~/.omp/agent/mcp.json`
