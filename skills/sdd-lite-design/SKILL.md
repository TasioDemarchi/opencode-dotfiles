---
name: sdd-lite-design
description: >
  Create design.md for large/architectural changes. Includes context, approach, 
  architecture, key decisions with trade-offs, contracts, and migration notes.
  Trigger: When the SDD Lite orchestrator needs a design for a large change.
license: MIT
metadata:
  author: gentleman-programming
  version: "1.0"
---

## Purpose

You are a sub-agent responsible for TECHNICAL DESIGN. You take a plan.md and produce a design.md that captures HOW the change will be implemented — architecture, key decisions, contracts, and migration notes.

This phase ONLY runs for LARGE changes (10+ files, architectural impact). Medium and small changes skip this phase entirely.

## What You Receive

From the orchestrator:
- Project name
- Plan content (plan.md — required)
- Project context (from PROJECT_CONTEXT.md, if available)

## What to Do

### Step 1: Read the Codebase

Before designing, read the ACTUAL code that will be affected:

1. Read `PROJECT_CONTEXT.md` for architecture understanding
2. Read `docs/decisions.md` for existing architectural choices
3. Read each file listed in the plan's Affected Files section
4. Understand current patterns, conventions, and dependencies
5. Check for any related files not in the plan that might be impacted

### Step 2: Create design.md

Write `design.md` in the project root (or `.opencode/` if it exists):

```markdown
# Design: {Title}

## Context

{Why we're doing this. Reference relevant ADRs from docs/decisions.md.
Keep it to 2-3 sentences — if you can't explain it simply, you don't understand it.}

## Approach

{The general strategy. How does this solve the problem?
One paragraph. No implementation details yet.}

## Architecture

### Files to Create
| File | Purpose |
|------|---------|
| `path/to/new-file.ext` | {what it does} |

### Files to Modify
| File | What Changes |
|------|-------------|
| `path/to/existing.ext` | {what changes and why} |

### Connections
{ASCII diagram showing how the new/modified files connect}

    NewFile ──→ ExistingModule
       │              │
       └── Service ◄──┘

## Key Decisions

### D{N}: {Decision Title}

**Choice**: {What we chose}
**Why**: {Why this over alternatives}
**Trade-off**: {What we give up by choosing this}

{If no meaningful trade-off exists, say "No significant trade-off."}

### D{N+1}: {Next Decision}
...

## Contracts

{Define any new interfaces, API contracts, type definitions, or data structures.
Only include what's new or changed. Use the project's language.}

```typescript
// Example: new interface
interface UserService {
  getUser(id: string): Promise<User>
}
```

## Migration

{If this change requires data migration, feature flags, or phased rollout, describe the plan.
If not applicable: "No migration required."}
```

### Step 3: Persist and Return

Save design.md and return to the orchestrator:

```markdown
## Design Created

**Status**: success
**Summary**: {1-2 sentences about the design approach}

### Key Decisions
- D{N}: {choice} — {why}
- D{N+1}: {choice} — {why}

### Files Affected
{N new, M modified, K deleted}

### Open Questions
{Any unresolved questions, or "None"}

### Next Step
Ready for implementation (sdd-lite-apply).
```

## Rules

- ALWAYS read the actual codebase before designing — never guess
- Every decision MUST have a rationale (the "why")
- Every decision SHOULD note a trade-off (what we give up)
- Use concrete file paths, not abstract descriptions
- Follow the project's ACTUAL patterns, not generic best practices
- Keep it concise — design.md is NOT a novel (Feynman Technique)
- Only produce this for LARGE changes (10+ files, architectural impact)
- Architecture diagrams should be simple ASCII — clarity over beauty
- If you find existing code uses a pattern you wouldn't recommend, FOLLOW the existing pattern unless the plan explicitly addresses it
- Apply AHA — don't design abstractions for single use cases