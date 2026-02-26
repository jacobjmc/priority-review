---
description: Review uncommitted changes with P0-P3 priority bug detection
---

Review all uncommitted changes in the current repository.

## Context

- Working directory: !`pwd`
- Git status: !`git status --short`
- Has staged changes: !`git diff --cached --stat`
- Has unstaged changes: !`git diff --stat`

## Instructions

**Step 1: Verify there are changes to review**

If git status shows no changes, inform user: "No uncommitted changes to review."

**Step 2: Dispatch review (optional subagent)**

Try to dispatch the `priority-reviewer` subagent with:

- Git commands to run in parallel: `git status --porcelain`, `git diff`, `git diff --cached`
- Working directory from context above
- Check for `.priority-review-rules.json` custom rules
- Return findings in the `priority-review` skill format (P0-P3 + confidence + `Impact` + `Why` + summary, with optional `Needs verification`)

If the `priority-reviewer` agent is not available (not registered or not found), fall back: load the `priority-review` skill and perform the review directly in your current context, following the skill's workflow and output format.

Note: without the subagent, the review runs in the primary agent's context. The validation layer described in Step 3 still applies but provides less independent verification.

**Step 3: Validate findings before responding to user**

IMPORTANT: Do NOT show the subagent's response directly to the user.

When you receive the review findings:

1. Read each reported finding (`[P?][high|medium]`) and each `Needs verification` item if present
2. For each item, read the referenced file/line anchor (and nearby context as needed)
3. Verify the issue is real, caused by the reviewed change, and severity is reasonable
4. Discard false positives, speculative claims, or inaccurate descriptions

**Step 4: Report outcome and offer fixes**

After validation:

- If valid P0/P1 issues exist: Report blockers first and offer to fix them immediately
- If only valid P2/P3 issues exist: Present non-blocking findings and ask whether the user wants fixes
- If no valid issues (or all were false positives): Tell the user `No actionable findings` and that the code looks ready to commit
