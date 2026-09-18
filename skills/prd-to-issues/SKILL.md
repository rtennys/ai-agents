---
name: prd-to-issues
description: Split a PRD into small, self-contained local issue files plus a status index. Use when the user says "split this PRD", "break this into issues", or wants a PRD turned into implementable slices for future AI sessions, staged work, or a clear issue queue — especially for PRDs produced by to-prd. Feeds do-next-issue.
---

# PRD to Issues

Turn a PRD into issue files a fresh session on a cheaper model can implement one at a time.

## Workflow

1. Locate the PRD. Use the user's path when given; otherwise infer the nearest `prd.md` from the current artifact folder, falling back to `artifacts/prd.md`.
2. Read the PRD. Do not rewrite or "improve" it.
3. Inspect the repo only when it sharpens issue boundaries: owners, affected surfaces, existing patterns, tests, compatibility paths. Dispatch parallel read-only search agents for independent lookups.
4. Read `Problem And Solution` for framing — every issue must serve it. Slice from `Implementation Plan`; sharpen scope with `Behavior Contract`, `Scope And Non-Goals`, `Compatibility And Cleanup`, `Risks`, and `Validation`. Older PRDs may carry an `Ownership And Reuse` section — read it as part of the implementation plan.
5. Ask only when a missing answer changes issue boundaries, execution order, compatibility behavior, or whether a slice is safe.
6. Write issues to `<prd-directory>/issues/NNN-short-title.md` and create or update `<prd-directory>/issues/index.md`.

If the user asked to review the breakdown first, show it and wait. Otherwise create the files directly unless a material blocker remains.

Do not use GitHub commands.

Issue files and the index are local progress-tracking state, never committed. Confirm the issues directory is gitignored; if it is not, add the entry before writing the files, and tell the user you did. Do not otherwise inspect Git state just to report on these files.

## Citing Code

Never cite a line number. No `L123`, no `L123-456`, no "at line 123", no `file.cs:123`. Issues are read by sessions that start days or commits later, often after an earlier issue in the same queue has already moved the code. A stale line number is worse than no reference: it points confidently at the wrong thing.

Anchor on things that survive edits, in this order of preference:

1. A symbol name — `ValuationService.GetOptionGreeks`, `OptionModelSelector.SelectModel`.
2. A named position inside a symbol — "the `case (int)PositionType.ListedOption:` arm of `SetPlFields`", "the `if (savedVol != null)` block", "immediately around the `GetOptionGreeks(pos, impliedGreeks);` call".
3. A short quoted expression the reader can grep — `` `var strike = grp.Average(x => x.Vals.StrikePrice)` ``.
4. Just the file path, when the file is small or the symbol name already locates it.

Quote code exactly as it appears rather than paraphrasing, so references stay greppable. A range that describes a whole member is always just that member's name.

When an issue depends on a component an earlier issue in this queue will create, name the intended type, method, and file path, and say the earlier issue owns it — not a location in today's code that the earlier issue is about to change.

## Slicing Rules

- Prefer end-to-end behavior slices.
- Size each issue to one focused session: implementable, reviewable, and validatable in a single sitting without the session running out of context.
- Make each issue narrow enough to build, review, and validate independently.
- Allow an issue to touch multiple layers only where the behavior requires it.
- Avoid layer-only issues unless no useful behavior slice exists.
- Put blockers before dependents and number from the next available issue number.
- Preserve PRD scope, non-goals, unchanged behavior, compatibility shims, redirects, wrappers, and cleanup boundaries.
- Include enough local context in each issue that a new session can start from the issue file and the repo, using the parent PRD only as backup.

Classify each issue:

- `AFK` — enough PRD and repo context exists to implement without another human decision.
- `Review` — implementable, but risky enough to review before merge: auth, data migration, destructive cleanup, or broad shared behavior.
- `HITL` — blocked by a real decision, access need, credential, environment, or unresolved product/architecture choice. Anything the PRD lists under `Open Decisions` produces a `HITL` issue unless the user resolves it during this session.

Prefer `AFK` when the PRD and repo provide enough information.

## Issue File

Use this shape, omitting empty sections:

```markdown
# <Title>

Type: AFK | Review | HITL
Parent PRD: `<relative-or-absolute-prd-path>`
Depends on: None | `<issue-file>` | user decision | environment/access

## What To Build

The narrow behavior change. Name the main routes, screens, APIs, components, services, jobs, adapters, models, or tests involved when known.

## Boundaries

What must stay unchanged, what must not be touched, what compatibility shim/redirect/wrapper must remain, and what cleanup this issue owns.

## Implementation Notes

Only notes that prevent likely mistakes: repo patterns to reuse, existing helpers, ownership guidance, data-shape constraints, risky surfaces. Reference code per `Citing Code` — symbols and quoted expressions, never line numbers.

## Acceptance Criteria

- [ ] Observable behavior works.
- [ ] Required compatibility or non-regression behavior still works.
- [ ] The cheapest useful validation check passes.

## PRD Coverage

The PRD sections or bullets this issue implements.

## Open Questions Or Blockers

`None`, or the specific decision, dependency, or access need.
```

Do not store queue status — `Todo`, `In Progress`, `Blocked`, `Done` — in an issue file. That lives only in the index. The acceptance checkboxes are a within-issue implementation checklist for the session doing the work, not queue state, and no later session reads them.

`Depends on` in the issue header is documentation. The index's `Blocked by` column is authoritative; when the two disagree, the index wins.

## Index File

Always create or update `issues/index.md`. It is the re-entry point for every later session, and `do-next-issue` reads it as a contract.

The index must:

- list every issue exactly once, in recommended execution order
- hold the only workflow status for each issue: `Todo`, `In Progress`, `Blocked`, or `Done`
- mark new issues `Todo` when blockers are satisfied, otherwise `Blocked`
- list in `Blocked by` only issues that are not yet `Done`, so the column never names a finished blocker
- name the next issue explicitly
- keep completed issues listed as `Done`

The next issue is the first `In Progress` or `Todo` issue whose blockers are satisfied. If several are ready, prefer the earliest `AFK` issue.

Rows appear in execution order, so no separate order column is needed:

```markdown
## Parent PRD

`<prd-path>`

## Next Issue

`<issue-file>`, or why no issue is ready.

## Issues

| Status | Type | Issue | Blocked by | Notes |
| --- | --- | --- | --- | --- |
| Todo | AFK | `001-short-title.md` — <title> | None | <confidence or risk note> |
```

## Final Response

Summarize the PRD used, how many issues were created or updated, the next issue, any `HITL` blockers, and any high-risk compatibility areas. Offer to run `do-next-issue` in a fresh session.
