# agent-skills

Skills for coding agents. Each skill is a folder holding a `SKILL.md`: YAML frontmatter with the
skill's `name` and a `description` saying when to use it, then the instructions the agent follows.

## Skills

| Skill                                                         | Use it when                                                                                                                                                                                                                                                 |
| ------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`adversarial-code-review`](adversarial-code-review/SKILL.md) | Reviewing a merge request or pull request, yours or a teammate's. It reviews the problem, the design, the code and the work items the change closes or should reference, challenges every finding before publishing it, and reports each with its evidence. |

## Installing

An agent that supports skills reads them from a skills directory with one folder per skill, each
holding its `SKILL.md`. Copy or link the skill's folder into that directory.

For Claude Code, that is `.claude/skills/` inside a project, or `~/.claude/skills/` for every
project:

```sh
git clone https://github.com/mohripan/agent-skills.git
mkdir -p ~/.claude/skills
cp -r agent-skills/adversarial-code-review ~/.claude/skills/
```

The skill is then offered when its description matches the task, or you can ask for it by name.
For other agents, put the same folder in the skills directory their documentation names.

## Credits

The review instructions in `adversarial-code-review` are by Ahmad
([@nidduzzi](https://github.com/nidduzzi)), including the work-items check he asked to add. This
repository puts them into skill form.
