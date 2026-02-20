---
description: Create a detailed implementation plan
argument-hint: [spec-path] [research-file-path]
---

# Create Plan

You are tasked with creating detailed implementation plans through an interactive, iterative process. You should be skeptical, thorough, and work collaboratively with the user to produce high-quality technical specifications.

## Mode-Aware Model Selection

**Before spawning any agent sub-tasks**, read `plugins/ct/config/mode.md` to determine the active mode.

Apply the following model assignment when using the Task tool:
- **Quality mode**: Use `model: opus` for Tier 1 and Tier 2 agents, `model: sonnet` for Tier 3 agents
- **Balanced mode**:
  - Use `model: opus` for **Tier 1** agents: `codebase-analyzer`, `code-review-expert`, `debug-investigator`
  - Use `model: sonnet` for **Tier 2** agents: `codebase-pattern-finder`, `web-search-researcher`, and all implementation/testing agents
  - Use `model: haiku` for **Tier 3** agents: `codebase-locator`

Always include the `model` parameter on every Task tool call based on the active mode and the agent's tier.

## Initial Response

When this command is invoked:

1. **Check if parameters were provided**:
   - If a file path or ticket reference was provided as a parameter, skip the default message
   - Immediately read any provided files FULLY
   - Begin the research process

2. **If no parameters provided**, use the `AskUserQuestion` tool to prompt the user inline:
   - Call `AskUserQuestion` with a single question asking for the task/ticket description. Example question: "What task or feature are you planning? (Describe it or reference a ticket file like `/create_plan path/to/ticket.md`)"
   - After receiving the task description, proceed to Step 1 research — do NOT immediately ask follow-up questions about context or constraints. Let the research phase inform what clarifications are actually needed.

## Progress Signaling

**CRITICAL**: This is a long-running, multi-step process. You MUST signal progress to the user at every stage so they know the command is actively working and not hung.

1. **Step announcements**: Before beginning each step, announce it:
   > **Step N/6: [Step Name]** — [Brief description of what's happening next]...

2. **Research phase tracking**: When spawning research agents (which can take 30-60+ seconds), create a **single** TaskCreate task to cover the entire research phase (e.g., activeForm: "Researching codebase and gathering context..."). Mark it `in_progress` before spawning agents. Do NOT mark it `completed` until ALL agents have returned and you have processed their results. The individual agent calls already show their own progress in the UI — do not duplicate per-agent tracking with separate TaskCreate tasks.

3. **Research summaries**: After research agents return, present a brief summary of findings before moving on, so the user sees concrete output from the wait.

## Process Steps

### Step 1: Context Gathering & Initial Analysis

**Announce**: > **Step 1/6: Context Gathering & Initial Analysis** — Reading referenced files and researching the codebase...

1. **Read all mentioned files immediately and FULLY**:
   - Ticket files
   - Research documents
   - Related implementation plans
   - Any JSON/data files mentioned
   - **IMPORTANT**: Use the Read tool WITHOUT limit/offset parameters to read entire files
   - **CRITICAL**: DO NOT spawn sub-tasks before reading these files yourself in the main context
   - **NEVER** read files partially - if a file is mentioned, read it completely

2. **Spawn initial research tasks to gather context**:
   Before asking the user any questions, use specialized agents to research.

   **Create a single TaskCreate task for the research phase:**
   - `activeForm`: "Researching codebase for [feature/ticket topic]..."
   - Mark it `in_progress` immediately so the user sees a spinner
   - Do NOT mark it `completed` until ALL research agents below have returned and you have summarized findings

   **Prefer parallel execution**: When research agents are independent of each other, include all Task tool calls in a single message so they run concurrently. Only run agents sequentially when one agent's output is needed to inform the next agent's prompt.

   **CRITICAL ORDERING — codebase-locator MUST run before codebase-analyzer**:
   The codebase-locator identifies which files and components are relevant. The codebase-analyzer needs those file locations to know what to read and analyze. Always spawn codebase-locator first, wait for its results, then spawn codebase-analyzer with the locator's findings included in the prompt. Other independent agents (e.g., linear-ticket-reader) CAN run in parallel with codebase-locator.

   Agents to spawn:
   - **First**: Use the **codebase-locator** agent to find all files related to the ticket/task
   - **Then** (after locator returns): Use the **codebase-analyzer** agent to understand how the current implementation works, providing it the file locations from the locator
   - **In parallel with locator**: If a Linear ticket is mentioned, use the **linear-ticket-reader** agent to get full details

   These agents will:
   - Find relevant source files, configs, and tests
   - Trace data flow and key functions
   - Return detailed explanations with file:line references

3. **Read all files identified by research tasks**:
   - After research tasks complete, read ALL files they identified as relevant
   - Read them FULLY into the main context
   - This ensures you have complete understanding before proceeding

4. **Research summary checkpoint** — Present a consolidated summary of what research found:
   ```
   **Research Complete.** Here's what I found:
   - [X] files identified as relevant across [Y] directories
   - Key discovery: [Most important finding with file:line reference]
   - Pattern: [Relevant convention or pattern to follow]
   - Potential concern: [Any complexity or edge case spotted]
   ```

5. **Analyze and verify understanding**:
   - Cross-reference the ticket requirements with actual code
   - Identify any discrepancies or misunderstandings
   - Note assumptions that need verification
   - Determine true scope based on codebase reality

6. **Present informed understanding, then enter sequential questioning loop**:
   - First, output your research summary:
     ```
     Based on the ticket and my research of the codebase, I understand we need to [accurate summary].

     I've found that:
     - [Current implementation detail with file:line reference]
     - [Relevant pattern or constraint discovered]
     - [Potential complexity or edge case identified]
     ```
   - Then, generate (internally) a prioritized list of questions your research couldn't answer. Only include questions that genuinely require human judgment — NOT questions answerable by reading code.
   - **Sequential questioning loop**: Present EXACTLY ONE question at a time using the `AskUserQuestion` tool. For each question:
     - If there are clear discrete options, provide them as `AskUserQuestion` options (2-4 choices)
     - After the user answers, record the answer and proceed to the next question
     - Each subsequent question may be informed by prior answers
     - Stop asking when: all critical ambiguities are resolved, OR the user signals completion ("done", "let's proceed", etc.)
   - **CRITICAL**: Use the `AskUserQuestion` tool for each question so the user answers inline in the running prompt. Do NOT output questions as plain text and end your response.

   **If no questions are needed**, announce to the user:
   ```
   My research covered all the context needed — no clarifying questions required. Skipping Step 2 and moving directly to plan structure.
   ```
   Then skip Step 2 and proceed to Step 3.

### Step 2: Research & Discovery

**Announce**: > **Step 2/6: Research & Discovery** — Spawning research agents to investigate the codebase in depth...

After getting initial clarifications:

1. **If the user corrects any misunderstanding**:
   - DO NOT just accept the correction
   - Spawn new research tasks to verify the correct information
   - Read the specific files/directories they mention
   - Only proceed once you've verified the facts yourself

2. **Create a research todo list** using TodoWrite to track exploration tasks

3. **Spawn parallel sub-tasks for comprehensive research**:
   - Create a **single** TaskCreate task for this research phase (e.g., activeForm: "Deep-diving into [area] implementation and patterns..."). Mark `in_progress` before spawning, `completed` only after ALL agents return.
   - **Prefer parallel execution**: When research agents are independent of each other, include all Task tool calls in a single message so they run concurrently. Only run agents sequentially when one agent's output is needed to inform the next agent's prompt.
   - **CRITICAL ORDERING — codebase-locator MUST run before codebase-analyzer and codebase-pattern-finder**: Always spawn codebase-locator first, wait for its results, then spawn codebase-analyzer and codebase-pattern-finder (which CAN run in parallel with each other) with the locator's findings included in their prompts.
   - Use the right agent for each type of research:

   **For deeper investigation:**
   - **First**: **codebase-locator** - To find more specific files (e.g., "find all files that handle [specific component]")
   - **Then** (after locator returns, can run in parallel with each other): **codebase-analyzer** - To understand implementation details (e.g., "analyze how [system] works"), and **codebase-pattern-finder** - To find similar features we can model after. Provide both agents the file locations from the locator.

   Each agent knows how to:
   - Find the right files and code patterns
   - Identify conventions and patterns to follow
   - Look for integration points and dependencies
   - Return specific file:line references
   - Find tests and examples

**Specifically research for template sections:**
   - Current state analysis: What works (✅), what's broken (❌), what needs improvement (⚠️)
   - Concrete solution structures with code examples
   - Specific file paths and line numbers for each change
   - Testing strategies and rollback procedures
   - Common pitfalls specific to this codebase area

3. **Wait for ALL sub-tasks to complete** before proceeding. Only now mark the research phase TaskCreate task as `completed`.

4. **Present findings and design options, then ask for selection using `AskUserQuestion`**:
   - First, output your findings:
     ```
     Based on my research, here's what I found:

     **Current State:**
     - [Key discovery about existing code]
     - [Pattern or convention to follow]

     **Design Options:**
     1. [Option A] - [pros/cons]
     2. [Option B] - [pros/cons]
     ```
   - Then immediately call `AskUserQuestion` to get the user's design choice inline. Present each option as a selectable choice (e.g., "Option A: [summary]", "Option B: [summary]").
   - If there are open questions that affect the design choice, ask them via `AskUserQuestion` BEFORE presenting the design options.
   - **Do NOT output "Which approach?" as plain text and end your response.**

### Step 3: Plan Structure Development

**Announce**: > **Step 3/6: Plan Structure Development** — Drafting the plan outline and implementation phases...

Once aligned on approach:

1. **Create initial plan outline** — output the proposed structure:
   ```
   Here's my proposed plan structure:

   ## Overview
   [1-2 sentence summary]

   ## Implementation Phases:
   1. [Phase name] - [what it accomplishes]
   2. [Phase name] - [what it accomplishes]
   3. [Phase name] - [what it accomplishes]
   ```

2. **Get feedback on structure using `AskUserQuestion`** — immediately after outputting the outline above, call `AskUserQuestion` to ask the user for approval inline. Example:
   - Question: "Does this phasing make sense?"
   - Options: "Approve", "Needs adjustments"
   - If the user selects an adjustment option, iterate on the structure and ask again until approved.

### Step 4: Detailed Plan Writing

**Announce**: > **Step 4/6: Detailed Plan Writing** — Writing the full implementation plan with code examples and file references...

After structure approval:

1. **Write a separate plan file for EACH phase** approved in Step 3. For every phase (1, 2, 3, etc.), write a file to `project_docs/[git-branch-name]/Phase_[num]_Implementation_Plan.md` where:
   - `[git-branch-name]` is the current git branch name
   - `[num]` is the phase number (1, 2, 3, etc.) — one file per phase
   - Create the directory if it doesn't exist
   - **CRITICAL**: If Step 3 proposed N phases and the user approved them, you MUST produce N separate files (e.g., `Phase_1_...`, `Phase_2_...`, `Phase_3_...`). Do NOT collapse all phases into a single file.
   - Each phase file should be self-contained: it should include its own tasks, code examples, testing checklist, and success criteria relevant to that phase
   - Cross-reference other phase files in the "Dependencies and Prerequisites" or "Related plans" sections where applicable
2. **Use the exact PLAN_TEMPLATE.md format for each phase file**:

```markdown
# Plan [NUMBER]: [DESCRIPTIVE TITLE]

## Current State Analysis
**Purpose**: Understand what exists now and identify what needs to change

- ✅ **[Component/Feature that works]** - Brief description
- ❌ **[Component/Feature that's broken]** - What's wrong
- ⚠️ **[Component/Feature that needs improvement]** - What could be better
- [Document file locations and key line numbers for reference]

## Current Problem
**Purpose**: Clearly define the issues that need solving

- [Primary problem statement]
- [Secondary issues that relate to primary problem]
- [User-visible symptoms or errors]
- [Technical debt or architectural issues]
- [Performance or scalability concerns]

## Proposed Solution Structure
**Purpose**: Show the desired end state with concrete examples

### [Component A] Expected Structure:
```json/python/typescript
// Show exact structure/schema/interface expected
{
  "field": "value",
  "nested": {
    "structure": "example"
  }
}
```

### [Component B] Expected Structure:
```json/python/typescript
// Another concrete example of desired state
```

## Task 1: [High-Level Task Description]
**Purpose**: [Why this task is needed]

### 1.1 [Specific Sub-task]
**Location**: `path/to/file.ext` (lines X-Y)

**Current Code**:
```language
// Show existing code that needs to change
existing_function() {
  current_implementation
}
```

**New Code**:
```language
// Show exact replacement code
new_function() {
  updated_implementation
}
```

### 1.2 [Another Sub-task]
- Bullet points for simpler changes
- Reference specific line numbers
- Include any configuration updates

## Task 2: [Second High-Level Task]
**Purpose**: [Why this task is needed]

### 2.1 [Specific Implementation Step]
**Important**: Note any dependencies or order requirements

```language
// Code examples with clear before/after
```

### 2.2 [Configuration or Setup Changes]
- Update file: `config/settings.py`
- Add environment variable: `NEW_VAR=value`
- Install dependency: `npm install package-name`

## Task 3: [Integration and Testing]
**Purpose**: Ensure all changes work together

### 3.1 Manual Testing Steps
1. Start the backend server
2. Start the frontend development server
3. Navigate to [specific page/feature]
4. Perform [specific action]
5. Verify [expected result]

### 3.2 Automated Test Updates
- Update test file: `tests/test_feature.py`
- Add new test cases for [functionality]
- Ensure existing tests still pass

## Testing Checklist
**Purpose**: Verify implementation is complete and correct

- [ ] [Specific functionality works as expected]
- [ ] [API endpoint returns correct structure]
- [ ] [Frontend displays data properly]
- [ ] [No console errors in browser]
- [ ] [No errors in backend logs]
- [ ] [Performance is acceptable]
- [ ] [Edge cases handled properly]
- [ ] [Documentation updated if needed]

## Critical Implementation Rules
**Purpose**: Prevent common mistakes and ensure quality

1. **[Rule Name]** - Explanation of what to do/avoid
2. **Keep changes minimal** - Don't refactor unrelated code
3. **Test after each task** - Don't batch changes without testing
4. **Preserve working code** - Don't break existing functionality
5. **Follow existing patterns** - Match the codebase style

## Common Pitfalls to Avoid
**Purpose**: Learn from past mistakes

- **Don't** [specific anti-pattern with example]
- **Don't** create circular dependencies
- **Don't** use magic methods or overly clever code
- **Don't** change database schema without migration
- **Watch for** [specific gotcha in this codebase]
- **Remember** [important codebase-specific detail]

## Dependencies and Prerequisites
**Purpose**: Identify what needs to be in place before starting

- Required tools: [List any tools needed]
- Access needed: [Database, API keys, etc.]
- Related plans: [Reference other Phase_X_Implementation_Plan.md files if applicable]
- Documentation: [Links to relevant docs]

## Rollback Plan
**Purpose**: How to undo changes if something goes wrong

1. Git revert the commits: `git revert [commit-hash]`
2. Restore database: [Specific restore steps if applicable]
3. Clear caches: [Any cache clearing needed]
4. Restart services: [Which services need restarting]

## Success Metrics
**Purpose**: How to know the implementation is successful

- [ ] All tests pass
- [ ] Feature works in development environment
- [ ] No performance degradation
- [ ] Code review completed
- [ ] Documentation updated

## Notes and Context
**Purpose**: Additional information for implementers

- Historical context: [Why decisions were made]
- Related issues: [Links to tickets or discussions]
- Future considerations: [What might change later]
- Technical debt acknowledged: [What we're accepting for now]
```

### Step 5: Template Compliance Check

**Announce**: > **Step 5/6: Template Compliance Check** — Verifying all required sections and formatting...

1. **Verify all template sections are included**:
   - Current State Analysis with ✅❌⚠️ symbols
   - Current Problem with specific issues
   - Proposed Solution Structure with concrete examples
   - Tasks with exact file paths and line numbers
   - Before/after code examples for each change
   - All required sections from PLAN_TEMPLATE.md

2. **Ensure formatting matches PLAN_TEMPLATE.md exactly**:
   - Use exact section headers and structure
   - Include "Purpose" descriptions for each section
   - Follow the established formatting conventions

### Step 6: Final Review

**Announce**: > **Step 6/6: Final Review** — Presenting the plan for your review...

1. **Present all draft plan locations**:
   ```
   I've created the implementation plans at:
   - `project_docs/[git-branch-name]/Phase_1_Implementation_Plan.md` — [Phase 1 summary]
   - `project_docs/[git-branch-name]/Phase_2_Implementation_Plan.md` — [Phase 2 summary]
   - `project_docs/[git-branch-name]/Phase_3_Implementation_Plan.md` — [Phase 3 summary]
   (list all phases)
   ```

2. **Suggest next step** — after presenting the plan locations, end your response with a suggestion to review:
   ```
   When you're ready, run `/ct:review-plan` to review the plans for completeness, clarity, and any adjustments.
   ```
   - **Do NOT ask the user for feedback here.** The user hasn't had time to read the full plans yet. Adjustments are handled by the `/ct:review-plan` command.

## Important Guidelines

1. **Be Skeptical**:
   - Question vague requirements
   - Identify potential issues early
   - Ask "why" and "what about"
   - Don't assume - verify with code

2. **Be Interactive**:
   - Don't write the full plan in one shot
   - Get buy-in at each major step
   - Allow course corrections
   - Work collaboratively
   - **Always use the `AskUserQuestion` tool** for questions so the user can answer inline in the running prompt — never just output questions as text and end your response
   - Ask ONE question at a time in a sequential loop; never dump all questions at once

3. **Be Thorough**:
   - Read all context files COMPLETELY before planning
   - Research actual code patterns using parallel sub-tasks
   - Include specific file paths and line numbers
   - Write measurable success criteria with clear automated vs manual distinction

4. **Be Practical**:
   - Focus on incremental, testable changes
   - Consider migration and rollback
   - Think about edge cases
   - Include "what we're NOT doing"

5. **Follow PLAN_TEMPLATE.md Format**:
   - Use ✅❌⚠️ symbols in Current State Analysis
   - Include specific file paths and line numbers
   - Show concrete before/after code examples
   - Include all template sections even if brief
   - Focus on actionable, AI-executable instructions

6. **Track Progress**:
   - Use a single TaskCreate task per research phase with a visible `activeForm` spinner — do not create per-agent tasks (the agent UI already shows individual agent progress)
   - Mark the research task `in_progress` before spawning agents, `completed` only after ALL agents return and findings are summarized
   - Announce each step transition so the user always knows what's happening

7. **No Open Questions in Final Plan**:
   - If you encounter open questions during planning, STOP
   - Research or ask for clarification immediately
   - Do NOT write the plan with unresolved questions
   - The implementation plan must be complete and actionable
   - Every decision must be made before finalizing the plan

## Success Criteria Guidelines

**Always separate success criteria into two categories:**

1. **Automated Verification** (can be run by execution agents):
   - Commands that can be run: `make test`, `npm run lint`, etc.
   - Specific files that should exist
   - Code compilation/type checking
   - Automated test suites

2. **Manual Verification** (requires human testing):
   - UI/UX functionality
   - Performance under real conditions
   - Edge cases that are hard to automate
   - User acceptance criteria

**Format example:**
```markdown
### Success Criteria:

#### Automated Verification:
- [ ] Database migration runs successfully: `make migrate`
- [ ] All unit tests pass: `go test ./...`
- [ ] No linting errors: `golangci-lint run`
- [ ] API endpoint returns 200: `curl localhost:8080/api/new-endpoint`

#### Manual Verification:
- [ ] New feature appears correctly in the UI
- [ ] Performance is acceptable with 1000+ items
- [ ] Error messages are user-friendly
- [ ] Feature works correctly on mobile devices
```

## Common Patterns

### For Database Changes:
- Start with schema/migration
- Add store methods
- Update business logic
- Expose via API
- Update clients

### For New Features:
- Research existing patterns first
- Start with data model
- Build backend logic
- Add API endpoints
- Implement UI last

### For Refactoring:
- Document current behavior
- Plan incremental changes
- Maintain backwards compatibility
- Include migration strategy

## Sub-task Spawning Best Practices

When spawning research sub-tasks:

1. **Spawn multiple tasks in parallel** for efficiency
2. **Each task should be focused** on a specific area
3. **Provide detailed instructions** including:
   - Exactly what to search for
   - Which directories to focus on
   - What information to extract
   - Expected output format
4. **Be EXTREMELY specific about directories**:
   - Include the full path context in your prompts
5. **Specify read-only tools** to use
6. **Request specific file:line references** in responses
7. **Wait for all tasks to complete** before synthesizing
8. **Verify sub-task results**:
   - If a sub-task returns unexpected results, spawn follow-up tasks
   - Cross-check findings against the actual codebase
   - Don't accept results that seem incorrect

Example of spawning multiple tasks:
```python
# Spawn these tasks concurrently:
tasks = [
    Task("Research database schema", db_research_prompt),
    Task("Find API patterns", api_research_prompt),
    Task("Investigate UI components", ui_research_prompt),
    Task("Check test patterns", test_research_prompt)
]
```

## Example Interaction Flow

```
User: /implementation_plan
Assistant: I'll help you create a detailed implementation plan...

User: We need to add parent-child tracking for Claude sub-tasks. See tickets/eng_1478.md
Assistant: Let me read that ticket file completely first...

[Reads file fully]

Based on the ticket, I understand we need to track parent-child relationships for Claude sub-task events in the daemon. Before I start planning, I have some questions...

[Interactive process continues...]
```
