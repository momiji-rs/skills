# Agent Skills for momiji-rs tools

[Agent skills](https://agentskills.io) for the momiji-rs command-line tools. Each
skill teaches a coding agent to use one tool through its CLI.

```bash
npx skills add momiji-rs/skills
```

Or, as a Claude Code plugin:

```bash
claude plugin marketplace add momiji-rs/skills
claude plugin install momiji-skills@momiji-skills
```

## Skills

| Skill | Tool | Description |
|-------|------|-------------|
| [termshot](skills/termshot/SKILL.md) | [termshot](https://github.com/momiji-rs/termshot) | Render terminal output (a PTY log, an asciinema cast, or a tmux pane) as a PNG, text, or JSON of the final screen. Screenshot TUIs, check what a screen shows, and make deterministic images for docs and CI. |

## Requirements

Each skill drives its tool's CLI, so install the tools you use. The skill explains
how, and the agent can install a tool itself.

**termshot**: download the binary for your platform from the
[releases](https://github.com/momiji-rs/termshot/releases). Note that
`brew install termshot` installs a different project.

## Layout

```
.claude-plugin/marketplace.json   # the marketplace: one plugin, source "./"
.claude-plugin/plugin.json        # the plugin; its skills are skills/*
skills/<name>/SKILL.md            # one directory per skill
```

To add a skill, create `skills/<name>/SKILL.md`, add a row to the table above,
and bump `version` in both manifests so that installed copies update.
