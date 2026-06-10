# SDD Lite — Common Protocol

Minimal shared protocol for all SDD Lite agents. Sub-agents SHOULD read this alongside their phase-specific SKILL.md.

Executor boundary: every SDD Lite phase agent is an EXECUTOR, not an orchestrator. Do the phase work yourself. Do NOT launch sub-agents, do NOT call `delegate`/`task`, and do NOT bounce work back unless the phase skill explicitly says to stop and report a blocker.

## A. Project Context Loading

ALL SDD Lite agents MUST read project context before starting work:

```
1. Read PROJECT_CONTEXT.md — project architecture and patterns
2. Read docs/decisions.md — PROJECT-LEVEL architectural decisions only (not change-specific)
3. Read CHANGELOG.md — project history
4. Check engram: mem_search(query: "sdd-lite", project: "{project}")
```

If PROJECT_CONTEXT.md doesn't exist, proceed but note it. Suggest onboarding if needed.

When reading docs/decisions.md, note that it ONLY contains project-level decisions (stack, architecture, cross-cutting patterns). Change-specific decisions live in each change's plan.md under `## Decisions`.

## B. Skill Loading

1. Check if the orchestrator injected a `## Project Standards (auto-resolved)` block in your launch prompt. If yes, follow those rules.
2. If no Project Standards block, check for `SKILL: Load` instructions. If present, load those exact skill files.
3. If neither was provided, proceed with your phase skill only.

## C. Artifact Persistence

SDD Lite uses TWO persistence layers:

### Filesystem (Primary)

Write artifacts to the project directory:
- `docs/changes/{change-name}/plan.md` → per-change plan
- `docs/changes/{change-name}/design.md` → per-change design (large changes only)
- `PROJECT_CONTEXT.md` → project root (permanent)
- `docs/decisions.md` → ADRs (permanent)
- `CHANGELOG.md` → project root (permanent)

### Engram (Secondary)

Save key decisions and discoveries:

```
mem_save(
  title: "sdd-lite/{change-name}/{artifact-type}",
  topic_key: "sdd-lite/{change-name}/{artifact-type}",
  type: "architecture",
  project: "{project}",
  content: "{artifact content}"
)
```

For plan.md specifically:
```
mem_save(
  title: "sdd-lite/{change-name}/plan",
  topic_key: "sdd-lite/{change-name}/plan",
  type: "architecture",
  project: "{project}",
  content: "{full plan content}"
)
```

### Both layers

- Write files to filesystem for version control and human review
- Save key decisions to engram for cross-session persistence
- These are complementary, not redundant

## D. Return Envelope

Every phase MUST return a structured envelope to the orchestrator:

- `status`: `success`, `partial`, or `blocked`
- `summary`: 1-3 sentence executive summary of what was done. An executive summary, NOT a reproduction of file contents. See Section G.
- `files_changed`: list of files created or modified
- `observations`: things noticed outside scope (Least Touch principle)
- `next_recommended`: the next phase to run, or "none"

Example:

```markdown
**Status**: success
**Summary**: Implemented auth middleware and JWT validation as described in plan.md.
**Files Changed**: src/auth/middleware.ts (created), src/routes/index.ts (modified)
**Observations**: Found TODO in user.service.ts about rate limiting — outside scope, noted for future.
**Next**: None — implementation complete, ready for review.
```

## E. Least Touch Principle

This is the core principle of SDD Lite. ALL agents must follow it:

1. ONLY modify files explicitly listed in the plan
2. If you notice something outside scope that could be improved, NOTE it as an observation but DON'T implement it
3. If you find a bug while implementing, NOTE it but DON'T fix it (unless it directly blocks the planned change)
4. Observations go in the return envelope and can be added to plan.md's "Observations" section for future reference

## F. Change Size Routing

The orchestrator routes based on change size:

| Size | Files | Flow | Artifacts |
|------|-------|------|-----------|
| Simple | 1-3 | Direct → sdd-lite-apply | Instructions in delegation prompt |
| Medium | 4-10 | Plan → sdd-lite-apply | docs/changes/{name}/plan.md |
| Large | 10+ | Plan → Design → sdd-lite-apply | docs/changes/{name}/plan.md + design.md |
| Explore | any | Investigate → sdd-lite-explore | Structured report (no files) |
| Onboard | any | Archaeology → sdd-lite-onboard | PROJECT_CONTEXT.md, decisions.md, CHANGELOG.md |

## G. Summarize, Don't Duplicate

When an agent writes an artifact to disk, it MUST return a SUMMARY of key points to the orchestrator, NOT the full file contents. The user can open the file in their editor.

For plan.md: Return intent, scope, affected files list, and impact checklist status.
For design.md: Return approach, key decisions (D1, D2...), and files to create/modify.
For PROJECT_CONTEXT.md: Return stack summary, architecture summary, and decision count.
For decisions.md: Return the new ADRs added (title and one-liner each).
For CHANGELOG.md: Return the version and entry summary.

This rule exists because token waste on duplicating file contents helps NO ONE. The orchestrator presents summaries to the user. If the user wants details, they open the file.