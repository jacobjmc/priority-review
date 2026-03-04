# priority-review

Code review skill for AI agents. Flags security, correctness, and reliability defects using P0-P3 severity.

Works with Claude Code, OpenCode, Codex, Gemini CLI, and [40+ other agents](https://skills.sh/docs) that support the open agent skills format.

## Install

```bash
npx skills add jacobjmc/priority-review
```

Your agent loads the skill automatically when reviewing code.

## What the skill does

It gives your agent a rubric for reviewing diffs. Findings are triaged into four levels:

| Priority | Meaning | Examples |
|----------|---------|----------|
| P0 | Must fix before commit | Security holes, crashes, data corruption |
| P1 | Should fix before merge | Correctness bugs on common paths, missing error handling |
| P2 | Worth fixing | Edge-case bugs, validation gaps, perf regressions |
| P3 | Minor | Low-severity correctness or maintainability risks |

Each finding has a confidence label (high or medium). Low-confidence concerns go under "Needs verification" instead of being reported as blockers. Reviews can end with "No actionable findings."

## Usage

Ask your agent to review code. It loads the skill when relevant.

Example prompts:

- "Review my current diff with P0-P3 severity."
- "Do a priority review of this commit for security and correctness issues."

## Custom rules

Add `.priority-review-rules.json` to your repo root:

```json
{
  "rules": [
    {
      "id": "no-console-log",
      "title": "No console.log in production code",
      "description": "Console.log statements should be removed before commit",
      "severity": "P2",
      "pattern": "console.log"
    }
  ]
}
```

## Optional: plugin integration (OpenCode / Claude Code)

The skill install above is all most people need. If you also want slash commands and a dedicated reviewer subagent, there is an optional plugin layer.

### What the plugin adds

| Feature | Skill only | With plugin |
|---------|:----------:|:-----------:|
| P0-P3 review methodology | Yes | Yes |
| Custom rules | Yes | Yes |
| `/priority-review` commands | - | Yes |
| Dedicated reviewer subagent | - | Yes |
| Validation (subagent findings verified by primary agent) | - | Yes |

### Slash commands

```
/priority-review              # uncommitted changes
/priority-review-commit [sha] # specific commit
/priority-review-pr [base]    # PR / branch against base
/priority-review-custom "focus" path/  # custom scope
```

### Claude Code

```bash
git clone https://github.com/jacobjmc/priority-review.git ~/.config/priority-review
claude plugin add ~/.config/priority-review
```

Uninstall:

```bash
claude plugin uninstall priority-review
rm -rf ~/.config/priority-review  # optional: remove source
```

### OpenCode

```bash
git clone https://github.com/jacobjmc/priority-review.git ~/.config/opencode/priority-review

# Symlink skill
ln -s ~/.config/opencode/priority-review/skills/priority-review \
      ~/.config/opencode/skill/priority-review

# Copy agent and commands
cp ~/.config/opencode/priority-review/agents/priority-reviewer.md \
   ~/.config/opencode/agent/priority-reviewer.md
cp ~/.config/opencode/priority-review/commands/*.md \
   ~/.config/opencode/command/
```

Add this frontmatter to `~/.config/opencode/agent/priority-reviewer.md`:

```yaml
---
name: priority-reviewer
description: Evidence-based code reviewer with P0-P3 priority system. Analyzes git diffs for actionable security, correctness, and reliability defects. Returns structured findings for the main agent to validate.
mode: subagent
temperature: 0.1
tools:
  write: false
  edit: false
permission:
  edit: deny
  write: deny
  bash:
    "*": deny
    "git *": allow
    "ls *": allow
    "cat *": allow
---
```

<details>
<summary>Paste-in prompt (have your agent install it for you)</summary>

```text
Install the `priority-review` optional OpenCode integration on this machine.

Please do the following:
1. Clone `https://github.com/jacobjmc/priority-review.git` into `~/.config/opencode/priority-review` (or update it if already present).
2. Ensure these directories exist: `~/.config/opencode/skill`, `~/.config/opencode/agent`, `~/.config/opencode/command`.
3. Install the core skill by symlinking:
   `~/.config/opencode/priority-review/skills/priority-review` -> `~/.config/opencode/skill/priority-review`
4. Copy the subagent file:
   `~/.config/opencode/priority-review/agents/priority-reviewer.md` -> `~/.config/opencode/agent/priority-reviewer.md`
5. Copy command files from `~/.config/opencode/priority-review/commands/*.md` into `~/.config/opencode/command/`.
6. Add the frontmatter block from the README to the top of `~/.config/opencode/agent/priority-reviewer.md` if not already present.
7. Verify installation by checking these files exist:
   - `~/.config/opencode/skill/priority-review/SKILL.md`
   - `~/.config/opencode/agent/priority-reviewer.md`
   - `~/.config/opencode/command/priority-review.md`
8. Summarize what you installed and any issues.
```

</details>

Uninstall:

```bash
rm ~/.config/opencode/skill/priority-review
rm ~/.config/opencode/agent/priority-reviewer.md
rm ~/.config/opencode/command/priority-review*.md
rm -rf ~/.config/opencode/priority-review  # optional: remove source
```

Note: remove the source repo only after removing the skill symlink, otherwise `~/.config/opencode/skill/priority-review` becomes a broken symlink.

### Codex

```bash
npx skills add jacobjmc/priority-review -a codex
```

### Gemini CLI

```bash
npx skills add jacobjmc/priority-review -a gemini-cli
```

## How it works

1. The agent reads a diff and applies the skill's rubric.
2. It triages risk signals and does targeted investigation.
3. It returns findings with priority, confidence, impact, and reasoning.
4. With the plugin, the primary agent verifies each finding against the code before responding.

## Repository structure

```
skills/priority-review/SKILL.md    # Core review methodology
agents/priority-reviewer.md        # Subagent (optional plugin)
commands/*.md                      # Slash commands (optional plugin)
.claude-plugin/plugin.json         # Claude Code plugin manifest
```

## License

MIT
