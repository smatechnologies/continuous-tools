---
description: Research the codebase and generate a report
argument-hint: [research topic or @context-file-path]
---

# Research Codebase

You are tasked with conducting comprehensive research across the codebase to answer user questions by spawning parallel sub-agents and synthesizing their findings.

## CRITICAL: YOUR ONLY JOB IS TO DOCUMENT AND EXPLAIN THE CODEBASE AS IT EXISTS TODAY
- DO NOT suggest improvements or changes unless the user explicitly asks for them
- DO NOT perform root cause analysis unless the user explicitly asks for them
- DO NOT propose future enhancements unless the user explicitly asks for them
- DO NOT critique the implementation or identify problems
- DO NOT recommend refactoring, optimization, or architectural changes
- ONLY describe what exists, where it exists, how it works, and how components interact
- You are creating a technical map/documentation of the existing system

## Mode-Aware Model Selection

**Before spawning any agent sub-tasks**, read `plugins/ct/config/mode.md` to determine the active mode.

Apply the following model assignment when using the Task tool:
- **Quality mode**: Use `model: opus` for Tier 1 and Tier 2 agents, `model: sonnet` for Tier 3 agents
- **Balanced mode**:
  - Use `model: opus` for **Tier 1** agents: `codebase-analyzer`, `code-review-expert`, `debug-investigator`
  - Use `model: sonnet` for **Tier 2** agents: `codebase-pattern-finder`, `web-search-researcher`, and all implementation/testing agents
  - Use `model: haiku` for **Tier 3** agents: `codebase-locator`

Always include the `model` parameter on every Task tool call based on the active mode and the agent's tier.

## Handling User Input

**CRITICAL**: When the user invokes this command, they may provide input in different ways:

1. **Direct question**: `/research-codebase How does authentication work?` - Use the text after the command as the research query
2. **File reference**: `/research-codebase @path/to/context.md` - The file contents will be expanded and provided. Use THIS CONTENT as context for the research, not any examples in this command file.
3. **No parameters**: Just `/research-codebase` - Ask the user for input as described in the Initial Response section

**IMPORTANT**: NEVER use example text from this command file as the research query. Always use the ACTUAL content provided by the user.

When a file is provided via `@`, the file contents replace the `@path` reference. Treat those expanded contents as the authoritative source for research context.

## Initial Response

When this command is invoked:

1. **Check if parameters were provided**:
   - If a question or file was provided as a parameter, use it as the research query
   - Begin the research process immediately

2. **If no parameters provided**, respond with:
```
I'm ready to research the codebase. Please provide your research question or area of interest, and I'll analyze it thoroughly by exploring relevant components and connections.

Tip: You can invoke this command with a query directly: `/research-codebase How does the authentication system work?`
```

Then wait for the user's research query.

## Progress Signaling

**CRITICAL**: This is a long-running, multi-step process. You MUST signal progress to the user at every stage so they know the command is actively working and not hung.

1. **Step announcements**: Before beginning each step, announce it:
   > **Step N/8: [Step Name]** — [Brief description of what's happening next]...

2. **Research phase tracking**: When spawning research agents (which can take 30-60+ seconds), create a **single** TaskCreate task to cover the entire research phase (e.g., activeForm: "Researching codebase for [topic]..."). Mark it `in_progress` before spawning agents. Do NOT mark it `completed` until ALL agents have returned and you have processed their results. The individual agent calls already show their own progress in the UI — do not duplicate per-agent tracking with separate TaskCreate tasks.

3. **Research summaries**: After research agents return, present a brief summary of findings before moving on, so the user sees concrete output from the wait.

## Steps to follow after receiving the research query:

1. **Read any directly mentioned files first:**

   **Announce**: > **Step 1/8: Reading Referenced Files** — Reading any files you mentioned for full context...

   - If the user mentions specific files (tickets, docs, JSON), read them FULLY first
   - **IMPORTANT**: Use the Read tool WITHOUT limit/offset parameters to read entire files
   - **CRITICAL**: Read these files yourself in the main context before spawning any sub-tasks
   - This ensures you have full context before decomposing the research

2. **Analyze and decompose the research question:**

   **Announce**: > **Step 2/8: Decomposing Research Question** — Breaking down the query into focused research areas...

   - Break down the user's query into composable research areas
   - Take time to think deeply about the underlying patterns, connections, and architectural implications the user might be seeking
   - Identify specific components, patterns, or concepts to investigate
   - Consider which directories, files, or architectural patterns are relevant

3. **Spawn parallel sub-agent tasks for comprehensive research:**

   **Announce**: > **Step 3/8: Researching Codebase** — Spawning research agents to explore the codebase...

   **Before spawning research agents**, create a single TaskCreate task for the research phase:
   - `activeForm`: "Researching codebase for [topic]..."
   - Mark it `in_progress` immediately so the user sees a spinner
   - Do NOT mark it `completed` until ALL research agents have returned and you have summarized findings

   **Prefer parallel execution**: When research agents are independent of each other, include all Task tool calls in a single message so they run concurrently. Only run agents sequentially when one agent's output is needed to inform the next agent's prompt.

   We have specialized agents that know how to do specific research tasks:

   **For codebase research:**
   - Use the **codebase-locator** agent to find WHERE files and components live
   - Use the **codebase-analyzer** agent to understand HOW specific code works (without critiquing it)
   - Use the **codebase-pattern-finder** agent to find examples of existing patterns (without evaluating them)

   **CRITICAL ORDERING — codebase-locator MUST run before codebase-analyzer and codebase-pattern-finder**:
   The codebase-locator identifies which files and components are relevant. Both the codebase-analyzer and codebase-pattern-finder need those file locations to know what to read and analyze. Always spawn codebase-locator first, wait for its results, then spawn codebase-analyzer and codebase-pattern-finder in parallel with the locator's findings included in their prompts.

   **IMPORTANT**: All agents are documentarians, not critics. They will describe what exists without suggesting improvements or identifying issues.

   **For web research (only if user explicitly asks):**
   - Use the **web-search-researcher** agent for external documentation and resources
   - IF you use web-research agents, instruct them to return LINKS with their findings, and please INCLUDE those links in your final report

   The key is to use these agents intelligently:
   - Start with locator agents to find what exists
   - Then use analyzer agents on the most promising findings to document how they work
   - Each agent knows its job - just tell it what you're looking for
   - Don't write detailed prompts about HOW to search - the agents already know
   - Remind agents they are documenting, not evaluating or improving

4. **Wait for all sub-agents to complete and synthesize findings:**

   **Announce**: > **Step 4/8: Synthesizing Findings** — All research agents have returned, compiling results...

   - IMPORTANT: Wait for ALL sub-agent tasks to complete before proceeding
   - Only now mark the research phase TaskCreate task as `completed`
   - **Verify sub-agent results**: If a sub-agent returns unexpected, incomplete, or suspicious results, spawn follow-up agents to verify. Cross-check findings against the actual codebase. Don't accept results that seem incorrect.
   - Compile all sub-agent results
   - Prioritize live codebase findings as primary source of truth
   - Connect findings across different components
   - Include specific file paths and line numbers for reference
   - Highlight patterns, connections, and architectural decisions
   - Answer the user's specific questions with concrete evidence

   **Research summary checkpoint** — Present a consolidated summary:
   ```
   **Research Complete.** Here's what I found:
   - [X] files identified as relevant across [Y] directories
   - Key discovery: [Most important finding with file:line reference]
   - Pattern: [Relevant convention or pattern documented]
   - Architecture: [How components connect]
   ```

5. **Gather metadata for the research document:**

   **Announce**: > **Step 5/8: Gathering Metadata** — Collecting git info and preparing document metadata...

   - Get current git info: `git rev-parse HEAD` and `git branch --show-current`
   - Get current date: use today's date in YYYY-MM-DD format
   - Filename: `project_docs/research/YYYY-MM-DD-description.md`
     - Format: `YYYY-MM-DD-description.md` where:
       - YYYY-MM-DD is today's date
       - description is a brief kebab-case description of the research topic
     - Example: `2025-01-08-authentication-flow.md`

6. **Generate research document:**

   **Announce**: > **Step 6/8: Generating Research Document** — Writing the full research document to disk...

   - Use the metadata gathered in step 5
   - Structure the document with YAML frontmatter followed by content:
     ```markdown
     ---
     date: [Current date in YYYY-MM-DD format]
     git_commit: [Current commit hash]
     branch: [Current branch name]
     repository: [Repository name from git remote or directory name]
     topic: "[User's Question/Topic]"
     tags: [research, codebase, relevant-component-names]
     status: complete
     last_updated: [Current date in YYYY-MM-DD format]
     ---

     # Research: [User's Question/Topic]

     **Date**: [Current date]
     **Git Commit**: [Current commit hash]
     **Branch**: [Current branch name]

     ## Research Question
     [Original user query]

     ## Summary
     [High-level documentation of what was found, answering the user's question by describing what exists]

     ## Detailed Findings

     ### [Component/Area 1]
     - Description of what exists (file.ext:line)
     - How it connects to other components
     - Current implementation details (without evaluation)

     ### [Component/Area 2]
     ...

     ## Code References
     - `path/to/file.py:123` - Description of what's there
     - `another/file.ts:45-67` - Description of the code block

     ## Architecture Documentation
     [Current patterns, conventions, and design implementations found in the codebase]

     ## Related Research
     [Links to other research documents in project_docs/research/]

     ## Open Questions
     [Any areas that need further investigation]
     ```

7. **Present findings:**

   **Announce**: > **Step 7/8: Presenting Findings** — Summarizing results for your review...

   - Present a concise summary of findings to the user
   - Include key file references for easy navigation
   - Ask if they have follow-up questions or need clarification

8. **Handle follow-up questions:**

   **Announce**: > **Step 8/8: Follow-up Research** — Investigating your follow-up question...
   - If the user has follow-up questions, append to the same research document
   - Update the frontmatter field `last_updated` to reflect the update
   - Add a new section: `## Follow-up Research [timestamp]`
   - Spawn new sub-agents as needed for additional investigation

## Important Notes

1. **Parallel Execution**:
   - Always use parallel Task agents to maximize efficiency and minimize context usage
   - **Prefer parallel execution**: When research agents are independent of each other, include all Task tool calls in a single message so they run concurrently. Only run agents sequentially when one agent's output is needed to inform the next agent's prompt.
   - **Exception — codebase-locator before codebase-analyzer and codebase-pattern-finder**: Always run codebase-locator first and feed its results into codebase-analyzer and codebase-pattern-finder. Both depend on the locator's output to know which files to read.

2. **Sub-agent Result Verification**:
   - If a sub-agent returns unexpected or incomplete results, spawn follow-up agents to verify
   - Cross-check findings against the actual codebase
   - Don't accept results that seem incorrect — investigate further
   - If an agent misses key files, re-run with more specific prompts

3. **Research Freshness**:
   - Always run fresh codebase research — never rely solely on existing research documents
   - The project_docs/research directory provides historical context to supplement live findings
   - Explore all of project_docs/research directory for related prior work

4. **Documentation Standards**:
   - Focus on finding concrete file paths and line numbers for developer reference
   - Research documents should be self-contained with all necessary context
   - Each sub-agent prompt should be specific and focused on read-only documentation operations
   - Document cross-component connections and how systems interact
   - Include temporal context (when the research was conducted)
   - Keep the main agent focused on synthesis, not deep file reading
   - Have sub-agents document examples and usage patterns as they exist

5. **Core Principles**:
   - **CRITICAL**: You and all sub-agents are documentarians, not evaluators
   - **REMEMBER**: Document what IS, not what SHOULD BE
   - **NO RECOMMENDATIONS**: Only describe the current state of the codebase

6. **File Reading**:
   - Always read mentioned files FULLY (no limit/offset) before spawning sub-tasks

7. **Critical Ordering** — Follow the numbered steps exactly:
   - ALWAYS read mentioned files first before spawning sub-tasks (step 1)
   - ALWAYS wait for all sub-agents to complete before synthesizing (step 4)
   - ALWAYS gather metadata before writing the document (step 5 before step 6)
   - NEVER write the research document with placeholder values

8. **Progress Tracking**:
   - Use a single TaskCreate task per research phase with a visible `activeForm` spinner — do not create per-agent tasks (the agent UI already shows individual agent progress)
   - Mark the research task `in_progress` before spawning agents, `completed` only after ALL agents return and findings are summarized
   - Announce each step transition so the user always knows what's happening

9. **Frontmatter Consistency**:
   - Always include frontmatter at the beginning of research documents
   - Keep frontmatter fields consistent across all research documents
   - Update frontmatter when adding follow-up research
   - Use snake_case for multi-word field names (e.g., `last_updated`, `git_commit`)
   - Tags should be relevant to the research topic and components studied
