---
name: prd-to-issues
description: Split a PRD into small, self-contained local issue files plus a status index. Use when the user wants a PRD broken into implementable slices for future AI sessions, staged work, or a clear issue queue, especially for PRDs produced by to-prd.
---

# PRD to Issues

Turn a PRD into issue files a fresh, cheaper model can implement one at a time.

## Workflow

1. Locate the PRD. Use the user's path when given; otherwise infer the nearest `prd.md` from the current artifact folder, falling back to `artifacts/prd.md`.
2. Read the PRD. Do not rewrite or "improve" it.
3. Inspect the repo only when it improves issue boundaries. Focus on owners, affected surfaces, existing patterns, tests, and compatibility paths.
4. Slice from the PRD's `Implementation Plan`; sharpen scope with `Behavior Contract`, `Ownership And Reuse`, `Compatibility And Cleanup`, `Risks`, and `Validation`.
5. Ask only when a missing answer changes issue boundaries, execution order, compatibility behavior, or whether a slice is safe.
6. Write issues to `<prd-directory>/issues/NNN-short-title.md` and update `<prd-directory>/issues/index.md`.

If the user asked to review the breakdown first, show it and wait. Otherwise create the files directly unless a material blocker remains.

Do not use GitHub commands. When writing under `artifacts/`, do not inspect Git state just to report those files.

## Slicing Rules

- Prefer end-to-end behavior slices.
- Make each issue narrow enough to build, review, and validate independently.
- Allow an issue to touch multiple layers only where the behavior requires it.
- Avoid layer-only issues unless no useful behavior slice exists.
- Put blockers before dependents and number from the next available issue number.
- Preserve PRD scope, non-goals, unchanged behavior, compatibility shims, redirects, wrappers, and cleanup boundaries.
- Include enough local context in each issue that a new session can start from the issue file and repo, using the parent PRD as backup.

Classify each issue:

- `AFK`: enough PRD and repo context exists to implement without another human decision.
- `Review`: implementable, but risky enough to review before merge because it touches auth, data migration, destructive cleanup, or broad shared behavior.
- `HITL`: blocked by a real decision, access need, credential, environment, or unresolved product/architecture choice.

Prefer `AFK` when the PRD and repo provide enough information.

## Issue File

Use this shape, omitting empty sections:

```markdown
# <Title>

Type: AFK | Review | HITL
Parent PRD: `<relative-or-absolute-prd-path>`
Depends on: None | `<issue-file>` | user decision | environment/access

## What To Build

Describe the narrow behavior change. Name the main routes, screens, APIs, components, services, jobs, adapters, models, or tests involved when known.

## Boundaries

State what must stay unchanged, what must not be touched, what compatibility shim/redirect/wrapper must remain, and what cleanup this issue owns.

## Implementation Notes

Include only notes that prevent likely mistakes: repo patterns to reuse, existing helpers, ownership guidance, data-shape constraints, line-ending requirements from the PRD, or risky files/surfaces.

## Acceptance Criteria

- [ ] Observable behavior works.
- [ ] Required compatibility or non-regression behavior still works.
- [ ] The cheapest useful validation check passes.

## PRD Coverage

Name the relevant PRD sections or bullets this issue implements.

## Open Questions Or Blockers

Use `None` or state the specific decision, dependency, or access need.
```

Do not store workflow status in issue files; status belongs only in the index.

## Index File

Always create or update `issues/index.md`. It is the re-entry point for future sessions.

The index must:

- list every issue exactly once in recommended execution order
- store the only workflow status for each issue
- use statuses `Todo`, `In Progress`, `Blocked`, and `Done`
- mark new issues `Todo` when blockers are satisfied, otherwise `Blocked`
- state the next issue explicitly
- keep completed issues listed as `Done`

The next issue is the first `In Progress` or `Todo` issue whose blockers are satisfied. If several are ready, prefer the earliest `AFK` issue.

Use this compact table:

```markdown
## Parent PRD

`<prd-path>`

## Next Issue

`<issue-file>` or why no issue is ready.

## Issues

| Order | Status | Type | Issue | File | Blocked by | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| 001 | Todo | AFK | <title> | `<file>` | None | <confidence or risk note> |
```

## Final Response

After writing files, summarize the PRD used, how many issues were created or updated, the next issue, any `HITL` blockers, and any high-risk compatibility areas.
