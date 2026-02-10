The user input to you can be provided directly by the agent or as a command argument - you MUST consider it before proceeding with the prompt (if not empty).

User input:

$ARGUMENTS

Goal: Validate that the feature specification is clear, complete, and ready to break down into implementation plans. Detect and reduce ambiguity, missing requirements, or incomplete sections, and record clarifications directly in the spec file. This spec will drive the `/create_plan` command to produce implementation plans for AI coding agents. It is IMPERATIVE that requirements are unambiguous, testable, and complete enough to enable accurate planning.

Note: This review workflow is expected to run (and be completed) AFTER `/create_spec` and BEFORE `/create_plan`. If the user explicitly states they are skipping review (e.g., simple feature, time-boxed spike), you may proceed, but must warn that downstream planning quality may suffer.

Execution steps:

1. Read the spec file in its entirety.

2. Evaluate the spec against the completeness taxonomy below. For each category, assign a status:
   - **Clear**: Section is complete, unambiguous, and actionable
   - **Partial**: Section exists but has gaps, vague language, or missing details
   - **Missing**: Section is absent or essentially empty

   **Completeness Taxonomy:**
   | Category | What to Check |
   |----------|---------------|
   | User Stories | Are all user types covered? Are stories in proper format with clear benefit statements? |
   | Acceptance Criteria | Are criteria testable (Given/When/Then)? Do they cover happy path, errors, and edge cases? |
   | Data Requirements | Are all entities defined with fields, types, validation rules, and relationships? |
   | API Requirements | Are endpoints specified with methods, paths, request/response schemas, and error codes? |
   | UI/UX Requirements | Are interactions, layouts, and responsive behaviors defined? Are wireframes/mockups referenced? |
   | Business Logic | Are rules explicit and testable? Are conditions, actions, and exceptions documented? |
   | Security Requirements | Are auth, authorization, data protection, and audit needs specified? |
   | Performance Requirements | Are response times, throughput, and data volume targets defined? |
   | Integration Points | Are all external systems and their interfaces documented? |
   | Success Metrics | Are measurable outcomes defined? Can success be objectively determined? |
   | Dependencies | Are internal/external dependencies identified with version requirements? |
   | Scope Boundaries | Is "what we're NOT building" explicitly stated? Are future enhancements deferred? |

3. Generate (internally) a prioritized queue of candidate clarification questions. Do NOT output them all at once. Apply these constraints:
   - Each question must be answerable with EITHER:
     * A short multiple-choice selection (2-5 distinct, mutually exclusive options), OR
     * A one-word / short-phrase answer.
   - Only include questions whose answers materially impact:
     * Implementation planning accuracy
     * Test design completeness
     * Architecture decisions
     * Data modeling clarity
     * UX behavior specificity
     * Security posture
     * Integration requirements
   - Ensure category coverage balance: prioritize the highest-impact unresolved categories first; avoid asking two low-impact questions when a critical area (e.g., data model) remains unclear.
   - Exclude questions already answered in the spec and trivial stylistic preferences.
   - Favor clarifications that reduce downstream planning ambiguity.
   - Prioritize categories by (Impact * Uncertainty) heuristic, asking the highest-scoring ones first.

4. **Codebase investigation before asking (CRITICAL)**:
   For EACH question in your queue, determine if it is "code-discoverable" - i.e., could be answered by examining the existing codebase. Examples of code-discoverable questions:
   - "Does a similar feature already exist?"
   - "What data model pattern does the codebase use?"
   - "Is there an existing API endpoint we should follow as a pattern?"
   - "How does the current system handle this type of validation?"
   - "What authentication mechanism is already in place?"

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
   | Override | Provide a different answer |
   ```
   If user selects "Accept", use the proposed answer. If "Override", use their input.

   **Path B - Answer NOT discoverable from code:**
   Present the question normally (as in step 5) with multiple-choice or short-answer format.

   This step ensures the user is only asked questions that genuinely require human judgment or domain knowledge - NOT questions an AI can answer by reading the code.

5. Sequential questioning loop (interactive):
   - Present EXACTLY ONE question at a time.
   - For code-discoverable questions (from step 4), present using the Path A format with your discovery and proposed answer.
   - For non-code-discoverable questions, use multiple-choice format as a Markdown table:

   | Option | Description |
   |--------|-------------|
   | A | <Option A description> |
   | B | <Option B description> |
   | C | <Option C description> | (add D/E as needed up to 5)
   | Short | Provide a different short answer | (Include only if free-form alternative is appropriate)

   - For short-answer style (no meaningful discrete options), output a single line after the question: `Format: Short answer`.
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
   - Maintain in-memory representation of the spec (loaded once at start) plus the raw file contents.
   - For the first integrated answer in this session:
     * Ensure a `## Clarifications` section exists (create it just before the `## Open Questions` section if present, or at the end of the document if not).
     * Under it, create (if not present) a `### Session YYYY-MM-DD` subheading for today.
   - Append a bullet line immediately after acceptance: `- Q: <question> → A: <final answer>`.
   - Then immediately apply the clarification to the most appropriate section(s) of the spec document:
     * If the answer affects User Stories, update the relevant story's acceptance criteria
     * If the answer affects Data Requirements, add/update the entity or field definition
     * If the answer affects API Requirements, update the endpoint specification
     * If the answer affects Business Logic, add/update the rule definition
     * If the answer affects Security/Performance, update those sections
   - Save the spec file AFTER each integration to minimize risk of context loss (atomic overwrite).
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
   - Coverage summary table listing each taxonomy category with Status:
     * **Resolved** - Was Partial/Missing and has been addressed by clarifications
     * **Deferred** - User terminated early or better suited for planning phase
     * **Clear** - Already sufficient in original spec
     * **Outstanding** - Still Partial/Missing but low impact or user chose to proceed
   - Readiness assessment: Explicitly state whether the spec is ready for `/create_plan` or needs another review pass.
   - If any Outstanding or Deferred remain with high impact, recommend running `/review_spec` again or returning to `/create_spec` for deeper requirements gathering.

Behavior rules:
- **Codebase-first principle**: ALWAYS attempt to answer code-related questions from the codebase BEFORE asking the user. The user should only be asked questions that require human judgment, domain knowledge, or business decisions - NOT questions like "what pattern exists?" or "how does X work?" that can be discovered by reading code.
- If no meaningful ambiguities found (or all potential questions would be low-impact), respond: "No critical ambiguities detected. Spec appears ready for implementation planning." and suggest proceeding to `/create_plan`.
- Ask as many questions as needed to resolve all critical ambiguities (clarification retries for a single question do not count as new questions).
- Ensure tech stack alignment with codebase conventions discovered during investigation.
- Respect user early termination signals ("stop", "done", "proceed", "good enough").
- If no questions asked due to full coverage, output a compact coverage summary (all categories Clear) then recommend advancing to `/create_plan`.
- If the user terminates early with unresolved high-impact categories remaining, explicitly flag them under Deferred with rationale.
- Focus on PLANNING READINESS: The goal is not perfection, but ensuring the spec contains enough clarity for `/create_plan` to produce accurate implementation plans.

Planning readiness checklist (final assessment):
The spec is ready for `/create_plan` when:
- [ ] User stories clearly identify who, what, and why
- [ ] Acceptance criteria are testable (not vague)
- [ ] Data entities are defined well enough to design a schema
- [ ] API contracts are clear enough to implement endpoints
- [ ] Business rules can be translated into code logic
- [ ] Security requirements are explicit (not "TBD")
- [ ] Integration points identify specific systems and interfaces
- [ ] Success metrics can be measured objectively

If ANY of these are not met for HIGH-PRIORITY items, recommend additional clarification before proceeding.
