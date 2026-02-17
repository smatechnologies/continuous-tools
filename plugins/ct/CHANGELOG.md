# Changelog

All notable changes to the **ct** plugin will be documented in this file.

## [1.0.5] - 2026-02-17
### Improved
- `/ct:implement-plan`, `/ct:review-code`, `/ct:debug`, and `/ct:commit` now use `AskUserQuestion` for inline interactive prompting, completing the migration across all commands.

## [1.0.4] - 2026-02-17
### Improved
- `/ct:review-plan` and `/ct:review-spec` now use `AskUserQuestion` for inline interactive questioning, matching the pattern established in v1.0.3 for create commands.

## [1.0.3] - 2026-02-17
### Improved
- `/ct:create-plan` and `/ct:create-spec` now use `AskUserQuestion` for inline interactive questioning — one question at a time in a sequential loop, so users answer directly in the running prompt instead of typing out all answers in a separate message.

## [1.0.2] - 2026-02-17
### Improved
- `/ct:create-plan` now writes a separate plan file per phase instead of collapsing all phases into a single file. Each phase file is self-contained with its own tasks, code examples, testing checklist, and cross-phase dependency references.

## [1.0.1] - 2026-02-13
### Fixed
- Updated `/ct:debug` command and `debug-investigator` agent to more consistently show all potential solutions.

## [1.0.0] - Initial Release
### Added
- 14 specialized agents with tiered model selection (Quality/Balanced modes)
- `/ct:create-spec` — interactive feature specification creation
- `/ct:review-spec` — spec review for clarity and completeness
- `/ct:research-codebase` — deep codebase research and reporting
- `/ct:create-plan` — implementation plan builder
- `/ct:review-plan` — plan review for ambiguity and feasibility
- `/ct:implement-plan` — phased plan execution with validation
- `/ct:commit` — git commits following project conventions
- `/ct:review-code` — delegated code reviews via specialized agents
- `/ct:debug` — bug investigation with debug-investigator agent
- `/ct:mode` — switch between Quality and Balanced modes
- `/ct:info` — plugin info dashboard
- GitHub marketplace distribution
