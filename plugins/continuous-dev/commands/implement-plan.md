# Implement Plan

You are tasked with implementing an approved technical plan from the referenced file. These plans contain phases with specific changes and success criteria.

## Getting Started

When given a plan path:
- Read the plan completely and check for any existing checkmarks (- [x])
- **Analyze scope and present implementation options** - don't assume one approach
- If no plan path provided, ask for one

## Scope Analysis & Implementation Strategy

**First, analyze the plan to determine appropriate implementation approach:**

1. **Plan Assessment**:
   - Count estimated lines of code changes across all files
   - Identify number of files affected and their types (.py, .ts, .tsx, etc.)
   - Assess complexity: Simple edits vs new features vs architectural changes
   - Note technology stack involved (FastAPI, Next.js, database, etc.)

2. **Present Options to User**:
   ```
   This appears to be a [SMALL/MEDIUM/LARGE] change:
   - Estimated lines: ~X
   - Files affected: Y ([file types])
   - Complexity: [Brief description]
   - Technology: [Main stack components]
   
   Implementation Options:
   A) Quick implementation (minimal validation, direct edits)
   B) Delegate to appropriate specialized agents  
   C) Full structured process with comprehensive validation
   
   Recommended: [A/B/C] ([Brief reason])
   
   How would you like me to proceed?
   ```

3. **Route Based on User Choice**:
   - **Option A**: Proceed with streamlined process below
   - **Option B**: Use Task tool to delegate to appropriate agent(s)
   - **Option C**: Use full comprehensive process (see Phase 0-3 sections)

## Option A: Streamlined Implementation

**For small changes (≤30 lines, 1-3 files, simple edits):**

1. **Quick Validation**:
   - Check git status for uncommitted changes
   - Verify mentioned files exist and code snippets match reality
   - Run existing tests once to establish baseline

2. **Direct Implementation**:
   - Create simple TodoWrite list from plan tasks
   - Make changes incrementally, one logical unit at a time
   - Test after each file modification
   - Update plan checkboxes as you complete sections

3. **Verification**:
   - Run linters if available (`npm run lint`, etc.)
   - Run tests for affected areas
   - Check for console errors/warnings

## Option B: Agent Delegation

**Route to specialized agents based on technology:**

- **Python/FastAPI/Backend**: Use `python-pro-backend` agent
- **Next.js/React/Frontend**: Use `nextjs-fullstack-expert` agent  
- **Mixed stack**: Delegate phases to appropriate agents
- **Database/Infrastructure**: Use `python-pro-backend` for data layer changes

**Delegation format**:
```
Task: [Agent Name] - Implement [specific section] from plan
Context: [Brief context about the change and why it's needed]
Plan Section: [Relevant plan section to implement]
Files Involved: [List of files from plan]
Success Criteria: [What constitutes successful completion]
```

## Option C: Full Comprehensive Process

### Phase 0: Pre-Flight Validation

**Before any code changes**, perform these critical checks:

1. **Plan Assessment**:
   - Read the plan completely and understand all tasks
   - Identify any existing checkmarks (- [x]) to resume from correct point
   - Extract all file paths and line numbers mentioned
   - Create a comprehensive TodoWrite list from the plan's tasks

2. **Environment Validation**:
   ```bash
   # Check git status - ensure clean working directory
   git status
   # If uncommitted changes exist, ask user how to proceed
   ```

3. **Baseline Testing**:
   - Run existing test suite to establish green baseline
   - Note any pre-existing failures (don't fix them unless plan requires)
   - Record current state for rollback reference

4. **Dependency Check**:
   - Verify all files mentioned in plan exist
   - Check that imports/packages referenced are available
   - Validate database connectivity if plan involves DB changes

### Phase 1: Plan Currency Validation

**Critical**: Plans can become stale. Validate each planned change:

1. **File Reality Check**:
   - For each file:line reference in plan, read the actual file
   - Compare plan's "Current Code" snippets with reality
   - If ANY mismatch found, STOP and report:
     ```
     Plan Drift Detected in [filename]:
     Plan Expected (line X): [code from plan]
     Actual Found: [actual code]
     
     Plan may be outdated. How should I proceed?
     ```

2. **Architecture Verification**:
   - Confirm the codebase structure matches plan assumptions
   - Check that planned integration points still exist
   - Verify dependencies haven't changed significantly

### Phase 2: Incremental Implementation

**Never implement entire phases at once.** Break down into atomic changes:

1. **Task Decomposition**:
   - For each plan task, create specific TodoWrite items
   - Each todo should be a single, testable change
   - Order tasks by dependency (database → backend → API → frontend)
   - Mark each todo as in_progress when starting

2. **Atomic Change Process**:
   ```
   For each atomic change:
   1. Make the minimal code change
   2. Save file and check syntax (no runtime errors)
   3. Run targeted tests if available
   4. If tests fail, analyze and fix immediately
   5. Only proceed to next change if green
   6. Mark todo as completed
   ```

3. **Change Validation**:
   - After each file edit, verify syntax is valid
   - Run linters/formatters if project has them
   - Check imports resolve correctly
   - Verify no obvious runtime errors

### Phase 3: Progressive Verification

**Verify continuously, not just at the end:**

1. **After Each Task Group** (every 3-5 atomic changes):
   - Run relevant test subset
   - Check for console errors/warnings
   - Verify no regressions in working functionality
   - Update plan checkboxes using Edit tool

2. **After Each Plan Task**:
   - Run full test suite for affected areas
   - Test the specific functionality being implemented
   - Check integration points work correctly
   - Validate success criteria from plan

3. **Rollback Strategy**:
   ```bash
   # If tests fail after a change:
   git diff HEAD~1  # See what changed
   git checkout -- [failing-file]  # Rollback if needed
   # Then retry with smaller change
   ```

## Guidelines for All Approaches

**Universal principles regardless of approach chosen:**
- Always update plan checkboxes as you complete sections
- Stop and ask if you encounter unexpected situations or plan drift
- Make changes incrementally with verification between steps
- Focus on the plan's intent, adapting to current reality as needed

When things don't match the plan exactly, think about why and communicate clearly:

```
Issue in Phase [N]:
Expected: [what the plan says]
Found: [actual situation]  
Why this matters: [explanation]
Impact: [how this affects other changes]

Options:
A) [suggested approach 1]
B) [suggested approach 2]

How should I proceed?
```

## Structured Task Management

**Always use TodoWrite for task tracking:**

1. **Initial Task Creation**:
   - Parse plan into specific, actionable todos
   - Include file paths and brief descriptions
   - Order by dependencies (infrastructure → business logic → UI)
   - Example format: `"Update UserService.getOrganizations() in backend/app/services/user.py:45-60"`

2. **Task Status Management**:
   - Mark exactly ONE task as `in_progress` at a time
   - Complete tasks immediately after finishing
   - Never batch completions - update status in real-time
   - If task fails, keep as `in_progress` and create sub-tasks for blockers

3. **Progress Tracking**:
   - Update TodoWrite after every change
   - Update plan checkboxes using Edit tool 
   - Keep both in sync to maintain clear progress visibility

## Error Prevention Strategies

**Common causes of implementation errors and how to avoid them:**

1. **Plan Drift** (Plan doesn't match current code):
   - Always validate "Current Code" sections before changing
   - Check file paths and line numbers are still accurate
   - Read entire functions, not just the lines mentioned

2. **Syntax Errors**:
   - Save and syntax-check after every edit
   - Use linters if available (`npm run lint`, `pylint`, etc.)
   - Test imports resolve before proceeding

3. **Test Failures**:
   - Run tests after each logical change, not just at the end
   - Understand WHY tests fail before fixing
   - Don't ignore "unrelated" test failures - they might be related

4. **Missing Dependencies**:
   - Check imports/packages exist before using them
   - Verify database schema matches assumptions
   - Test API endpoints respond before building on them

## If You Get Stuck

**When something isn't working as expected:**

1. **Immediate Actions**:
   - Stop making changes immediately
   - Run `git status` to see what you've changed
   - Run tests to understand current failure state

2. **Root Cause Analysis**:
   - Compare your changes with plan expectations
   - Check if codebase has evolved since plan was written
   - Verify all dependencies and imports are correct

3. **Escalation Format**:
   ```
   Stuck on: [specific task]
   What I tried: [exact changes made]
   Current error: [exact error message]
   Files affected: [list of changed files]
   
   I think the issue is: [your analysis]
   Need help with: [specific question]
   ```

4. **Use sub-tasks sparingly** - mainly for targeted debugging or exploring unfamiliar territory

## Resuming Work

**If the plan has existing checkmarks:**

1. **Trust Completed Work**: 
   - Don't re-verify completed checkboxes unless something seems broken
   - Focus on first unchecked item in plan

2. **Validate Context**:
   - Read the completed sections to understand current state
   - Run tests to ensure previous work still functions
   - Check if any dependencies have changed since last work

3. **Resume Process**:
   - Create new TodoWrite list for remaining tasks
   - Follow same validation phases for uncompleted work
   - Maintain same incremental approach

**Remember**: You're implementing a solution, not just checking boxes. The goal is working software that meets the plan's objectives with minimal errors and iterations.
