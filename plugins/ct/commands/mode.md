---
description: Switch between Quality and Balanced modes
argument-hint: [quality] [balanced]
allowed-tools: Read Write Edit
---

# Set Plugin Mode

You are tasked with viewing or changing the ct plugin's operating mode. The mode controls which AI models are used for different agent tiers.

## Modes

| Mode | Tier 1 (Planning/Analysis) | Tier 2 (Execution/Research) | Tier 3 (Search/Locate) |
|------|----------------------------|------------------------------|-------------------------|
| **Quality** (default) | `opus` | `opus` | `sonnet` |
| **Balanced** | `opus` | `sonnet` | `haiku` |

### Tier 1 Agents (Always Opus)
These agents handle complex reasoning, architecture, and critical analysis:
- `codebase-analyzer` - Deep implementation analysis
- `code-review-expert` - Security and quality review
- `debug-investigator` - Root cause analysis

### Tier 2 Agents (Opus or Sonnet)
These agents handle code generation, pattern analysis, research, and testing:
- `codebase-pattern-finder` - Pattern discovery
- `web-search-researcher` - Web research
- `dotnet-backend-expert` - .NET implementation
- `react-typescript-expert` - React/TS implementation
- `nextjs-fullstack-expert` - Next.js implementation
- `python-pro-backend` - Python implementation
- `database-efcore-expert` - Database implementation
- `integration-workflow-expert` - Integration implementation
- `acceptance-test-pro` - Acceptance testing
- `fullstack-testing-expert` - Test writing

### Tier 3 Agents (Opus or Haiku)
These agents perform mechanical search-and-organize tasks with no code analysis:
- `codebase-locator` - File/component finding

## Process

1. **Read current mode** from `plugins/ct/config/mode.md`

2. **If no arguments were provided**, use the `AskUserQuestion` tool to let the user pick a mode interactively:
   - Use the `AskUserQuestion` tool with a single question
   - Set `header` to "Mode"
   - Set `question` to "Current mode: **[CURRENT_MODE]**. Which mode would you like to use?"
   - Provide these options:
     - Label: `Quality (default)`, Description: `Opus for Tier 1/2, Sonnet for Tier 3. Best for complex, high-stakes work.`
     - Label: `Balanced`, Description: `Opus for Tier 1 planning/analysis, Sonnet for Tier 2 execution/research, Haiku for Tier 3 search. Faster and more cost-effective.`
   - If the user picks the same mode that's already active, confirm no change was made and stop.

3. **If an argument was provided** (e.g., `/ct:mode quality` or `/ct:mode balanced`):
   - Parse the argument as the desired mode
   - Skip the prompt and go directly to step 4

4. **Update the mode**:
   - Write the new mode to `plugins/ct/config/mode.md` using this exact format:
     ```
     # Mode Configuration

     mode: [quality|balanced]
     ```
   - Confirm the change:
     ```
     Mode updated: [OLD_MODE] -> [NEW_MODE]

     Tier 1 agents (codebase-analyzer, code-review-expert, debug-investigator) will use: opus
     Tier 2 agents (execution/research/testing) will use: [opus if quality | sonnet if balanced]
     Tier 3 agents (codebase-locator) will use: [sonnet if quality | haiku if balanced]
     ```

5. **If user selects the same mode or says "keep"**, confirm no change was made.

## Important
- Only two valid modes: `quality` and `balanced`
- If user provides an invalid mode, show the available options
- The mode file must always use lowercase: `quality` or `balanced`
- Default mode (if config is missing or corrupted) is `quality`
