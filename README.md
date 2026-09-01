# AI Agent Skills

Personal, reusable skills for AI coding agents. Each skill lives in its own
directory under `skills/` and defines its behavior in a `SKILL.md` file.

## Skills

The four skills form one pipeline. Each hands a named artifact to the next.

| Skill | Consumes | Produces |
| --- | --- | --- |
| `grill-me` | A hand-written prompt or brief | A `Grilled Design` block |
| `to-prd` | A `Grilled Design` block or discussion | `prd.md`, plus a one-session or split-into-issues recommendation |
| `prd-to-issues` | `prd.md` | `issues/NNN-*.md` and `issues/index.md` |
| `do-next-issue` | `issues/index.md` | One implemented, tested, committed issue |

- `grill-me` - Stress-test a design through a focused interview before writing
  a product requirements document. Answers what it can from the repo first,
  then asks only the questions that change the outcome.
- `to-prd` - Turn a brief, design discussion, or rough draft into a concise,
  implementation-ready product requirements document under an explicit length
  budget.
- `prd-to-issues` - Split a product requirements document into small,
  self-contained implementation issues and a status index.
- `do-next-issue` - Implement the next unresolved issue from an index, validate
  it, and stop after that single issue.

Issue files and the index are local progress-tracking state and are never
committed. Neither line numbers nor stale code locations belong in any artifact
these skills produce; code is referenced by symbol name or greppable
expression throughout.

## Structure

```text
skills/
  <skill-name>/
    SKILL.md
```

The repository intentionally contains no root-level or model-specific agent
instruction file. Individual skills apply only when they are selected by a
compatible agent environment.

## License

Released into the public domain under [The Unlicense](LICENSE).
