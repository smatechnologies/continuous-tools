The user input to you can be provided directly by the agent or as a command argument - you MUST consider it before proceeding with the prompt (if not empty).

User input:

$ARGUMENTS

Goal: Detect and reduce ambiguity or missing decision points in the implementation plan and record the clarifications directly in the plan file. This plan will be given to an AI coding agent to implement. It is IMPERITIVE that the goals and directions are clear and the scope is appropriately sized for AI implementation.

Note: This review workflow is expected to run (and be completed) BEFORE invoking /implement_plan. If the user explicitly states they are skipping review (e.g., exploratory spike), you may proceed, but must warn that downstream rework risk increases.

Execution steps:

1. Read the file in its entirety.

2. Evaluate the file against your goal.

3. Generate (internally) a prioritized queue of candidate clarification questions. Do NOT output them all at once. Apply these constraints:
- Each question must be answerable with EITHER:
* A short multiple‑choice selection (2–5 distinct, mutually exclusive options), OR
* A one-word / short‑phrase answer.
- Only include questions whose answers materially impact architecture, data modeling, task decomposition, test design, UX behavior, operational readiness, or compliance validation.
- Ensure category coverage balance: attempt to cover the highest impact unresolved categories first; avoid asking two low-impact questions when a single high-impact area (e.g., security posture) is unresolved.
- Exclude questions already answered and trivial stylistic preferences.
- Favor clarifications that reduce downstream rework risk.
- Prioritize categories by (Impact * Uncertainty) heuristic, asking the highest-scoring ones first.

4. **Codebase investigation before asking (CRITICAL)**:
For EACH question in your queue, determine if it is "code-discoverable" - i.e., could be answered by examining the codebase. Examples of code-discoverable questions:
- "Does component X currently exist?"
- "Does service Y already implement Z?"
- "What pattern does the codebase use for X?"
- "Is there an existing hook/utility/model for Y?"
- "How does feature X currently handle Y?"

For code-discoverable questions, you MUST:
a. Use the Task tool with `subagent_type=codebase-locator` to find relevant files/components.
b. If deeper analysis is needed, use the Task tool with `subagent_type=codebase-analyzer` to examine implementation details.
c. Based on findings, take ONE of these paths:

**Path A - Answer found with high confidence:**
Present the question AND your discovery to the user:
```
Q: <the question>

**Codebase discovery:** <what you found, with file references>

**Proposed answer:** <your recommended answer based on findings>

| Option | Description |
|--------|-------------|
| Accept | Use the proposed answer above |
| Override | Provide a different answer (short description) |
```
If user selects "Accept", use the proposed answer. If "Override", use their input.

**Path B - Answer NOT discoverable from code:**
Present the question normally (as in step 5) with multiple-choice or short-answer format.

This step ensures the user is only asked questions that genuinely require human judgment or domain knowledge - NOT questions an AI can answer by reading the code.

5. Sequential questioning loop (interactive):
- Present EXACTLY ONE question at a time.
- For code-discoverable questions (from step 4), present using the Path A format with your discovery and proposed answer.
- For non-code-discoverable questions, use multiple‑choice format as a Markdown table:

| Option | Description |
|--------|-------------|
| A | <Option A description> |
| B | <Option B description> |
| C | <Option C description> | (add D/E as needed up to 5)
| Short | Provide a different short answer | (Include only if free-form alternative is appropriate)

- For short‑answer style (no meaningful discrete options), output a single line after the question: `Format: Short answer`.
- After the user answers:
* Validate the answer maps to one option or is a custom short answer.
* If ambiguous, ask for a quick disambiguation (count still belongs to same question; do not advance).
* Once satisfactory, record it in working memory (do not yet write to disk) and move to the next queued question.
- Stop asking further questions when:
* All critical ambiguities resolved (remaining queued items become unnecessary), OR
* User signals completion ("done", "good", "no more").
- Never reveal future queued questions in advance.
- If no valid questions exist at start, immediately report no critical ambiguities.

6. Integration after EACH accepted answer (incremental update approach):
- Maintain in-memory representation of the plan (loaded once at start) plus the raw file contents.
- For the first integrated answer in this session:
* Ensure a `## Clarifications` section exists (create it just after the highest-level contextual/overview section per the plan template if missing).
* Under it, create (if not present) a `### Session YYYY-MM-DD` subheading for today.
- Append a bullet line immediately after acceptance: `- Q: <question> → A: <final answer>`.
- Then immediately apply the clarification to the most appropriate section(s) of the Plan document (current state or a phase)
- Save the plan file AFTER each integration to minimize risk of context loss (atomic overwrite).
- Preserve formatting: do not reorder unrelated sections; keep heading hierarchy intact.
- Keep each inserted clarification minimal and testable (avoid narrative drift).

7. Validation (performed after EACH write plus final pass):
- Clarifications session contains exactly one bullet per accepted answer (no duplicates).
- No duplicate questions asked.
- Updated sections contain no lingering vague placeholders the new answer was meant to resolve.
- No contradictory earlier statement remains (scan for now-invalid alternative choices removed).
- Markdown structure valid; only allowed new headings: `## Clarifications`, `### Session YYYY-MM-DD`.
- Terminology consistency: same canonical term used across all updated sections.

8. Write the updated spec back to its file location.

9. Report completion (after questioning loop ends or early termination):
- Number of questions asked & answered.
- Path to updated spec.
- Sections touched (list names).
- Coverage summary table listing each taxonomy category with Status: Resolved (was Partial/Missing and addressed), Deferred (user terminated early or better suited for planning), Clear (already sufficient), Outstanding (still Partial/Missing but low impact).
- If any Outstanding or Deferred remain, recommend whether to proceed to `/implement_plan` or run `/review` again.

Behavior rules:
- **Codebase-first principle**: ALWAYS attempt to answer code-related questions from the codebase BEFORE asking the user. The user should only be asked questions that require human judgment, domain knowledge, or business decisions - NOT questions like "does X exist?" or "what pattern is used for Y?" that can be discovered by reading code.
- If no meaningful ambiguities found (or all potential questions would be low-impact), respond: "No critical ambiguities detected worth formal clarification." and suggest proceeding.
- Ask as many questions as needed to resolve all critical ambiguities (clarification retries for a single question do not count as new questions).
- Ensure tech stack clarity.
- Respect user early termination signals ("stop", "done", "proceed").
- If no questions asked due to full coverage, output a compact coverage summary (all categories Clear) then suggest advancing.
- If the user terminates early with unresolved high-impact categories remaining, explicitly flag them under Deferred with rationale.
