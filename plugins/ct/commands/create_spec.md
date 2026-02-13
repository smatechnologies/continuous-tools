---
description: Create a detailed feature specification
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

## Initial Response

## Handling User Input

**CRITICAL**: When the user invokes this command, they may provide input in different ways:

1. **Direct description**: `/create_spec Add user authentication` - Use the text after the command as the feature description
2. **File reference**: `/create_spec @path/to/requirements.md` - The file contents will be expanded and provided. Use THIS CONTENT as the feature requirements, not any examples in this command file.
3. **No parameters**: Just `/create_spec` - Ask the user for input as described in the Initial Response section

**IMPORTANT**: NEVER use example text from this command file as the feature description. Always use the ACTUAL content provided by the user.

When a file is provided via `@`, the file contents replace the `@path` reference. Treat those expanded contents as the authoritative source for requirements.

---

When this command is invoked:

1. **Check if parameters were provided**:
   - If a description or file was provided as a parameter, use it as the starting point
   - Begin the interactive requirements gathering process

2. **If no parameters provided**, respond with:
```
I'll help you create a comprehensive feature specification that will drive implementation planning.

Please provide:
1. A brief description of the feature you want to build
2. The primary problem this feature solves
3. Who the target users are

You can also provide a longer description or paste requirements if you have them.

Tip: You can invoke this command with a description directly: `/create_spec <your feature description here>`
```

Then wait for the user's input.

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

3. **Present initial understanding and gather core information**:
   ```
   Based on your description, I understand you want to build [summary of feature].
   
   Let me gather some key information to create a comprehensive specification:
   
   **Core Questions:**
   1. **User Types**: Who are the primary users? Are there different user roles?
   2. **Key Problem**: What specific problem does this solve? What's the current pain point?
   3. **Success Metrics**: How will we know this feature is successful?
   4. **Scope Boundaries**: What should this feature NOT do? Any explicit non-goals?
   5. **Technical Constraints**: Any existing systems to integrate with? Performance requirements?
   ```

### Step 2: Requirements Discovery

**Announce**: > **Step 2/7: Requirements Discovery** — Diving deeper into user journeys, technical constraints, and edge cases...

Based on initial answers, dig deeper with targeted questions:

1. **For User Experience**:
   ```
   **User Journey Questions:**
   - What triggers a user to use this feature?
   - What's the step-by-step flow from start to finish?
   - What happens if something goes wrong?
   - Are there different paths for different user types?
   ```

2. **For Technical Requirements**:
   ```
   **Technical Clarifications:**
   - Data persistence: What needs to be saved/stored?
   - Real-time requirements: Any live updates needed?
   - Security: What access controls are required?
   - Scale: Expected usage volume?
   - Integration: What systems does this touch?
   ```

3. **For Edge Cases**:
   ```
   **Edge Cases to Consider:**
   - What happens with invalid/malformed input?
   - How should the system behave under high load?
   - What if dependent services are unavailable?
   - How do we handle concurrent operations?
   ```

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
   
   Are there any user stories I'm missing? Should we adjust priorities?
   ```

2. **Refine stories based on feedback**

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

Does this capture the expected behavior correctly?
```

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

2. **Get confirmation on technical approach**:
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
   
   Does this align with your technical vision?
   ```

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

1. **Present the specification for review**:
   ```
   I've created the feature specification at:
   `/project_docs/[branch-name]/[feature-name]-spec.md`

   Please review and let me know:
   - Are all user stories captured with clear acceptance criteria?
   - Are the functional requirements specific enough for implementation?
   - Any missing edge cases or requirements?
   - Do the technical constraints and architecture align with your vision?

   This specification is designed to provide everything needed for the `/create_plan` command to build a comprehensive implementation plan.
   ```

2. **Iterate based on feedback** until the specification is complete

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