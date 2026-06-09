# Design: {Title}

## Context

{Why we're doing this. Reference relevant ADRs from docs/decisions.md. Keep it to 2-3 sentences.}

## Approach

{The general strategy. How does this solve the problem? One paragraph, no implementation details.}

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

```
NewFile ──→ ExistingModule
   │              │
   └── Service ◄──┘
```

## Key Decisions

### D1: {Decision Title}

**Choice**: {What we chose}
**Why**: {Why this over alternatives}
**Trade-off**: {What we give up}

### D2: {Decision Title}

**Choice**: {What we chose}
**Why**: {Why}
**Trade-off**: {What we give up}

## Contracts

```typescript
// New interfaces, API contracts, type definitions
interface Example {
  id: string
  name: string
}
```

## Migration

{If data migration, feature flags, or phased rollout is needed, describe the plan. Otherwise: "No migration required."}