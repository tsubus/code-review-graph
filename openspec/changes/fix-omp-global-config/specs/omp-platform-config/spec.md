## ADDED Requirements

### Requirement: Oh My Pi config path uses agent subdirectory
The system SHALL write Oh My Pi MCP configuration to `<repo_root>/.omp/agent/config.json`.

#### Scenario: Fresh install on repo without existing OMP config
- **WHEN** `install_platform_configs(repo_root, target="omp")` is called on a repo with no `.omp/` directory
- **THEN** the system creates `.omp/agent/config.json` containing the `code-review-graph` entry under `mcpServers`

#### Scenario: Install merges with existing agent config
- **WHEN** `install_platform_configs(repo_root, target="omp")` is called and `.omp/agent/config.json` already exists with other settings
- **THEN** the system preserves all existing keys and appends `code-review-graph` to `mcpServers`

#### Scenario: Reinstall is idempotent
- **WHEN** `install_platform_configs(repo_root, target="omp")` is called and `code-review-graph` already exists in `mcpServers`
- **THEN** the system skips writing and reports "already configured"

### Requirement: Oh My Pi hooks use agent subdirectory
The system SHALL install Oh My Pi hooks to `<repo_root>/.omp/agent/hooks/code-review-graph.ts`.

#### Scenario: Fresh hook install
- **WHEN** `install_omp_hooks(repo_root)` is called on a repo without existing hooks
- **THEN** the system creates `.omp/agent/hooks/code-review-graph.ts` with valid TypeScript content

#### Scenario: Hook overwrite is idempotent
- **WHEN** `install_omp_hooks(repo_root)` is called and `.omp/agent/hooks/code-review-graph.ts` already exists
- **THEN** the system overwrites the file with the current hook content

### Requirement: Config path reflects OMP structure in documentation
The system SHALL document the Oh My Pi config path as `.omp/agent/config.json` in all user-facing documentation.

#### Scenario: README lists correct path
- **WHEN** a user reads the supported platforms table in README.md
- **THEN** the Oh My Pi row shows config file `.omp/agent/config.json`

#### Scenario: USAGE guide lists correct path
- **WHEN** a user reads the supported platforms table in docs/USAGE.md
- **THEN** the Oh My Pi row shows config file `.omp/agent/config.json`
