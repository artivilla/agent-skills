---
name: entire-commit-composer
description: Compose grouped Conventional Commits from the current branch by reading Entire context first. Use when Codex needs to review the current branch plus all active Entire sessions in the same worktree, synthesise committed, staged, and unstaged changes, propose commit groups, stage each group, and ask for one approval before creating the full commit sequence.
---

# Entire Commit Composer

Read the existing Entire skill instructions before composing a plan:

1. `entire:using-entire`
2. `entire:review`
3. `entire:session-handoff`
4. `entire:search` only if the active-session transcripts do not explain a meaningful part of the branch

Treat those skills as the source of truth for CLI usage and transcript handling. This skill exists to combine them into one deliberate commit-composition workflow.

## Workflow

### 1. Verify the environment

Run:

```bash
entire version
```

If the command is unavailable, stop and tell the user that Entire CLI is required for this workflow.

### 2. Resolve the worktree and active sessions

Run:

```bash
entire session current --json
entire session list --json
```

Use `worktree_path` from `session current` when available. Include every active session whose `worktree_path` matches the current worktree exactly. If no exact matches exist, fall back to the bidirectional prefix-match behaviour described in `entire:session-handoff`.

Do not limit the workflow to the current session. Read every active session in the worktree explicitly.

### 3. Read every active session transcript

For each chosen session:

```bash
entire session info <session_id> --transcript > /tmp/entire-commit-composer-<session_id>.jsonl
```

Extract the user requests, key decisions, completed work, failed attempts, blockers, and touched files. Build a concise intent summary for each session. Do not dump raw transcript lines to the user.

If there are no active sessions in the worktree, continue in degraded mode and state that commit grouping will rely on diffs and checkpoint-backed history only.

### 4. Detect the base ref and branch scope

Detect the base ref using the same order as `entire:review`:

```bash
git rev-parse --verify origin/HEAD 2>/dev/null
git rev-parse --verify origin/main 2>/dev/null
git rev-parse --verify origin/master 2>/dev/null
git rev-parse --verify main 2>/dev/null
git rev-parse --verify master 2>/dev/null
```

Then read:

```bash
git diff --stat <base>..HEAD
git diff <base>..HEAD
git diff --cached
git diff
git status --short
```

Interpret them as:

- `git diff <base>..HEAD`: committed branch changes
- `git diff --cached`: staged but uncommitted changes
- `git diff`: unstaged changes

If no base ref exists, stop and ask the user to provide one. Do not guess.

### 5. Build the intent and change map

Merge three sources of truth:

- active-session summaries
- checkpoint-backed intent from branch commits
- actual diffs in the working tree

When branch commits need richer intent context, inspect checkpoint trailers from the branch log and use the same `entire explain --checkpoint <id> --json --no-pager` flow described by `entire:review`.

Organise the result by coherent change area, not merely by path. A useful grouping unit is a feature slice, bug fix, refactor seam, or test/documentation companion change.

Label each conclusion as one of:

- recorded session intent
- checkpoint-backed intent
- inferred from diff

Never present inferred intent as recorded fact.

### 6. Derive commit groups

Propose the smallest sensible sequence of reviewable commits.

Use Conventional Commit headers:

```text
type(scope): summary
```

Commit grouping rules:

- group by coherent intent, behaviour, or refactor seam
- keep feature work separate from refactors unless they are inseparable
- keep tests with the code they validate when that improves reviewability
- separate mechanical follow-ups from functional changes when practical
- surface ambiguous files as their own review item rather than forcing them into a weak group
- avoid grouping solely by directory, extension, or whether a file is staged already

For each proposed group, include:

- commit title
- rationale
- files
- whether the files are committed-history context, staged, unstaged, or mixed
- exact staging command

Prefer `git add <paths...>` for explicit groups. Use `git add -p` only when a file must be split across groups and say so plainly.

### 7. Present the composition plan

Before staging anything, present the ordered commit groups in this format:

```text
Entire Commit Composer:

Composing <branch> against <base>
Active session context: <N sessions | unavailable>

1. <type(scope): summary>
   Rationale: <why this is its own commit>
   Files:
   - path/one
   - path/two
   Staging: <exact command>

2. <type(scope): summary>
   Rationale: <why this follows>
   Files:
   - path/three
   Staging: <exact command>

Ambiguous:
 - <path>: <why it does not fit cleanly yet>
```

If the grouping is weak or a file could belong to two groups, say so before staging.

### 8. Stage the proposed groups after approval

After presenting the full composition plan, ask for one explicit approval to execute the grouped commit sequence.

Example:

```text
Proposed commit sequence:
- feat(api): add fulfillment audit logging
- docs(api): add smoke test runbook

Stage and create these commits in order?
```

Do not stage or create any commit until that approval is received.

Once approved, stage one group at a time using the exact command you proposed.

After staging each group, run:

```bash
git diff --cached --stat
git diff --cached
```

Use that check to confirm the staged content matches the intended group. If it does not, unstage the affected paths and correct the grouping before creating commits.

### 9. Create each commit without re-asking

After the user has approved the full grouped sequence, create each staged commit in order without asking again between groups.

Use a non-interactive git command:

```bash
git commit -m "<type(scope): summary>"
```

### 10. Create the commit and continue

After the single approval, continue through the remaining groups, repeating the stage, verify, and commit cycle until all groups are handled or the user stops the process.

## Failure Modes

- No Entire CLI: stop and say Entire CLI is required.
- No active sessions: continue in degraded mode with diff and checkpoint context only.
- No base ref: stop and ask the user for the base branch or commit.
- More than one plausible grouping: present the alternatives and explain the trade-off.
- Files need partial staging: call that out explicitly and use patch staging carefully.
- Generated, lock, or binary files: include them only when they materially belong to a commit group.

## Boundaries

- Do not create commits automatically after composing the plan.
- Do not stage or create any commit until the user has approved the full grouped commit sequence.
- Do not amend or squash existing commits unless the user explicitly asks.
- Do not hide uncertainty when grouping files; surface it before staging.
- Do not mutate files as part of this skill. This workflow is for review, grouping, staging, and committing only.
