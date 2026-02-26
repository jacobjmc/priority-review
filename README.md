# priority-review

Evidence-based code review skill for AI coding agents with P0-P3 priority defect review.

Works with Claude Code, OpenCode, Codex, Cursor, Gemini CLI, and [40+ other agents](https://skills.sh/docs) that support the open agent skills format.

## Quick Install (Core Skill - Portable)

```bash
npx skills add jacobjmc/priority-review
```

This installs the core `priority-review` skill. Your agent will use it automatically when reviewing code.

This is the stable, portable path and works across agents that support the open skills format.

## Optional Plugin Integrations (Commands + Subagent)

The quick install gives you the canonical review methodology. The optional plugin integration adds slash commands and a dedicated reviewer subagent as convenience layers on top of the core skill.

This repository currently bundles `skills/`, `agents/`, and `commands/` in a plugin-compatible layout via `.claude-plugin/plugin.json` (no hooks are included yet).

### Which install should I use?

- Use **Quick Install (Core Skill - Portable)** if you want the standard `skills.sh` path that works across many agents.
- Use **Optional Plugin Integrations** only if you want OpenCode/Claude-specific extras like slash commands and a dedicated reviewer subagent.
- The optional integration includes the same core skill plus extra wrappers; it does not replace the portable skill.

### What you get

| Feature | Quick Install | Optional Plugin |
|---------|:------------:|:---------------:|
| P0-P3 review methodology | Yes | Yes |
| Custom rules (`.priority-review-rules.json`) | Yes | Yes |
| `/priority-review` commands | - | Yes |
| Dedicated reviewer subagent | - | Yes |
| Validation layer (subagent findings verified by primary agent) | - | Yes |

### Architecture (source of truth)

- `skills/priority-review/SKILL.md` is the canonical review spec (P0-P3, confidence, evidence standard, user-facing output format).
- `agents/priority-reviewer.md` is an optional subagent adapter for isolated review execution.
- `commands/*.md` are optional orchestration wrappers (context gathering, dispatch, validation, UX).
- The optional layers should follow the skill's definitions and output contract; they are convenience layers, not separate review systems.

### Claude Code

Install as a plugin:

```bash
# Clone the repo
git clone https://github.com/jacobjmc/priority-review.git ~/.config/priority-review

# Register the plugin directory
claude plugin add ~/.config/priority-review
```

The `.claude-plugin/plugin.json` manifest registers the skill, agent, and commands automatically.

#### Uninstall / Cleanup (Claude Code plugin)

To remove the Claude Code plugin integration:

```bash
claude plugin uninstall priority-review
```

If you installed it from a local directory path and want to remove the cloned source repo too, you can optionally delete it after uninstalling:

```bash
rm -rf ~/.config/priority-review
```

### OpenCode

```bash
# Clone the repo
git clone https://github.com/jacobjmc/priority-review.git ~/.config/opencode/priority-review

# Symlink the skill
ln -s ~/.config/opencode/priority-review/skills/priority-review \
      ~/.config/opencode/skill/priority-review

# Copy the agent (add OpenCode-specific frontmatter - see below)
cp ~/.config/opencode/priority-review/agents/priority-reviewer.md \
   ~/.config/opencode/agent/priority-reviewer.md

# Copy the commands
cp ~/.config/opencode/priority-review/commands/*.md \
   ~/.config/opencode/command/
```

The cloned folder (`~/.config/opencode/priority-review`) is used as the source for the skill, agent, and commands.

- Keep it if you want to update later with `git pull`.
- Remove it only if you also remove the installed symlink/copied files (otherwise the skill symlink will break).

After copying the agent file, add the following OpenCode-specific frontmatter to `~/.config/opencode/agent/priority-reviewer.md`:

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

#### Paste Into Your AI Agent (OpenCode optional integration)

If you want your AI agent to install the optional OpenCode integration for you, paste this.

This installs the OpenCode convenience layer (commands + subagent) on top of the same core `priority-review` skill. If you only want the portable standard skill, use `npx skills add jacobjmc/priority-review` instead.

Paste this:

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
6. Add this frontmatter to the top of `~/.config/opencode/agent/priority-reviewer.md` if it is not already present:

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

7. Verify installation by checking these files exist:
   - `~/.config/opencode/skill/priority-review/SKILL.md`
   - `~/.config/opencode/agent/priority-reviewer.md`
   - `~/.config/opencode/command/priority-review.md`
8. Summarize what you installed and any issues.
9. Explain whether the cloned repo folder should be kept for future updates, and what would break if it is deleted.

Do not remove unrelated files.
```

#### Uninstall / Cleanup (OpenCode optional integration)

To remove the optional OpenCode integration:

```bash
# Remove the installed skill symlink
rm ~/.config/opencode/skill/priority-review

# Remove the installed subagent
rm ~/.config/opencode/agent/priority-reviewer.md

# Remove installed slash commands
rm ~/.config/opencode/command/priority-review.md
rm ~/.config/opencode/command/priority-review-pr.md
rm ~/.config/opencode/command/priority-review-commit.md
rm ~/.config/opencode/command/priority-review-custom.md
```

Optional: remove the cloned source repo used for updates/copying:

```bash
rm -rf ~/.config/opencode/priority-review
```

Notes:

- Only remove the cloned repo after removing the skill symlink (or replacing it), otherwise `~/.config/opencode/skill/priority-review` will become a broken symlink.
- If you installed a modified local copy of the agent or commands, remove only the files related to `priority-review`.

### Codex

```bash
npx skills add jacobjmc/priority-review -a codex
```

Or manually:

```bash
git clone https://github.com/jacobjmc/priority-review.git ~/.codex/priority-review
ln -s ~/.codex/priority-review/skills/priority-review ~/.agents/skills/priority-review
```

### Gemini CLI

```bash
npx skills add jacobjmc/priority-review -a gemini-cli
```

## Usage

If you installed the optional plugin integrations, use the slash commands below. Otherwise use the skill-only prompts in the next section.

### Review uncommitted changes

```
/priority-review
```

### Review a specific commit

```
/priority-review-commit [sha]
```

### Review PR / branch against base

```
/priority-review-pr [base-branch]
```

### Review with custom focus

```
/priority-review-custom "check for accessibility issues" src/components/
```

### Skill-only usage (no commands)

If you installed via `npx skills add` without the optional plugin integrations, your default agent can still use the skill. Ask it to review a diff/commit/PR with P0-P3 priorities and it will load the `priority-review` skill automatically when relevant.

Example prompts:

- "Review my current diff with P0-P3 severity. Only report actionable defects."
- "Do a priority review of this commit for security, correctness, and reliability issues."

## Priority Levels

| Priority | Severity | Action |
|----------|----------|--------|
| **P0** | Critical | Must fix before commit (security, crashes, data corruption) |
| **P1** | High | Should fix before merge (common-path correctness, error handling, realistic state/concurrency bugs) |
| **P2** | Medium | Consider fixing (edge-case correctness, validation/runtime type gaps, concrete perf regressions) |
| **P3** | Low | Low-severity correctness or maintainability risk introduced by the change |

### Output format (current)

- Findings are reported by priority (`P0` -> `P3`) with confidence labels (`high` / `medium`).
- Low-confidence concerns should be listed under `Needs verification` instead of being reported as blockers.
- Reviews can end with `No actionable findings` when no credible defects are identified.

## Custom Rules

Create `.priority-review-rules.json` in your repository root:

```json
{
  "rules": [
    {
      "id": "no-console-log",
      "title": "No console.log in production code",
      "description": "Console.log statements should be removed before commit",
      "severity": "P2",
      "pattern": "console.log"
    },
    {
      "id": "no-any-type",
      "title": "Avoid 'any' type in TypeScript",
      "description": "Use proper types instead of 'any'",
      "severity": "P3",
      "pattern": ": any"
    }
  ]
}
```

## How It Works

1. **Dispatch (optional)**: A slash command can dispatch the `priority-reviewer` subagent with diff context. If no subagent is installed, the primary agent loads the `priority-review` skill directly.
2. **Analysis**: The reviewer follows the core skill's rubric to triage risk signals and perform targeted, bounded investigation.
3. **Report**: The reviewer returns structured findings in the core skill format (P0-P3 + confidence + impact + why + summary).
4. **Validation**: The main agent verifies each reported finding against the referenced code before responding.
5. **Action**: The main agent and user decide whether to fix blockers now or defer non-blocking findings.

With skill-only install (no commands/subagent), the primary agent performs the review directly using the same canonical `priority-review` skill format.

## Repository Structure

```
skills/
  priority-review/
    SKILL.md                   # Core review methodology (installed by npx skills)

agents/
  priority-reviewer.md         # Subagent definition (optional plugin install)

commands/
  priority-review.md           # Review uncommitted changes
  priority-review-commit.md    # Review specific commit
  priority-review-custom.md    # Review with custom focus
  priority-review-pr.md        # Review PR / branch

.claude-plugin/
  plugin.json                  # Claude Code plugin manifest
```

## License

MIT
