## ADDED Requirements

### Requirement: Oh My Pi MCP config uses global agent path
The system SHALL write Oh My Pi MCP configuration to `~/.omp/agent/mcp.json`.

#### Scenario: Fresh install on system with OMP present
- **WHEN** `install_platform_configs(repo_root, target="omp")` is called and `~/.omp/` exists
- **THEN** the system creates `~/.omp/agent/mcp.json` containing the `code-review-graph` entry under `mcpServers`

#### Scenario: Install merges with existing global agent config
- **WHEN** `install_platform_configs(repo_root, target="omp")` is called and `~/.omp/agent/mcp.json` already exists with other MCP servers
- **THEN** the system preserves all existing servers and appends `code-review-graph` to `mcpServers`

#### Scenario: Reinstall is idempotent
- **WHEN** `install_platform_configs(repo_root, target="omp")` is called and `code-review-graph` already exists in `mcpServers`
- **THEN** the system skips writing and reports "already configured"

#### Scenario: Install skips when OMP is not present
- **WHEN** `install_platform_configs(repo_root, target="all")` is called and `~/.omp/` does not exist
- **THEN** the system does not include "Oh My Pi" in the configured platforms list

### Requirement: Oh My Pi hooks use global agent path
The system SHALL install Oh My Pi hooks to `~/.omp/agent/hooks/code-review-graph.ts`.

#### Scenario: Fresh hook install
- **WHEN** `install_omp_hooks(repo_root)` is called
- **THEN** the system creates `~/.omp/agent/hooks/code-review-graph.ts` with valid TypeScript content

#### Scenario: Hook overwrite is idempotent
- **WHEN** `install_omp_hooks(repo_root)` is called and `~/.omp/agent/hooks/code-review-graph.ts` already exists
- **THEN** the system overwrites the file with the current hook content

### Requirement: Config path reflects OMP global structure in documentation
The system SHALL document the Oh My Pi config path as `~/.omp/agent/mcp.json` in all user-facing documentation.

#### Scenario: README lists correct path
- **WHEN** a user reads the supported platforms table in README.md
- **THEN** the Oh My Pi row shows config file `~/.omp/agent/mcp.json`

#### Scenario: USAGE guide lists correct path
- **WHEN** a user reads the supported platforms table in docs/USAGE.md
- **THEN** the Oh My Pi row shows config file `~/.omp/agent/mcp.json`
