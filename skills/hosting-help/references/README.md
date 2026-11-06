# Hosting Help references

Reference documentation for the **hosting-help** Skill. The agent answers
how-to and conceptual questions from these files only.

## Files

| File | Purpose |
| --- | --- |
| `hosting-help-kb.md` | Primary KB — dashboard + CLI answers, supported stacks (`nodejs`, `fastapi`), MVP limits (no databases), lifecycle, errors |

## Conventions

- One `##` section per topic in each KB file.
- Task-style topics use the six-part answer format defined in `../SKILL.md`:
  dashboard clicks first, then CLI, then redeploy caveat and security notes.
- CLI commands must match shipped `netsol-cli` commands exactly.
