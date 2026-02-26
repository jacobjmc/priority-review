---
description: Review a specific commit with P0-P3 priority bug detection
---

Review a specific commit. If no SHA provided, show recent commits for selection.

## Context

- Working directory: !`pwd`
- Git status: !`git status --short`

## Instructions

**Step 1: Check if commit SHA was provided as argument**

If user provided a SHA (e.g., `/priority-review-commit abc123`), use that SHA and skip to Step 3.

If no SHA provided, continue to Step 2.

**Step 2: Show commits for selection**

Run: `git log --oneline --format="%h  %s  (%cr)" -n 15`

Format output as a numbered table and ask user to pick:

| #  | SHA     | Message                                    | When           |
|----|---------|--------------------------------------------|----------------|
| 1  | abc123  | feat: example commit message               | 2 hours ago    |
| 2  | def456  | fix: another example                       | 3 hours ago    |
...

Say: "Reply with a number (1-15) or commit SHA to review, or 'cancel' to abort."

**STOP and wait for user response.**

**Step 3: Dispatch review (optional subagent)**

Once you have a commit SHA, try to dispatch the `priority-reviewer` subagent with:

- Git commands to run: `git show {sha} --stat`, `git show {sha}`, `git log -1 --format="%s%n%n%b" {sha}`
- Working directory from context above
- Check for `.priority-review-rules.json` custom rules
- Return findings in the `priority-review` skill format (P0-P3 + confidence + `Impact` + `Why` + summary, with optional `Needs verification`)

If the `priority-reviewer` agent is not available (not registered or not found), fall back: load the `priority-review` skill and perform the review directly in your current context, following the skill's workflow and output format.

Note: without the subagent, the review runs in the primary agent's context. The validation layer described in Step 4 still applies but provides less independent verification.

**Step 4: Validate findings before responding to user**

IMPORTANT: Do NOT show the subagent's response directly to the user.

When you receive the review findings:

1. Read each reported finding (`[P?][high|medium]`) and each `Needs verification` item if present
2. For each item, read the referenced file/line anchor (and nearby context as needed)
3. Verify the issue is real, caused by the reviewed change, and severity is reasonable
4. Discard false positives, speculative claims, or inaccurate descriptions

**Step 5: Report outcome and offer fixes**

After validation:

- If valid P0/P1 issues exist: Report blockers and offer to create a fix commit
- If only valid P2/P3 issues exist: Present non-blocking findings and ask whether the user wants fixes
- If no valid issues (or all were false positives): Tell the user `No actionable findings` and that the commit looks good
