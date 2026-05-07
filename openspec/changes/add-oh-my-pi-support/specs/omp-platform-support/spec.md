## ADDED Requirements

### Requirement: MCP server configuration for Oh My Pi
The system SHALL register the code-review-graph MCP server with Oh My Pi via a workspace-level `.omp/mcp.json` file.

#### Scenario: Installing for OMP explicitly
- **WHEN** the user runs `code-review-graph install --platform omp`
- **THEN** the system creates or updates `.omp/mcp.json` with a `mcpServers.code-review-graph` entry containing the command, args, and type fields

#### Scenario: Installing for all platforms with OMP present
- **WHEN** the user runs `code-review-graph install --platform all` in a repository containing a `.omp/` directory
- **THEN** the system includes `.omp/mcp.json` in the set of configured platform files

#### Scenario: Idempotent re-installation
- **WHEN** the user runs `code-review-graph install --platform omp` and `.omp/mcp.json` already contains the code-review-graph server entry
- **THEN** the system skips the write and reports "already configured"

### Requirement: OMP hook generation
The system SHALL generate a `.omp/hooks/code-review-graph.ts` file containing a TypeScript hook module that registers event handlers.

#### Scenario: First-time hook installation
- **WHEN** the user runs `code-review-graph install --platform omp` and `.omp/hooks/code-review-graph.ts` does not exist
- **THEN** the system creates the `.omp/hooks/` directory and writes a `code-review-graph.ts` module with a default export factory registering event handlers

#### Scenario: Hook follows OMP module contract
- **WHEN** the generated hook module is inspected
- **THEN** it SHALL contain a default-exported factory function receiving a `HookAPI`, register at least one event handler via `pi.on(...)`, and handle errors with try/catch

#### Scenario: Hook shows graph status on session start
- **WHEN** an OMP session starts and the hook is loaded
- **THEN** the hook SHALL run `code-review-graph status --brief` and log the output

#### Scenario: Hook auto-updates graph on file edits
- **WHEN** a `tool_result` event fires for a `write` or `edit` tool call
- **THEN** the hook SHALL run `code-review-graph update --skip-flows` asynchronously, swallowing any errors

### Requirement: CLI platform integration
The system SHALL recognize `omp` as a valid platform choice in the CLI.

#### Scenario: Platform appears in help text
- **WHEN** the user runs `code-review-graph install --help`
- **THEN** `omp` SHALL appear in the list of supported platform choices

#### Scenario: Install handler routes to OMP
- **WHEN** the user runs `code-review-graph install --platform omp`
- **THEN** the install handler SHALL invoke the OMP-specific MCP configuration and hook generation code paths

### Requirement: Test coverage for OMP integration
The system SHALL include unit tests verifying OMP hook generation and MCP config installation.

#### Scenario: Hook generation test
- **WHEN** the test suite runs the OMP hook installer in a temporary directory
- **THEN** it SHALL assert that `.omp/hooks/code-review-graph.ts` exists, contains a default export, and registers event handlers

#### Scenario: MCP config installation test
- **WHEN** the test suite runs the OMP platform configuration in a temporary directory
- **THEN** it SHALL assert that `.omp/mcp.json` is created with the correct `mcpServers.code-review-graph` structure
