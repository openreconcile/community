# Portable agent context

Tool-agnostic copies of the standing rules. Cursor loads `.cursor/rules/` and
`.cursor/skills/`. Claude Code / Codex / others should vendor this directory
plus `AGENTS.md`.

| File | Same content as |
|---|---|
| `CONSTITUTION.md` | `.cursor/rules/00-constitution.mdc` (body) |
| `CONTROLLER.md` | `.cursor/rules/10-controller-conventions.mdc` (body) |
| `TESTING.md` | `.cursor/rules/30-testing-reconcilebench.mdc` (body) |

Skills (`../.cursor/skills/`) are the workflow layer. Rules are constraints.
Both are required.

Hero user: an MLE describing an existing ML tool in English. The `new-operator`
Skill is the front door; `openreconcile new` is the same intake without an IDE.
