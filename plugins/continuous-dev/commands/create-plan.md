# Create Plan

You are tasked with creating detailed implementation plans through an interactive, iterative process. You should be skeptical, thorough, and work collaboratively with the user to produce high-quality technical specifications.

## Initial Response

When this command is invoked:

1. **Check if parameters were provided**:
   - If a file path or ticket reference was provided as a parameter, skip the default message
   - Immediately read any provided files FULLY
   - Begin the research process

2. **If no parameters provided**, respond with:
```
I'll help you create a detailed implementation plan. Let me start by understanding what we're building.

Please provide:
1. The task/ticket description (or reference to a ticket file)
2. Any relevant context, constraints, or specific requirements
3. Links to related research or previous implementations

I'll analyze this information and work with you to create a comprehensive plan.

Tip: You can also invoke this command with a ticket file directly: `/create_plan thoughts/allison/tickets/eng_1234.md`
For deeper analysis, try: `/create_plan think deeply about thoughts/allison/tickets/eng_1234.md`
```

Then wait for the user's input.

## Process Steps

### Step 1: Context Gathering & Initial Analysis

1. **Read all mentioned files immediately and FULLY**:
   - Ticket files (e.g., `thoughts/allison/tickets/eng_1234.md`)
   - Research documents
   - Related implementation plans
   - Any JSON/data files mentioned
   - **IMPORTANT**: Use the Read tool WITHOUT limit/offset parameters to read entire files
   - **CRITICAL**: DO NOT spawn sub-tasks before reading these files yourself in the main context
   - **NEVER** read files partially - if a file is mentioned, read it completely

2. **Spawn initial research tasks to gather context**:
   Before asking the user any questions, use specialized agents to research in parallel:

   - Use the **codebase-locator** agent to find all files related to the ticket/task
   - Use the **codebase-analyzer** agent to understand how the current implementation works
   - If relevant, use the **thoughts-locator** agent to find any existing thoughts documents about this feature
   - If a Linear ticket is mentioned, use the **linear-ticket-reader** agent to get full details

   These agents will:
   - Find relevant source files, configs, and tests
   - Trace data flow and key functions
   - Return detailed explanations with file:line references

3. **Read all files identified by research tasks**:
   - After research tasks complete, read ALL files they identified as relevant
   - Read them FULLY into the main context
   - This ensures you have complete understanding before proceeding

4. **Analyze and verify understanding**:
   - Cross-reference the ticket requirements with actual code
   - Identify any discrepancies or misunderstandings
   - Note assumptions that need verification
   - Determine true scope based on codebase reality

5. **Present informed understanding and focused questions**:
   ```
   Based on the ticket and my research of the codebase, I understand we need to [accurate summary].

   I've found that:
   - [Current implementation detail with file:line reference]
   - [Relevant pattern or constraint discovered]
   - [Potential complexity or edge case identified]

   Questions that my research couldn't answer:
   - [Specific technical question that requires human judgment]
   - [Business logic clarification]
   - [Design preference that affects implementation]
   ```

   Only ask questions that you genuinely cannot answer through code investigation.

### Step 2: Research & Discovery

After getting initial clarifications:

1. **If the user corrects any misunderstanding**:
   - DO NOT just accept the correction
   - Spawn new research tasks to verify the correct information
   - Read the specific files/directories they mention
   - Only proceed once you've verified the facts yourself

2. **Create a research todo list** using TodoWrite to track exploration tasks

3. **Spawn parallel sub-tasks for comprehensive research**:
   - Create multiple Task agents to research different aspects concurrently
   - Use the right agent for each type of research:

   **For deeper investigation:**
   - **codebase-locator** - To find more specific files (e.g., "find all files that handle [specific component]")
   - **codebase-analyzer** - To understand implementation details (e.g., "analyze how [system] works")
   - **codebase-pattern-finder** - To find similar features we can model after

   **For historical context:**
   - **thoughts-locator** - To find any research, plans, or decisions about this area
   - **thoughts-analyzer** - To extract key insights from the most relevant documents

   **For related tickets:**
   - **linear-searcher** - To find similar issues or past implementations

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

3. **Wait for ALL sub-tasks to complete** before proceeding

4. **Present findings and design options**:
   ```
   Based on my research, here's what I found:

   **Current State:**
   - [Key discovery about existing code]
   - [Pattern or convention to follow]

   **Design Options:**
   1. [Option A] - [pros/cons]
   2. [Option B] - [pros/cons]

   **Open Questions:**
   - [Technical uncertainty]
   - [Design decision needed]

   Which approach aligns best with your vision?
   ```

### Step 3: Plan Structure Development

Once aligned on approach:

1. **Create initial plan outline**:
   ```
   Here's my proposed plan structure:

   ## Overview
   [1-2 sentence summary]

   ## Implementation Phases:
   1. [Phase name] - [what it accomplishes]
   2. [Phase name] - [what it accomplishes]
   3. [Phase name] - [what it accomplishes]

   Does this phasing make sense? Should I adjust the order or granularity?
   ```

2. **Get feedback on structure** before writing details

### Step 4: Detailed Plan Writing

After structure approval:

1. **Write the plan** to `project_docs/[git-branch-name]/Phase_[num]_Implementation_Plan.md` where:
   - `[git-branch-name]` is the current git branch name
   - `[num]` is the phase number (1, 2, 3, etc.)
   - Create the directory if it doesn't exist
2. **Use the exact PLAN_TEMPLATE.md format**:

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

1. **Present the draft plan location**:
   ```
   I've created the implementation plan at:
   `project_docs/[git-branch-name]/Phase_[num]_Implementation_Plan.md`

   Please review it and let me know:
   - Are the tasks properly scoped with specific file changes?
   - Are the code examples concrete and actionable?
   - Do the Current State Analysis symbols (✅❌⚠️) accurately reflect reality?
   - Are the Critical Implementation Rules and Common Pitfalls specific to this task?
   - Missing edge cases or considerations?
   ```

3. **Iterate based on feedback** - be ready to:
   - Add missing phases
   - Adjust technical approach
   - Clarify success criteria (both automated and manual)
   - Add/remove scope items

4. **Continue refining** until the user is satisfied

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
   - Use TodoWrite to track planning tasks
   - Update todos as you complete research
   - Mark planning tasks complete when done

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

User: We need to add parent-child tracking for Claude sub-tasks. See thoughts/allison/tickets/eng_1478.md
Assistant: Let me read that ticket file completely first...

[Reads file fully]

Based on the ticket, I understand we need to track parent-child relationships for Claude sub-task events in the daemon. Before I start planning, I have some questions...

[Interactive process continues...]
```
