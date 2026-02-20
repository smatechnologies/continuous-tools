---
description: View plugin version, mode, and commands
---

# Plugin Info

You are tasked with displaying the ct plugin's info dashboard. Output the logo, version, current mode, and available commands/agents.

## Process

1. **Read current mode** from `plugins/ct/config/mode.md`. Default to `quality` if missing or unreadable.

2. **Output the following exactly** (replace `[MODE]` with the current mode capitalized, e.g., `Quality` or `Balanced`):

```
───────────────────────────────────────────────────────
       #############               #############
    ###################         ###################
  ##### ########## ######      #### ########### ####
 #### ####################  ##### ############### ####
#### ####        ##### ########  ####         #### ###
### ####           ####  ##### #####           #### ###
### ###              # ##### #####              ### ###
### ###              ##### # #####              ### ###
### ####           #####  #### #####           #### ###
#### ####        ##### ### ##### #####       ##### ###
 #### ############## #####   ################### ####
  ###### ######### #####       ##### ######### #####
    ##################           #################
       ############                 ###########
───────────────────────────────────────────────────────
```

## Continuous Tools for Claude Code

### Plugin Status
| | |
|---|---|
| **Version** | [1.0.6](https://github.com/smatechnologies/continuous-tools/blob/main/plugins/ct/CHANGELOG.md)|
| **Mode** | [MODE] |
| **Agents** | 14 available |
| **Commands** | 11 loaded |

### Workflow Commands (in suggested order of execution)

| Command | Description |
|---------|-------------|
| `/ct:create-spec` | Create a feature specification interactively |
| `/ct:review-spec` | Review a spec for clarity and completeness |
| `/ct:research-codebase` | Research a particular aspect of the codebase in depth |
| `/ct:create-plan` | Build an implementation plan (provide spec and/or research doc) |
| `/ct:review-plan` | Review a plan for ambiguity and feasibility |
| `/ct:implement-plan` | Execute a plan with incremental validation |
| `/ct:commit` | Create a git commit following project conventions |
| `/ct:review-code` | Delegate a code review to a specialized agent |

### Additional Commands

| Command | Description |
|---------|-------------|
| `/ct:debug` | Investigate and fix bugs with debug-investigator |
| `/ct:mode` | Switch between Quality and Balanced modes |
| `/ct:info` | Show this dashboard |

### Agents ([MODE] mode)

| Agent | Model |
|-------|-------|
| codebase-analyzer | [T1_MODEL] |
| code-review-expert | [T1_MODEL] |
| debug-investigator | [T1_MODEL] |
| codebase-pattern-finder | [T2_MODEL] |
| web-search-researcher | [T2_MODEL] |
| dotnet-backend-expert | [T2_MODEL] |
| react-typescript-expert | [T2_MODEL] |
| nextjs-fullstack-expert | [T2_MODEL] |
| python-pro-backend | [T2_MODEL] |
| database-efcore-expert | [T2_MODEL] |
| integration-workflow-expert | [T2_MODEL] |
| acceptance-test-pro | [T2_MODEL] |
| fullstack-testing-expert | [T2_MODEL] |
| codebase-locator | [T3_MODEL] |

## Important
- Output the dashboard as-is. Do not add any commentary before or after it.
- Replace `[MODE]` with the current mode from the config file (capitalized, e.g., `Quality` or `Balanced`).
- Replace model placeholders based on the current mode:
  - **Quality**: `[T1_MODEL]` = `Opus`, `[T2_MODEL]` = `Opus`, `[T3_MODEL]` = `Sonnet`
  - **Balanced**: `[T1_MODEL]` = `Opus`, `[T2_MODEL]` = `Sonnet`, `[T3_MODEL]` = `Haiku`
