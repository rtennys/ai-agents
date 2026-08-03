# AI Agent Skills

Personal, reusable skills for AI coding agents. Each skill lives in its own
directory under `skills/` and defines its behavior in a `SKILL.md` file.

## Skills

- `grill-me` - Stress-test a design through a focused interview before writing
  a product requirements document.
- `to-prd` - Turn a brief, design discussion, or rough draft into a concise,
  implementation-ready product requirements document.
- `prd-to-issues` - Split a product requirements document into small,
  self-contained implementation issues and a status index.
- `do-next-issue` - Implement the next unresolved issue from an index, validate
  it, and stop after that single issue.

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
