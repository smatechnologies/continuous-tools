---
description: Investigate and fix bugs using debug-investigator agent
argument-hint: [bug-description]
---

# Debug Issue

## Objective
Investigate and fix bugs, unexpected behavior, or system malfunctions by delegating to the debug-investigator agent, then ensuring findings are presented and fixes are applied.

## Mode-Aware Model Selection

**Before spawning any agent sub-tasks**, read `plugins/ct/config/mode.md` to determine the active mode.

Apply the following model assignment when using the Task tool:
- **Quality mode**: Use `model: opus` for Tier 1 and Tier 2 agents, `model: sonnet` for Tier 3 agents
- **Balanced mode**:
  - Use `model: opus` for **Tier 1** agents: `codebase-analyzer`, `code-review-expert`, `debug-investigator`
  - Use `model: sonnet` for **Tier 2** agents: `codebase-pattern-finder`, `web-search-researcher`, and all implementation/testing agents
  - Use `model: haiku` for **Tier 3** agents: `codebase-locator`

Always include the `model` parameter on every Task tool call based on the active mode and the agent's tier.

## Input Parameters
- **Bug Description**: Natural language description of the issue, error, or unexpected behavior

## Execution Process

### Step 1: Investigate the Issue
Delegate to debug-investigator agent:
```
Task: Debug Investigation - [BUG_DESCRIPTION]
Context: User reported: "[BUG_DESCRIPTION]"
Requirements:
- Systematically investigate the root cause
- Analyze relevant code, logs, and recent changes
- Identify the specific issue and location
- Propose a fix with implementation details
- Return detailed findings and fix proposal
```

### Step 2: Present Findings (if agent doesn't)
If the agent's report doesn't clearly present findings to the user:
1. Summarize the root cause identified
2. Show the problematic code location (file:line)
3. Explain the issue clearly
4. Present the proposed fix, as well as any alternatives (explain their tradeoffs and why you recommend the proposed fix)

### Step 3: Apply Fix (if agent doesn't)
If the agent didn't apply the fix:
1. Ask user for confirmation to apply the proposed fix
2. If confirmed, implement the fix:
   - For simple fixes: Apply directly using Edit tool
   - For complex fixes involving multiple files/systems: Delegate to appropriate specialized agent (python-pro-backend, nextjs-fullstack-expert, etc.)
3. Verify the fix works (run tests, check for errors)

## Error Handling
- If description is too vague: Ask user for more specific details (error messages, steps to reproduce, expected vs actual behavior)
- If investigation finds no issue: Report that no problem was detected and ask for more context
- If fix requires breaking changes: Warn user before applying

## Usage Examples
```
Simple Bug:
User: The login button throws a 401 error even with valid credentials
Claude: [Investigates → Presents findings → Applies auth fix]

Complex Bug:
User: Database queries are slow when filtering orders
Claude: [Investigates → Presents performance analysis → Delegates to python-pro-backend for optimization]

Vague Bug:
User: Something is broken
Claude: "Can you provide more details? What specific error or unexpected behavior are you seeing?"
```
