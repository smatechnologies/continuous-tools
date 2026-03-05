---
description: Create a detailed feature specification (with branch protection for develop/main)
argument-hint: [feature-description]
---

# Create Feature Specification

You are tasked with creating comprehensive feature specifications through an interactive, iterative process. You will gather requirements, clarify ambiguities, and produce a detailed specification document suitable for implementation planning.

## Mode-Aware Model Selection

**Before spawning any agent sub-tasks**, read `plugins/ct/config/mode.md` to determine the active mode.

Apply the following model assignment when using the Task tool:
- **Quality mode**: Use `model: opus` for Tier 1 and Tier 2 agents, `model: sonnet` for Tier 3 agents
- **Balanced mode**:
  - Use `model: opus` for **Tier 1** agents: `codebase-analyzer`, `code-review-expert`, `debug-investigator`
  - Use `model: sonnet` for **Tier 2** agents: `codebase-pattern-finder`, `web-search-researcher`, and all implementation/testing agents
  - Use `model: haiku` for **Tier 3** agents: `codebase-locator`

Always include the `model` parameter on every Task tool call based on the active mode and the agent's tier.

## Pre-Flight: Branch Safety Check (Step 0)

When this command is invoked, you MUST perform these checks **before doing anything else** (before research, before asking for a feature description, before any file I/O):

1. **Check if this is a git repository**:
   - Run `git rev-parse --is-inside-work-tree` via the Bash tool.
   - If it fails (not a git repo), skip all worktree logic and proceed directly to the spec creation process below.

2. **Check the current branch**:
   - Run `git branch --show-current` via the Bash tool.
   - If the current branch is **NOT** `develop` **and NOT** `main`, no worktree is needed. Announce: "Currently on branch `<branch-name>` — proceeding without creating a worktree." Then proceed to the spec creation process below.

3. **If the current branch IS `develop` or `main`**:
   - You MUST create a git worktree before making any changes.
   - **Get the feature identifier**: Use `AskUserQuestion` to ask the user:
     - Question: "What is the feature/ticket ID for this spec? (e.g., PROJ-123, FEAT-456)"
     - This ID will be used for the worktree folder name and branch name.
   - **Create the worktree**:
     - Use the `EnterWorktree` tool with `name` set to the feature ID provided by the user (e.g., `PROJ-123`).
     - This creates a worktree at `.claude/worktrees/<feature-id>/` on a new branch based on the current HEAD.
   - **Confirm to the user**:
     - Announce: "Created worktree and switched to branch `<branch-name>` to protect the `develop`/`main` branch. All spec work will happen here."

### Important Rules

- **NEVER** create or modify files while on the `develop` or `main` branch. The worktree step is mandatory when on those branches.
- If `EnterWorktree` fails for any reason, **stop and inform the user**. Do not fall back to working on `develop` or `main`.
- The feature ID should appear in both the worktree directory name and the branch name so it is easy to identify later.

## Initial Response

## Handling User Input

**CRITICAL**: When the user invokes this command, they may provide input in different ways:

1. **Direct description**: `/create-spec Add user authentication` - Use the text after the command as the feature description
2. **File reference**: `/create-spec @path/to/requirements.md` - The file contents will be expanded and provided. Use THIS CONTENT as the feature requirements, not any examples in this command file.
3. **No parameters**: Just `/create-spec` - Ask the user for input as described in the Initial Response section

**IMPORTANT**: NEVER use example text from this command file as the feature description. Always use the ACTUAL content provided by the user.

When a file is provided via `@`, the file contents replace the `@path` reference. Treat those expanded contents as the authoritative source for requirements.

---

When this command is invoked:

1. **Check if parameters were provided**:
   - If a description or file was provided as a parameter, use it as the starting point
   - Begin the interactive requirements gathering process

2. **If no parameters provided**, use the `AskUserQuestion` tool to prompt the user inline:
   - Call `AskUserQuestion` with a single question: "What feature do you want to build? Give a brief description, or paste in requirements. (Tip: you can also invoke with `/create-spec <description>` directly.)"
   - After receiving the description, proceed to Step 1 research — let research inform what follow-up questions are actually needed rather than front-loading generic questions.

## Progress Signaling

**CRITICAL**: This is a long-running, multi-step process. You MUST signal progress to the user at every stage so they know the command is actively working and not hung.

1. **Step announcements**: Before beginning each step, announce it:
   > **Step N/7: [Step Name]** — [Brief description of what's happening next]...

2. **Research phase tracking**: When spawning research agents (which can take 30-60+ seconds), create a **single** TaskCreate task to cover the entire research phase (e.g., activeForm: "Researching codebase and gathering context..."). Mark it `in_progress` before spawning agents. Do NOT mark it `completed` until ALL agents have returned and you have processed their results. The individual agent calls already show their own progress in the UI — do not duplicate per-agent tracking with separate TaskCreate tasks.

3. **Research summaries**: After research agents return, present a brief summary of findings before moving on, so the user sees concrete output from the wait.

## Process Steps

**CRITICAL**: This is an interactive, research-driven process. Use specialized agents to gather context from the codebase and external sources. Present all findings to the user and engage in dialog before proceeding to requirements gathering.

### Step 1: Initial Understanding & Context Gathering

**Announce**: > **Step 1/7: Initial Understanding & Context Gathering** — Parsing your description and researching the codebase for relevant context...

**IMPORTANT**: The feature to specify is determined ONLY by the user's input — not by any templates, examples, or placeholder text in this prompt.

1. **Parse the initial description** and identify:
   - Core functionality being requested
   - Implicit requirements or assumptions
   - Potential scope boundaries
   - Technical domain (frontend, backend, full-stack, infrastructure)

2. **RESEARCH PHASE - Critical for comprehensive specs**:

   **Before spawning research agents**, create a single TaskCreate task for the research phase:
   - `activeForm`: "Researching codebase and gathering context for [feature]..."
   - Mark it `in_progress` immediately so the user sees a spinner
   - Do NOT mark it `completed` until ALL research agents have returned and you have summarized findings

   **Prefer parallel execution**: When research agents are independent of each other, include all Task tool calls in a single message so they run concurrently. Only run agents sequentially when one agent's output is needed to inform the next agent's prompt.

   **CRITICAL ORDERING — codebase-locator MUST run before codebase-analyzer**:
   If both agents are needed, always spawn codebase-locator first, wait for its results, then spawn codebase-analyzer with the locator's findings included in the prompt. The web-search-researcher is independent and CAN run in parallel with codebase-locator.

   **a) Research external knowledge when needed:**
   - Use **web-search-researcher** agent when clarification or domain knowledge is critical
   - Research: industry standards, best practices, technical approaches, domain-specific requirements
   - Only when findings would significantly impact spec quality
   - Present research findings to user for discussion

   **b) Search codebase for context (run locator first, then analyzer if needed):**
   - **First**: Use **codebase-locator** to find related existing features
   - **Then** (after locator returns): If deeper analysis is needed, use **codebase-analyzer** to examine implementation details, providing it the file locations from the locator
   - Identify patterns and conventions that should be followed
   - Present codebase findings to user

   **c) Research summary checkpoint:**
   After all research agents return, present a consolidated summary:
   ```
   **Research Complete.** Here's what I found:
   - Codebase: [Key patterns, related features, conventions discovered]
   - External: [Relevant standards, best practices, domain insights]
   ```

   **IMPORTANT**: The research phase is interactive. Present findings incrementally and engage user in dialog:
   - "I found these relevant patterns in the codebase. Here's what they tell us..."
   - "I researched [topic] and found these industry standards. Should we align with them?"
   - Wait for user feedback before proceeding to next research area

3. **Present initial understanding, then enter sequential questioning loop**:
   - First, output your research summary:
     ```
     Based on your description and my research, I understand you want to build [summary of feature].

     Here's what I found in the codebase:
     - [Relevant existing patterns or features]
     - [Technical context that informs the spec]
     ```
   - Then, generate (internally) a prioritized list of questions your research couldn't answer. Only include questions that genuinely require human judgment. Common areas (ask only what's needed, not all):
     - **User Types**: Who are the primary users? Different roles?
     - **Key Problem**: What specific problem does this solve?
     - **Success Metrics**: How will we know this is successful?
     - **Scope Boundaries**: What should this NOT do?
     - **Technical Constraints**: Systems to integrate with? Performance needs?
   - **Sequential questioning loop**: Present EXACTLY ONE question at a time using the `AskUserQuestion` tool. For each question:
     - If there are clear discrete options, provide them as `AskUserQuestion` options (2-4 choices)
     - After the user answers, record the answer and proceed to the next question
     - Each subsequent question may be informed by prior answers
     - Stop asking when: all critical areas are covered, OR the user signals completion
   - **CRITICAL**: Use the `AskUserQuestion` tool for each question so the user answers inline in the running prompt. Do NOT output questions as plain text and end your response.

### Step 2: Requirements Discovery

**Announce**: > **Step 2/7: Requirements Discovery** — Diving deeper based on your answers so far...

Continue the sequential questioning loop from Step 1, now digging into requirements. Use the `AskUserQuestion` tool for each question — one at a time, inline.

**Question sequence** (ask only what's needed based on prior answers):

1. **User Experience** (ask first, since it shapes everything else):
   - Use `AskUserQuestion` to ask about the user journey: what triggers use, step-by-step flow, error scenarios
   - Wait for answer, then proceed to next area

2. **Technical Requirements** (informed by UX answers):
   - Use `AskUserQuestion` to ask about data persistence, real-time needs, security, scale, integrations — only the ones genuinely unclear after research
   - Wait for answer, then proceed

3. **Edge Cases** (now that you have full context):
   - Propose the edge cases YOU identified from research and prior answers
   - Use `AskUserQuestion` to confirm: present your proposed list as options, with an "Other" path for additions
   - Frame as: "Here are the edge cases I think we need to handle. Anything I'm missing?"

**CRITICAL**: Each topic is its own `AskUserQuestion` call. Do NOT present multiple topics in a single question. Do NOT output questions as plain text and end your response.

### Step 3: User Story Development

**Announce**: > **Step 3/7: User Story Development** — Generating user stories from gathered requirements...

1. **Generate initial user stories** based on gathered requirements:
   ```
   Based on our discussion, here are the key user stories I've identified:

   **Epic: [Feature Name]**

   1. **As a [user type], I want to [action] so that [benefit]**
      - Priority: [High/Medium/Low]
      - Complexity: [Small/Medium/Large]

   2. **As a [user type], I want to [action] so that [benefit]**
      - Priority: [High/Medium/Low]
      - Complexity: [Small/Medium/Large]

   ```

2. **Get feedback using `AskUserQuestion`** — immediately after outputting the stories, call `AskUserQuestion` inline:
   - Question: "How do these user stories look?"
   - Options: "Approve", "Needs adjustments"
   - Iterate until the user approves. **Do NOT output questions as plain text and end your response.**

### Step 4: Acceptance Criteria Development

**Announce**: > **Step 4/7: Acceptance Criteria Development** — Defining testable acceptance criteria for each user story...

For each user story, develop detailed acceptance criteria:

```
Let me develop acceptance criteria for each story. I'll start with the highest priority:

**Story 1: [User Story Title]**

**Acceptance Criteria:**
✓ Given [context], when [action], then [expected result]
✓ Given [context], when [action], then [expected result]
✓ System validates [specific validation]
✓ Error message shows when [error condition]
✓ Performance: [Action] completes in less than [time]

**Test Scenarios:**
- Happy path: [Description]
- Error case: [Description]
- Edge case: [Description]

```

   Then call `AskUserQuestion` inline to confirm:
   - Question: "Does this capture the expected behavior?"
   - Options: "Approve", "Needs adjustments"
   - Iterate until approved. **Do NOT output questions as plain text and end your response.**

### Step 5: Functional Requirements Documentation

**Announce**: > **Step 5/7: Functional Requirements Documentation** — Organizing requirements by category and confirming technical approach...

1. **Organize requirements by category**:
   - Data requirements
   - Business logic
   - UI/UX requirements
   - API requirements
   - Security requirements
   - Performance requirements
   - Integration requirements

2. **Get confirmation on technical approach** — output the proposed approach:
   ```
   Based on the requirements, here's the technical approach I'm considering:

   **Data Model:**
   - [Entity 1]: [key fields and relationships]
   - [Entity 2]: [key fields and relationships]

   **API Design:**
   - [Endpoint 1]: [method, path, purpose]
   - [Endpoint 2]: [method, path, purpose]

   **Key Components:**
   - [Component 1]: [responsibility]
   - [Component 2]: [responsibility]
   ```

   Then call `AskUserQuestion` inline to confirm:
   - Question: "Does this technical approach align with your vision?"
   - Options: "Approve", "Needs adjustments"
   - Iterate until approved. **Do NOT output questions as plain text and end your response.**

### Step 6: Create Feature Specification Document

**Announce**: > **Step 6/7: Creating Feature Specification Document** — Writing the full specification to disk...

1. **Determine the feature directory from current git branch**:
   - Run `git branch --show-current` to get the current branch name (e.g., `007-extraction-enhancements`)
   - The feature directory is `/project_docs/[branch-name]/`
   - Extract the feature name from the branch (remove leading numbers and hyphens)
   - Save as: `/project_docs/[branch-name]/[feature-name]-spec.md`
   - Example: For branch `007-extraction-enhancements`, save to `/project_docs/007-extraction-enhancements/extraction-enhancements-spec.md`
   - Create the directory if it doesn't exist

2. **Write the specification** using this template:

```markdown
# Feature Specification: [Feature Name]

**Document**: /project_docs/[branch-name]/[feature-name]-spec.md
**Created**: [Date]
**Status**: Draft | In Review | Approved
**Target Release**: [Version/Sprint]

## Executive Summary

**Purpose**: [One paragraph explaining why this feature exists]

**Key Benefits**:
- [Benefit 1]
- [Benefit 2]
- [Benefit 3]

**Success Metrics**:
- [Measurable outcome 1]
- [Measurable outcome 2]

## Problem Statement

### Current State
[Describe the current situation and its limitations]

### Problems to Solve
1. **[Problem 1]**: [Description and impact]
2. **[Problem 2]**: [Description and impact]
3. **[Problem 3]**: [Description and impact]

### Proposed Solution
[High-level description of how this feature addresses the problems]

## User Personas

### Primary Users
**[Persona Name]**
- Role: [Description]
- Goals: [What they want to achieve]
- Pain Points: [Current frustrations]
- Technical Skill: [Low/Medium/High]

### Secondary Users
**[Persona Name]**
- Role: [Description]
- Goals: [What they want to achieve]
- Pain Points: [Current frustrations]
- Technical Skill: [Low/Medium/High]

## User Stories

### Epic: [Epic Name]

#### Story 1: [Story Title]
**As a** [user type]
**I want to** [action/feature]
**So that** [benefit/value]

**Priority**: High | Medium | Low
**Effort**: Small (1-2 days) | Medium (3-5 days) | Large (1-2 weeks)

**Acceptance Criteria**:
- [ ] Given [context], when [action], then [expected result]
- [ ] Given [context], when [action], then [expected result]
- [ ] System validates [validation rule]
- [ ] Error handling: [error scenario and expected behavior]
- [ ] Performance: [specific performance requirement]

**Test Scenarios**:
1. **Happy Path**: [Description]
2. **Error Case**: [Description]
3. **Edge Case**: [Description]

#### Story 2: [Story Title]
[Repeat structure for each story]

## Functional Requirements

### Data Requirements

#### [Entity Name]
**Purpose**: [Why this data exists]

**Fields**:
| Field | Type | Required | Description | Validation |
|-------|------|----------|-------------|------------|
| id | UUID | Yes | Unique identifier | Auto-generated |
| [field] | [type] | [Yes/No] | [Description] | [Rules] |

**Relationships**:
- [Relationship description]

### Business Logic Requirements

#### [Feature Area]
1. **[Rule Name]**: [Description of business rule]
   - Condition: [When this applies]
   - Action: [What happens]
   - Exception: [Special cases]

2. **[Rule Name]**: [Description of business rule]
   - Condition: [When this applies]
   - Action: [What happens]
   - Exception: [Special cases]

### API Requirements

#### [Endpoint Group]

**[Method] /api/[path]**
- **Purpose**: [What this endpoint does]
- **Authentication**: Required | Optional | None
- **Rate Limiting**: [Limits if applicable]

**Request**:
```json
{
  "field1": "type and description",
  "field2": "type and description"
}
```

**Response (Success - 200)**:
```json
{
  "field1": "type and description",
  "field2": "type and description"
}
```

**Response (Error - 4XX)**:
```json
{
  "error": "Error message",
  "code": "ERROR_CODE"
}
```

### UI/UX Requirements

#### [Screen/Component Name]

**Purpose**: [What this UI element does]

**Layout Requirements**:
- [Requirement 1]
- [Requirement 2]

**Interaction Requirements**:
- [Interaction pattern 1]
- [Interaction pattern 2]

**Responsive Behavior**:
- Desktop: [Behavior]
- Tablet: [Behavior]
- Mobile: [Behavior]

**Accessibility Requirements**:
- Keyboard navigation: [Requirements]
- Screen reader: [Requirements]
- WCAG compliance: [Level A/AA/AAA]

### Security Requirements

1. **Authentication**: [Requirements]
2. **Authorization**: [Role-based access rules]
3. **Data Protection**: [Encryption, PII handling]
4. **Audit Logging**: [What needs to be logged]
5. **Rate Limiting**: [Limits and throttling rules]

### Performance Requirements

1. **Response Time**:
   - API calls: < [X] ms at p95
   - Page load: < [X] seconds

2. **Throughput**:
   - Expected: [X] requests/second
   - Peak: [X] requests/second

3. **Data Volume**:
   - Expected records: [X]
   - Growth rate: [X] per [time period]

## Non-Functional Requirements

### Scalability
- [Requirement 1]
- [Requirement 2]

### Reliability
- Uptime requirement: [X]%
- Error rate threshold: < [X]%

### Maintainability
- Code coverage requirement: [X]%
- Documentation requirements: [List]

### Compatibility
- Browser support: [List]
- Device support: [List]
- API version support: [Requirements]

## Technical Architecture

### System Components
```
[ASCII or text diagram showing component relationships]
```

### Data Flow
```
[ASCII or text diagram showing data flow]
```

### Integration Points
1. **[System Name]**: [How this feature integrates]
2. **[System Name]**: [How this feature integrates]

## Implementation Constraints

### Technical Constraints
- [Constraint 1]: [Description and impact]
- [Constraint 2]: [Description and impact]

### Business Constraints
- Timeline: [Deadline or timeframe]
- Budget: [If applicable]
- Resources: [Team or skill constraints]

### Regulatory/Compliance
- [Requirement 1]: [Description]
- [Requirement 2]: [Description]

## Risks and Mitigations

| Risk | Probability | Impact | Mitigation |
|------|------------|---------|------------|
| [Risk description] | Low/Med/High | Low/Med/High | [Mitigation strategy] |

## Dependencies

### Internal Dependencies
- [System/Feature]: [What's needed and when]
- [System/Feature]: [What's needed and when]

### External Dependencies
- [Service/API]: [What's needed and when]
- [Library/Tool]: [Version and requirements]

## Migration Plan (if applicable)

### Data Migration
- [Migration requirement 1]
- [Migration requirement 2]

### Feature Toggle Strategy
- [How feature will be rolled out]
- [Rollback plan]

## Monitoring and Analytics

### Key Metrics to Track
1. **[Metric Name]**: [How to measure and target]
2. **[Metric Name]**: [How to measure and target]

### Logging Requirements
- [What to log]
- [Log retention policy]

### Alerts
- [Alert condition 1]: [Threshold and action]
- [Alert condition 2]: [Threshold and action]

## Open Questions

1. ❓ **[Question]**: [Context and what needs to be decided]
2. ❓ **[Question]**: [Context and what needs to be decided]

## Future Enhancements (Out of Scope)

1. **[Enhancement]**: [Brief description and why it's deferred]
2. **[Enhancement]**: [Brief description and why it's deferred]

## Appendix

### Glossary
- **[Term]**: [Definition]
- **[Term]**: [Definition]

### References
- [Document/Link]: [Description]
- [Document/Link]: [Description]

### Related Documents
- Implementation Plan: Will be created in same feature directory
- Design Documents: [Links]
- Research: [Links]
```

### Step 7: Review and Iteration

**Announce**: > **Step 7/7: Review and Iteration** — Presenting the specification for your review...

1. **Present the specification location**:
   ```
   I've created the feature specification at:
   `/project_docs/[branch-name]/[feature-name]-spec.md`

   This specification is designed to provide everything needed for the `/create_plan` command to build a comprehensive implementation plan.
   ```

2. **Ask for review feedback using `AskUserQuestion`** — immediately after presenting the location, call `AskUserQuestion` inline:
   - Question: "How does the specification look?"
   - Options: "Approve — ready for planning", "Needs changes"
   - If the user selects a change option, iterate and ask again until approved.
   - **Do NOT output review questions as plain text and end your response.**

3. **Iterate based on feedback** until the specification is complete

## Important Guidelines

1. **Leverage Research Agents Proactively**:
   - Use **web-search-researcher** when domain knowledge or clarification would improve spec quality
   - Use **codebase-locator** to find existing patterns and features
   - Present ALL findings to user - this builds shared understanding
   - Engage user in dialog about findings before proceeding

2. **Be Thorough in Discovery**:
   - Don't accept vague requirements
   - Ask "what if" questions
   - Identify hidden assumptions
   - Consider error cases early
   - Use research findings to uncover implicit requirements
   - **Always use the `AskUserQuestion` tool** for questions so the user can answer inline in the running prompt — never just output questions as text and end your response
   - Ask ONE question at a time in a sequential loop; never dump all questions at once

3. **Write for Implementation**:
   - Every requirement should be testable
   - Include specific validation rules
   - Provide concrete examples
   - Define clear boundaries

4. **Structure for Clarity**:
   - Organize requirements logically
   - Use consistent formatting
   - Include diagrams where helpful
   - Cross-reference related sections

5. **Focus on User Value**:
   - Start with user problems
   - Connect features to benefits
   - Prioritize based on impact
   - Consider the full user journey

6. **Make it Actionable**:
   - Requirements should be specific enough to implement
   - Include all technical details needed
   - Define clear success criteria
   - Identify all dependencies

7. **Plan for Reality**:
   - Include error handling
   - Consider performance early
   - Plan for scale
   - Document security requirements

## Success Criteria for a Good Specification

A complete feature specification should:

✅ **Have clear user stories** with acceptance criteria
✅ **Define all data structures** with field specifications
✅ **Specify all API contracts** with request/response examples
✅ **Include UI/UX requirements** with interaction patterns
✅ **Document business logic** as testable rules
✅ **Address security and performance** requirements
✅ **Identify all dependencies** and integration points
✅ **Include test scenarios** for each user story
✅ **Have measurable success metrics**
✅ **Be specific enough** for AI to create an implementation plan

## Common Patterns by Feature Type

### For CRUD Features:
1. Define data model first
2. Specify validation rules
3. Document API endpoints
4. Define UI forms and lists
5. Include search/filter requirements
6. Specify permissions per operation

### For Workflow Features:
1. Map complete state machine
2. Define transition rules
3. Specify notifications/triggers
4. Document rollback scenarios
5. Include approval processes
6. Define timeout behaviors

### For Integration Features:
1. Document external API contracts
2. Specify error handling
3. Define retry logic
4. Include circuit breaker requirements
5. Document data transformation
6. Specify monitoring needs

### For Reporting Features:
1. Define data sources
2. Specify aggregation rules
3. Document filter options
4. Include export formats
5. Define refresh/cache strategy
6. Specify performance targets

## Reminder: Handling User Input

**IMPORTANT**: The feature to specify comes ONLY from the user's input — not from any templates, examples, or placeholder text in this prompt. See the "Handling User Input" section at the top of this file.