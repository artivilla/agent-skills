---
name: pr-summary-formatter
description: Write clear, concise PR summaries from every commit on the current branch. Use when creating or updating pull request descriptions and commit breakdowns.
---

# PR Summary Formatter

Write for product and engineering reviewers. State what changed, why it matters, and nothing more.

## Required workflow

Before drafting the pull request body:

1. Resolve the base ref using the first reference that exists:

   ```bash
   git rev-parse --verify origin/HEAD
   git rev-parse --verify origin/main
   git rev-parse --verify origin/master
   git rev-parse --verify main
   git rev-parse --verify master
   ```

2. Read every branch commit in chronological order:

   ```bash
   git log --reverse --format='%H%x09%s' <base>..HEAD
   ```

3. Inspect the aggregate changes:

   ```bash
   git diff --stat <base>..HEAD
   git diff <base>..HEAD
   ```

4. Inspect repository test and hook configuration, such as `lefthook.yml`, `justfile`, and package scripts, to determine which local commands may have run.
5. Use the actual hash and exact subject returned by Git.
6. Include every commit exactly once, in commit order. Include commits added after the pull request was created.
7. Use short hashes in the final body unless the full hash is required.
8. Never use `unknown`, `<hash>`, guessed hashes, or guessed subjects. If commit data cannot be resolved, stop and report the problem.
9. Report local test execution only from observed command output, hook output, or an explicitly supplied result. Label the suite by project or scope, such as `Backend tests`, and state whether it passed, failed, or was not run locally. Never infer execution from changed files or from a hook definition.
10. Do not group multiple commits into one numbered entry. The numbered breakdown represents actual Git commits, not conceptual change areas.

## Format

Use exactly this structure:

```markdown
type(scope): summary line

  - net effect one
  - net effect two
  - net effect three

  This change <single sentence impact>.

This PR <2-3 sentence overview>.

1. `<short hash>` `<exact commit subject>`
- detail one
- detail two

2. `<short hash>` `<exact commit subject>`
- detail one
- detail two

3. `<short hash>` `<exact commit subject>`
- detail one
- detail two

Testing:
- Test files changed: <paths, or `none`>
- Backend tests: <command and `passed`, `failed`, or `not run locally`>
- Other test suites: <suite, command, and `passed`, `failed`, or `not run locally`, if applicable>
```

Repeat the numbered commit section until every branch commit is included.

## Writing rules
- Use short, direct sentences.
- Prefer concrete verbs: add, fix, move, remove, use, and update.
- Use technical terms when they name the thing precisely.
- State what changed before explaining why it matters.
- Keep each bullet to one idea.
- Tie every claim to an actual diff or commit.
- Do not invent behaviour changes.
- Avoid filler and vague praise such as improved, robust, seamless, and comprehensive.
- Use the three top-level bullets for net effects across the pull request.
- Use the numbered section for actual commits, never conceptual groupings.
- Keep each commit detail to two concise bullets where possible.
- Use the exact commit subject returned by Git.
- Separate test files changed from tests executed locally.
- Identify changed test files from the diff, including test fixtures and snapshots where relevant.
- Name individual test files only when the local command output identifies them as executed.
- If a command runs a suite without listing individual files, report the suite and command instead of guessing file-level coverage.
- When `just test` runs the backend suite, label it `Backend tests` and report the observed result as `passed` or `failed`.
- Do not call a test suite `passed` merely because the command was configured or started; require successful command output or an explicitly supplied result.
- If hooks skip tests for the changed file types, report that tests were not run locally and explain the skip when useful.
- Do not include raw Git commands in the final body unless they are relevant to the change.

## Validation before publishing

Run:

```bash
git rev-list --count <base>..HEAD
git log --reverse --format='%h%x09%s' <base>..HEAD
```

Confirm that:

- The number of numbered entries matches the commit count.
- Every actual commit appears exactly once.
- Every hash resolves to a commit on the branch.
- No `unknown`, `<hash>`, or other placeholders remain.
- The commit subjects match Git exactly.
- The three top-level bullets describe the pull request’s net effect.
- The testing section lists changed test files separately from locally executed tests.
- Any named executed test files or suites are supported by observed command output or an explicitly supplied result.
- Each reported suite has an explicit result: `passed`, `failed`, or `not run locally`.
- Backend execution is labelled `Backend tests`, not simply `Tests`.
- No test execution is inferred from application files or hook configuration alone.
- No later branch commits are missing from the summary.
