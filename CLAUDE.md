# oyvij/skills

Øyvind Johannessen's personal agent skills, packaged as a Claude Code plugin marketplace so they can be installed on any machine.

## Layout

- `.claude-plugin/marketplace.json`: the `oyvij` marketplace, listing one plugin.
- `.claude-plugin/plugin.json`: the `oyvij-skills` plugin, rooted at the repo root.
- `skills/<skill-name>/SKILL.md`: one folder per skill. Claude Code discovers skills in `skills/` automatically, so adding a skill needs no manifest change.
- `agents/<agent-name>.md`: subagents that skills dispatch. Claude Code discovers them in `agents/` automatically.

## Adding a skill

1. Create `skills/<skill-name>/SKILL.md` with `name` and `description` frontmatter. The folder name matches `name`.
2. Put supporting files (templates, scripts, references) next to `SKILL.md` and link them relatively.
3. Use `/writing-for-agents` when writing or editing skill content.
4. Run `claude plugin validate .` before committing.

## Agent skills

### Issue tracker

Issues are tracked in GitHub Issues on `oyvij/skills` via the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Domain docs

Single-context: one root `CONTEXT.md` glossary, no ADRs. See `docs/agents/domain.md`.
