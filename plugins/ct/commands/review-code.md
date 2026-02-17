---
description: Delegate code reviews to specialized review agents
argument-hint: [plan-file-path] [directory-path]
---

# Code Review

## Objective
Delegate code review tasks to the specialized code-review-expert agent. Supports both comprehensive directory reviews and focused reviews of recent changes from plan implementations.

## Mode-Aware Model Selection

**Before spawning any agent sub-tasks**, read `plugins/ct/config/mode.md` to determine the active mode.

Apply the following model assignment when using the Task tool:
- **Quality mode**: Use `model: opus` for Tier 1 and Tier 2 agents, `model: sonnet` for Tier 3 agents
- **Balanced mode**:
  - Use `model: opus` for **Tier 1** agents: `codebase-analyzer`, `code-review-expert`, `debug-investigator`
  - Use `model: sonnet` for **Tier 2** agents: `codebase-pattern-finder`, `web-search-researcher`, and all implementation/testing agents
  - Use `model: haiku` for **Tier 3** agents: `codebase-locator`

Always include the `model` parameter on every Task tool call based on the active mode and the agent's tier.

## Input Parameters & Mode Detection
- **Directory Path**: Comprehensive review of all code files in the directory
- **Plan File Path**: Focused review of code changes from implementing the plan

**Mode Detection Process:**
1. Check if input ends with `.md` → Plan Review Mode
2. Check if input is a valid directory → Directory Review Mode  
3. If ambiguous, call `AskUserQuestion` with:
   - Question: "I couldn't determine the review mode from your input. Which type of review would you like?"
   - Options:
     - "Directory Review" — Comprehensive review of all code files in the directory
     - "Plan Review" — Review only recent changes from plan implementation
   - **Do NOT output the options as plain text and end your response.**

## Execution Process

### Plan Review Mode
1. Validate the plan file exists
2. Delegate to code-review-expert agent:
   ```
   Task: Code Review Expert - Review plan implementation changes
   Context: User implemented [PLAN_FILE] and wants to validate the changes are correct and secure
   Plan File: [PLAN_FILE_PATH]
   Review Type: Plan Review (focused on recent git changes)

   Requirements:
   - Use git diff to identify changed files from recent commit
   - Validate implementation matches plan requirements
   - Focus on security issues and plan adherence

   Evidence Requirements (MANDATORY for each finding):
   - Exact code snippet showing the issue
   - Explanation of the vulnerability/problem and why it matters
   - Attack vector or failure mode (how it could be exploited or cause failure)
   - Mitigation check confirming no upstream sanitization, framework protection, or other mitigation exists
   - If you cannot provide concrete evidence for all four points, the issue belongs in "Needs Further Investigation" not "Confirmed Issues"

   Verification Phase (MANDATORY before finalizing report):
   - Re-read each flagged code section with 50+ lines of surrounding context
   - Search for related validation, sanitization, or error handling elsewhere in the codebase
   - Check if the framework or library provides built-in protection
   - Verify the issue is reachable (not dead code, behind feature flags, or test-only)
   - If ANY doubt remains after verification, move the finding to "Needs Further Investigation"

   Report Structure (MANDATORY format):
   ## Confirmed Issues
   Verified issues with concrete evidence. Each must include:
   - Location: file_path:line_number
   - Severity: CRITICAL / HIGH / MEDIUM / LOW
   - Evidence: Code snippet showing the issue
   - Proof: Attack vector or failure mode explanation
   - Mitigation Status: Confirmation no protection exists
   - Recommended Fix: Specific code changes

   ## Needs Further Investigation
   Potential concerns that could not be conclusively verified. Include:
   - Location: file_path:line_number
   - Concern: What caught attention
   - Uncertainty: Why it couldn't be confirmed
   - Suggested Follow-up: What's needed to confirm or dismiss

   ## Reviewed and Dismissed
   Items flagged during review but confirmed NOT to be issues:
   - Location: file_path:line_number
   - Initial Concern: What was flagged
   - Dismissal Reason: Why this is not an issue
   ```

### Directory Review Mode  
1. Validate the directory exists
2. Delegate to code-review-expert agent:
   ```
   Task: Code Review Expert - Comprehensive directory review
   Context: User wants comprehensive code quality and security analysis of [DIRECTORY]
   Directory: [DIRECTORY_PATH]
   Review Type: Directory Review (comprehensive analysis)

   Requirements:
   - Analyze all source code files recursively
   - Check for OWASP Top 10 security vulnerabilities
   - Assess code quality, performance, and architecture

   Evidence Requirements (MANDATORY for each finding):
   - Exact code snippet showing the issue
   - Explanation of the vulnerability/problem and why it matters
   - Attack vector or failure mode (how it could be exploited or cause failure)
   - Mitigation check confirming no upstream sanitization, framework protection, or other mitigation exists
   - If you cannot provide concrete evidence for all four points, the issue belongs in "Needs Further Investigation" not "Confirmed Issues"

   Verification Phase (MANDATORY before finalizing report):
   - Re-read each flagged code section with 50+ lines of surrounding context
   - Search for related validation, sanitization, or error handling elsewhere in the codebase
   - Check if the framework or library provides built-in protection
   - Verify the issue is reachable (not dead code, behind feature flags, or test-only)
   - If ANY doubt remains after verification, move the finding to "Needs Further Investigation"

   Report Structure (MANDATORY format):
   ## Confirmed Issues
   Verified issues with concrete evidence. Each must include:
   - Location: file_path:line_number
   - Severity: CRITICAL / HIGH / MEDIUM / LOW
   - Evidence: Code snippet showing the issue
   - Proof: Attack vector or failure mode explanation
   - Mitigation Status: Confirmation no protection exists
   - Recommended Fix: Specific code changes

   ## Needs Further Investigation
   Potential concerns that could not be conclusively verified. Include:
   - Location: file_path:line_number
   - Concern: What caught attention
   - Uncertainty: Why it couldn't be confirmed
   - Suggested Follow-up: What's needed to confirm or dismiss

   ## Reviewed and Dismissed
   Items flagged during review but confirmed NOT to be issues:
   - Location: file_path:line_number
   - Initial Concern: What was flagged
   - Dismissal Reason: Why this is not an issue
   ```

## Error Handling
- If plan file doesn't exist: "Error: Plan file not found: [PATH]"
- If directory doesn't exist: "Error: Directory not found: [PATH]"
- If input is ambiguous: Present mode selection options to user

## Usage Examples
```
Plan Review:
User: Review the code changes from PLAN_005_API_FIXES.md
Claude: [Delegates to code-review-expert agent for plan-based review]

Directory Review:
User: Review the code in ./src
Claude: [Delegates to code-review-expert agent for comprehensive review]
```
