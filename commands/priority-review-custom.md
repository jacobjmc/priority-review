---
description: Review code with custom focus instructions (e.g., plan review, component structure, specific feature)
---

Review code with custom instructions. Accepts a focus area and optional file/path targets.

## Usage

```
/priority-review-custom <instruction> [file/path...]
```

**Examples:**

- `/priority-review-custom "review this plan for feasibility"` - Review uncommitted changes with plan focus
- `/priority-review-custom "check component structure and prop drilling" src/components/` - Review specific directory
- `/priority-review-custom "verify error handling patterns" src/api/` - Check error handling in API layer
- `/priority-review-custom "review for accessibility issues" src/components/Button.tsx` - Review specific file

## Context

- Working directory: !`pwd`
- Git status: !`git status --short`
- Current branch: !`git branch --show-current`

## Instructions

**Step 1: Parse arguments**

The first quoted string or first argument is the custom review instruction.
Any remaining arguments are file/path targets to focus on.

If no instruction provided, ask: "What would you like me to focus on in this review?"

**Step 2: Determine scope**

| User Provided                     | Scope                          |
| --------------------------------- | ------------------------------ |
| Specific files/paths              | Review only those files        |
| No paths                          | Review all uncommitted changes |
| No uncommitted changes + no paths | Ask user what to review        |

**Step 3: Gather context for targeted files**

If paths were specified:

- Run: `git diff -- {paths...}` and `git diff --cached -- {paths...}`
- If no diff in those paths, read the files directly

If no paths (reviewing uncommitted changes):

- Run: `git diff` and `git diff --cached`

If no changes exist and no paths given:

- Ask user: "No uncommitted changes found. Please specify files/paths to review, or a commit SHA."

**Step 4: Dispatch review with custom focus (optional subagent)**

Try to dispatch the `priority-reviewer` subagent with:

- The custom instruction as the PRIMARY review focus
- Git commands or file contents from Step 3
- Working directory from context above
- Check for `.priority-review-rules.json` custom rules
- Return findings in the `priority-review` skill format (P0-P3 + confidence + `Impact` + `Why` + summary, with optional `Needs verification`)

Pass the custom instruction prominently to the subagent:

```
## Custom Review Focus

The user has requested this specific review focus:

> {custom_instruction}

Prioritize findings related to this focus area. Still report P0/P1 issues even if unrelated to the focus, but weight your analysis toward the requested focus.
```

If the `priority-reviewer` agent is not available (not registered or not found), fall back: load the `priority-review` skill and perform the review directly in your current context, following the skill's workflow and output format. Apply the custom focus instruction to your analysis.

Note: without the subagent, the review runs in the primary agent's context. The validation layer described in Step 5 still applies but provides less independent verification.

**Step 5: Validate findings before responding to user**

IMPORTANT: Do NOT show the subagent's response directly to the user.

When you receive the review findings:

1. Read each reported finding (`[P?][high|medium]`) and each `Needs verification` item if present
2. For each item, read the referenced file/line anchor (and nearby context as needed)
3. Verify the issue is real, caused by the reviewed change, and severity is reasonable
4. Discard false positives, speculative claims, or inaccurate descriptions

**Step 6: Report findings**

After validation:

- If valid P0/P1 issues exist: Report blockers first and offer to fix them immediately
- If only valid P2/P3 issues exist: Present non-blocking findings and ask whether the user wants fixes
- If no valid issues: Tell the user `No actionable findings` for the given focus

Include a summary that references the custom focus:

```
## Review Summary: "{custom_instruction}"

- P0: N
- P1: N
- P2: N
- P3: N
- Outcome: Blocker | Non-blocking findings | No actionable findings

{Assessment relative to the custom focus area}
```
