# My Skills

A repository of reusable [Claude Code](https://claude.com/claude-code) skills. Each skill lives in its own folder and can be installed on its own with `degit`.

## Skill tracker

**Skills available: 1**

| Skill | What it does | How to run | Source |
|---|---|---|---|
| [`pr`](my-skills/pr/SKILL.md) | Takes the current changes through a clean pull request workflow: branch, commit, self-review, raise the PR, then merge only after explicit user approval. | `/pr [optional: PR title, issue number, or existing PR number]` | `my-skills/pr/SKILL.md` |

## Install a skill

Replace `<skill-name>` with a name from the tracker.

**For all your projects:**

```bash
npx degit https://github.com/saifmohammed888/ai-coding/my-skills/<skill-name> ~/.claude/skills/<skill-name>
```

**For the current project only:**

```bash
npx degit https://github.com/saifmohammed888/ai-coding/my-skills/<skill-name> .claude/skills/<skill-name>
```

For example, to install `pr` for all your projects:

```bash
npx degit https://github.com/saifmohammed888/ai-coding/my-skills/pr ~/.claude/skills/pr
```

Then type `/pr` in Claude Code.

## Add a skill

1. Create a folder named after the skill, containing a `SKILL.md`.
2. Start `SKILL.md` with frontmatter:

   ```markdown
   ---
   name: skill-name
   description: What the skill does and when to use it.
   ---
   ```

3. Add a row to the tracker above and update the count.
