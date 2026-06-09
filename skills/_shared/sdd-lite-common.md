# SDD Lite — Common Protocol

Minimal shared protocol for all SDD Lite agents. Sub-agents SHOULD read this alongside their phase-specific SKILL.md.

Executor boundary: every SDD Lite phase agent is an EXECUTOR, not an orchestrator. Do the phase work yourself. Do NOT launch sub-agents, do NOT call `delegate`/`task`, and do NOT bounce work back unless the phase skill explicitly says to stop and report a blocker.

## A. Project Context Loading

ALL SDD Lite agents MUST read project context before starting work:

```
1. Read PROJECT_CONTEXT.md — project architecture and patterns
2. Read docs/decisions.md — existing architectural decisions
3. Read CHANGELOG.md — project history
4. Check engram: mem_search(query: "sdd-lite", project: "{project}")
```

If PROJECT_CONTEXT.md doesn't exist, proceed but note it. Suggest onboarding if needed.

## B. Skill Loading

1. Check if the orchestrator injected a `## Project Standards (auto-resolved)` block in your launch prompt. If yes, follow those rules.
2. If no Project Standards block, check for `SKILL: Load` instructions. If present, load those exact skill files.
3. If neither was provided, proceed with your phase skill only.

## C. Artifact Persistence

SDD Lite uses TWO persistence layers:

### Filesystem (Primary)

Write artifacts to the project directory:
- `plan.md` → project root (or `.opencode/` if it exists)
- `design.md` → project root (or `.opencode/` if it exists)
- `PROJECT_CONTEXT.md`
- `docs/decisions.md`
- `CHANGELOG.md`

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
- `summary`: 1-3 sentence summary of what was done
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
| Medium | 4-10 | Plan → sdd-lite-apply | plan.md |
| Large | 10+ | Plan → Design → sdd-lite-apply | plan.md + design.md |
| Explore | any | Investigate → sdd-lite-explore | Structured report (no files) |
| Onboard | any | Archaeology → sdd-lite-onboard | PROJECT_CONTEXT.md, decisions.md, CHANGELOG.md |