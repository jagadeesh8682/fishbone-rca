# Contributing to fishbone-rca

Thanks for helping improve this skill.

## Good first contributions

- A new playbook section in `plugins/fishbone-rca/skills/fishbone-rca/references/playbooks.md` (a language, framework, database or cloud provider)
- More common causes in `references/categories.md`
- Fixes to commands that are wrong or outdated
- A real-world (anonymised) RCA that the skill handled well or badly

## Guidelines

1. Keep `SKILL.md` focused on the workflow; put long checklists and commands in `references/`.
2. Every command must be copy-pasteable and use `<placeholders>` for values.
3. Keep RCA guidance blameless and evidence-based.
4. Bump `version` in `plugins/fishbone-rca/.claude-plugin/plugin.json` and in the `SKILL.md` metadata for user-visible changes.

## Testing locally

```
/plugin marketplace add ./path/to/fishbone-rca
/plugin install fishbone-rca@jagadeesh-skills
```

Then try it on a real error log or stack trace and check that it follows the steps in `SKILL.md`.

## Pull requests

Fork, create a branch, make your change, and open a PR describing what you changed and why.
