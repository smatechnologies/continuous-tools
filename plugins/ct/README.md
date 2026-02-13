# CT Plugin

Continuous Team plugin for Claude Code. Provides shared commands, agents, and skills for the Continuous engineering team.

## Components

### Commands
- `/ct:commit` — Create git commits following project conventions
- `/ct:create-plan` — Interactive planning process for features and tasks
- `/ct:create-spec` — Interactive feature specification creation
- `/ct:debug` — Investigate and fix bugs using debug-investigator agent
- `/ct:implement-plan` — Implement a development plan with validation phases
- `/ct:info` — Display the plugin dashboard with logo, version, mode, and available commands/agents
- `/ct:mode` — Switch between Quality and Balanced model modes
- `/ct:research-codebase` — Research the codebase with parallel agents and generate a documented report
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
- `@nextjs-fullstack-expert` — Next.js 14+ App Router, server components, full-stack development
- `@react-typescript-expert` — React 18+/TypeScript, Material-UI, Recoil, TanStack Query, ReactFlow
- `@web-search-researcher` — Web search for information beyond training data

### Mode System

The plugin supports two operating modes that control which AI models are used for agent sub-tasks. Switch modes with `/ct:mode`.

| Mode | Tier 1 (Planning/Analysis) | Tier 2 (Execution/Research) | Tier 3 (Search/Locate) |
|------|----------------------------|------------------------------|-------------------------|
| **Quality** (default) | Opus | Opus | Sonnet |
| **Balanced** | Opus | Sonnet | Haiku |

**Tier 1 agents** (always Opus): `codebase-analyzer`, `code-review-expert`, `debug-investigator`
**Tier 2 agents** (Opus or Sonnet): `codebase-pattern-finder`, `web-search-researcher`, and all implementation/testing agents
**Tier 3 agents** (Sonnet or Haiku): `codebase-locator`

Configuration is stored in `config/mode.md`.

## Installation

### Manual

```
/plugin marketplace add smatechnologies/continuous-tools
/plugin install ct@continuous-tools
```

### Auto-prompt for a project

Add this to your project's `.claude/settings.json` so team members are automatically prompted to install when they open the project:

```json
{
  "extraKnownMarketplaces": {
    "continuous-tools": {
      "source": {
        "source": "github",
        "repo": "smatechnologies/continuous-tools"
      }
    }
  },
  "enabledPlugins": {
    "ct@continuous-tools": true
  }
}
```

## Updates

### Manual Update
```
/plugin marketplace update continuous-tools
```

### Auto-Update
Enable automatic updates for the marketplace:
```
/plugin → Marketplaces tab → continuous-tools → Enable auto-update
```
