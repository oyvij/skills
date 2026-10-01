# Domain Docs

How the engineering skills should consume this repo's domain documentation.

This is a skills-only repo: single-context, with no ADRs.

## Before exploring, read this

- **`CONTEXT.md`** at the repo root: the glossary of terms used across the skills.

If it doesn't exist, **proceed silently**. Don't flag its absence; don't suggest creating it upfront. The `/domain-modeling` skill creates it lazily when terms actually get resolved.

## File structure

```
/
├── CONTEXT.md
└── skills/
    └── <skill-name>/SKILL.md
```

## Use the glossary's vocabulary

When your output names a domain concept (in a skill name, a skill description, an issue title), use the term as defined in `CONTEXT.md`. Don't drift to synonyms the glossary explicitly avoids.

If the concept you need isn't in the glossary yet, that's a signal: either you're inventing language the project doesn't use (reconsider) or there's a real gap (note it for `/domain-modeling`).
