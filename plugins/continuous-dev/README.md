# CT Plugin

Continuous Team plugin for Claude Code. Provides shared commands, agents, and skills for the Continuous engineering team.

## Components

### Commands
- `/ct:commit` — Create git commits following project conventions
- `/ct:create-plan` — Interactive planning process for features and tasks
- `/ct:implement-plan` — Implement a development plan with validation phases
- `/ct:review-code` — Delegate code reviews to specialized agents
- `/ct:review-plan` — Review plans for ambiguity and completeness
- `/ct:review-spec` — Review specs for clarity and identify needed clarifications

### Agents
- `@acceptance-test-pro` — Acceptance testing of implemented features against specifications
- `@code-review-expert` — Code reviews, security analysis, and code quality assessment
- `@codebase-analyzer` — Deep-dive analysis of specific codebase components
- `@codebase-locator` — Locate files, directories, and components relevant to a task
- `@codebase-pattern-finder` — Find similar implementations and existing patterns to model after
- `@database-efcore-expert` — Database design, EF Core migrations, and PostgreSQL optimization
- `@debug-investigator` — Bug investigation and unexpected behavior analysis
- `@dotnet-backend-expert` — .NET backend development, ASP.NET Core APIs, CQRS/MediatR patterns
- `@fullstack-testing-expert` — Vitest, Playwright E2E, xUnit, and integration testing strategies
- `@integration-workflow-expert` — Workflow execution, third-party API integrations, OAuth/SAML, webhooks
- `@react-typescript-expert` — React 18+/TypeScript, Material-UI, Recoil, TanStack Query, ReactFlow
- `@thoughts-analyzer` — Deep-dive research on topics (research equivalent of codebase-analyzer)
- `@thoughts-locator` — Discover relevant documents in the thoughts/ directory
- `@web-search-researcher` — Web search for information beyond training data

### Skills (auto-loaded by context)
- **csharp-dotnet** — C#/.NET patterns and conventions
- **python** — Python patterns and conventions
- **typescript** — TypeScript/Node.js patterns and conventions
- **react** — React component and hook patterns
- **postgresql** — PostgreSQL schema and query patterns
- **sqlserver** — SQL Server schema and query patterns

## Installation

```
/plugin marketplace add https://dev.azure.com/SMATechnologies/Internal/_git/ClaudeCode
/plugin install ct@continuous-tools
```

## Updates

```
/plugin marketplace update continuous-tools
```
