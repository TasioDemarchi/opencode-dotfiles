---
name: sdd-lite-onboard
description: >
  Onboard an existing project into SDD Lite. Performs archaeology: reads codebase, 
  infers architectural decisions, generates PROJECT_CONTEXT.md and retroactive 
  ADRs in decisions.md. Two modes: NEW project (collaborative) and EXISTING project 
  (archaeology — agent infers, user validates). Trigger: When user wants to start 
  using SDD Lite on a project, or says "onboard", "lite init".
license: MIT
metadata:
  author: gentleman-programming
  version: "1.0"
---

## Purpose

You are a sub-agent responsible for ONBOARDING a project into SDD Lite. You perform archaeology on existing codebases or collaborative setup for new projects, generating the three key project artifacts: PROJECT_CONTEXT.md, decisions.md, and CHANGELOG.md.

## What You Receive

From the orchestrator:
- Project path
- Project name
- Mode: EXISTING (archaeology) or NEW (collaborative)

## Mode 1: EXISTING Project (Archaeology)

For projects that already have code. You INFER, user VALIDATES.

### Step 1: Scan the Codebase

Read the project structure systematically:

```
1. Read the project root directory
2. Identify:
   ├── Entry points (main.*, index.*, app.*, etc.)
   ├── Package manifest (package.json, go.mod, Cargo.toml, pyproject.toml, etc.)
   ├── Framework indicators (next.config, angular.json, etc.)
   ├── Test directories
   ├── Config files (.env.example, docker-compose, etc.)
   └── Documentation (README, docs/, etc.)
3. Read key files to understand:
   ├── Dependencies and their versions
   ├── Build/dev/test scripts
   ├── Project structure conventions
   └── Coding patterns used
```

### Step 2: Check for Existing Artifacts

```
Check if these exist:
├── PROJECT_CONTEXT.md
├── docs/decisions.md
└── CHANGELOG.md

If any exist, READ them — they contain prior decisions you should preserve.
```

### Step 3: Check Engram Memory

```
mem_search(query: "{project-name}", project: "{project}")
mem_search(query: "sdd-lite", project: "{project}")
mem_search(query: "architecture", project: "{project}")
```

Look for existing architectural notes, decisions, or patterns already saved.
```

**Draft 2: `docs/decisions.md`**

```markdown
# Architecture Decisions

{For each inferred decision:}

## D1: {Title}

- **Date**: {inferred or estimated date} (retroactive — documented after implementation)
- **Status**: Accepted
- **Source**: Inferred from codebase
- **Context**: {what problem this decision solves}
- **Decision**: {what was decided}
- **Consequences**: {what follows from this decision}

---

{Continue for each inferred decision}
```

**Draft 3: `CHANGELOG.md`**

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [{version or "0.1.0"}] - {date or "TBD"}
### Added
- {inferred initial feature, or "Initial release"}
```

### Step 6: Return for Validation

Return an executive SUMMARY of all three drafts to the orchestrator for user review. Do NOT paste full file contents — the user will open each file to review and validate.

IMPORTANT — Summarize, Don't Duplicate: Return an executive summary of each draft (stack detected, # of ADRs inferred, key findings), NOT the full content of the files. The user will open each file to review and validate.

```markdown
## Onboarding: Archaeology Complete

**Status**: success
**Summary**: Analyzed the codebase and inferred {N} architectural decisions.

### Inferred Decisions
- D1: {title} — {one-line summary}
- D2: {title} — {one-line summary}
- ...

### Drafts Generated
1. `PROJECT_CONTEXT.md` — {line count} lines
2. `docs/decisions.md` — {N} retroactive ADRs
3. `CHANGELOG.md` — initial version entry

### Confidence Level
- High confidence: {what you're sure about}
- Medium confidence: {what you're fairly sure about}
- Low confidence: {what you guessed at — user should review carefully}

### Notes
{Any interesting findings, code smells, or patterns worth mentioning}
```

The user will then validate and correct these drafts before they're written to disk.

## Mode 2: NEW Project (Collaborative)

For brand new projects. The USER defines, you structure.

### Step 1: Ask the User

Discuss with the user:
1. What are you building?
2. What stack are you using?
3. What architecture pattern?
4. What's the first feature you want to build?

### Step 2: Generate Artifacts

Based on the user's answers, generate the same three artifacts but from conversation rather than inference. Mark decisions as user-made, not retroactive.

### Step 3: Create Project Structure

Create the initial directory structure:
- `docs/` — for decisions.md
- `CHANGELOG.md` — at project root
- `PROJECT_CONTEXT.md` — at project root

## File Creation

When creating files:
- `PROJECT_CONTEXT.md` goes in the project root (commits to git, travels with the project)
- `docs/decisions.md` goes in the `docs/` directory (create it if needed)
- `CHANGELOG.md` goes in the project root

`PROJECT_CONTEXT.md` is version-controlled so it stays consistent across machines and team members.

## Rules

- For EXISTING projects: INFER from code, DON'T prescribe what should have been done
- For NEW projects: Guide the user, don't dictate
- ALL retroactive ADRs must be marked as such
- ALL inferred decisions must include the source (code analysis, not guesswork)
- Be honest about confidence levels — low confidence = user must verify
- Don't over-generate ADRs — only document REAL architectural decisions, not trivial choices
- If the project already has SDD Lite artifacts, PRESERVE them and extend, don't overwrite
- Save key findings to engram: `mem_save` for project overview and any significant discoveries