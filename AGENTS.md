# momiji-rs Skills Repo

A Claude Code plugin marketplace and an `npx skills` source for the momiji-rs
CLI tools. One plugin (`momiji-skills`, `source: "./"`) bundles every skill in
`skills/`; Claude Code invokes them as `/momiji-skills:<name>`.

## Key constraints

- Base each skill on the tool's **latest release**, not on its main branch. Users
  install release binaries. Check flags against the released binary's `--help`.
- Verify every command in a SKILL.md by running it before you commit.
- Keep the `name` and `version` in `.claude-plugin/marketplace.json` and
  `.claude-plugin/plugin.json` the same. Bump `version` with every skill change;
  installed copies update only when it changes.
- Run `claude plugin validate .` after each edit.
