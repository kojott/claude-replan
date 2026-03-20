# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-03-20

### Added
- Initial release as installable Claude Code marketplace plugin
- Parallel subagent dispatch for plan validation from multiple perspectives
- 10 review perspectives: codebase alignment, best practices, project standards, feasibility, fresh perspective, research, design/UX, data/schema, security, operations
- Adaptive agent selection based on plan type and content
- Automatic plan updates based on agent findings
- Severity categorization (critical → important → minor)
- Support for any plan type: implementation, research, design, migration, refactor, infrastructure
- "Fresh perspective" agent that reads plans cold to find blind spots

[1.0.0]: https://github.com/kojott/claude-replan/releases/tag/v1.0.0
