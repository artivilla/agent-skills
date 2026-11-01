# Workflow Notes

## Base Ref Detection Order

Use the first ref that resolves successfully:

```bash
git rev-parse --verify origin/HEAD 2>/dev/null
git rev-parse --verify origin/main 2>/dev/null
git rev-parse --verify origin/master 2>/dev/null
git rev-parse --verify main 2>/dev/null
git rev-parse --verify master 2>/dev/null
```

If none resolve, stop and ask the user for a base ref.

## Session Selection Rules

1. Read `entire session current --json`.
2. Read `entire session list --json`.
3. Prefer exact `worktree_path` matches.
4. If exact matching yields nothing, use the bidirectional prefix-match fallback from `entire:session-handoff`.
5. Keep every active session in the worktree, not merely the current or most recent one.

## Commit Grouping Heuristics

- Prefer user-visible behavioural changes before follow-up refactors.
- Keep refactors separate when the code still works without them.
- Keep tests with the commit they best explain.
- Isolate tooling, docs, or fixture churn when it would distract from the functional change.
- Treat ambiguous files as unresolved until the user confirms placement.

## Confirmation Rules

- Present all proposed groups before staging.
- Stage one group at a time.
- Verify the staged diff before asking for approval.
- Ask for explicit confirmation before each `git commit -m ...`.
- Stop immediately if the user declines a group or wants regrouping.
