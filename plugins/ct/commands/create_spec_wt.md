---
description: Create a feature specification (with worktree protection for develop/main branches)
argument-hint: [feature-description]
---

# Create Feature Specification (with Worktree Protection)

This command creates a feature specification just like `/ct:create_spec`, but first ensures you are NOT working directly on the `develop` or `main` branch.

## Pre-Flight: Git Worktree Guard

When this command is invoked, you MUST perform these checks **before doing anything else**:

### Step 0: Branch Safety Check

1. **Check if this is a git repository**:
   - Run `git rev-parse --is-inside-work-tree` via the Bash tool.
   - If it fails (not a git repo), skip all worktree logic and proceed directly to the spec creation process below.

2. **Check the current branch**:
   - Run `git branch --show-current` via the Bash tool.
   - If the current branch is **NOT** `develop` **and NOT** `main`, no worktree is needed. Announce: "Currently on branch `<branch-name>` — proceeding without creating a worktree." Then skip to the spec creation process below.

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

- **NEVER** create or modify files while on the `develop` or `main` branch. The worktree step is mandatory when on `develop` or `main`.
- If `EnterWorktree` fails for any reason, **stop and inform the user**. Do not fall back to working on `develop` or `main`.
- The feature ID should appear in both the worktree directory name and the branch name so it is easy to identify later.

---

## Spec Creation Process

Once the branch safety check is complete, execute the full spec creation process exactly as defined in the `/ct:create_spec` command.

Invoke the Skill tool with `skill: "ct:create_spec"` and pass along any arguments the user originally provided.
