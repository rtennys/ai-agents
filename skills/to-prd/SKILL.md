---
name: to-prd
description: Write a concise, self-contained Product Requirements Document (PRD) from a client brief, grilled design, implementation discussion, or rough PRD draft. Use when the user says "write a PRD", "turn this into a PRD", or wants a plan another AI session can implement with minimal extra context, especially before coding or issue-splitting. Consumes the output of grill-me; feeds prd-to-issues.
---

# To PRD

Write a short PRD that a fresh session on a cheaper model can implement cleanly without re-deriving the design.

## Workflow

1. Use the current chat, brief, `Grilled Design` block, draft PRD, and repo inspection as source material.
2. Inspect the repo for implementation-critical facts rather than asking. Dispatch parallel read-only search agents for independent lookups.
3. Ask only about unresolved choices that change scope, behavior, ownership, data shape, compatibility, or validation. When a `Grilled Design` block is present, start from its `Unresolved` list — those questions are already earned. Add a question beyond that list only for a gap in one of those same areas that the repo cannot answer; the interview covered the rest, so re-asking wastes the user's attention.
4. Write the PRD to the user's requested path. If none is given, use `prd.md` beside the source prompt when the prompt is under `artifacts/`; otherwise `artifacts/prd.md`.
5. Recommend one session or `prd-to-issues`, using the rule below.

## Length Budget

A PRD is a briefing, not a specification. Budget by change size:

| Change | Target | Hard cap |
| --- | --- | --- |
| Single behavior, one owner | 250–400 words | 600 |
| Several coupled behaviors | 400–700 words | 1000 |
| Multi-slice, issue-split expected | 700–1000 words | 1400 |

Over the cap means you are specifying implementation, not requirements. Cut, do not compress into denser prose.

Enforce this with three rules:

- **State each fact once.** If the owner is named in `Implementation Plan`, it does not reappear in `Problem And Solution`.
- **Omit conditional sections by default.** `Compatibility And Cleanup` and `Risks` appear only when there is something real to say. "No known risks" is not a risk section; delete the heading.
- **No filler.** No restating the section heading as its first sentence, no "this document describes", no summary of what you are about to say.

Never include code blocks, written-out implementation code, GitHub issue text, brainstorming, or file-by-file instructions. Short inline expressions quoted as grep anchors are not code snippets — those are required, see below.

## Citing Code

Never cite a line number — no `L123`, no `L123-456`, no "at line 123", no `file.cs:123`. Refer to code by symbol name, by a named position inside a symbol, or by a short quoted expression the reader can grep. Line numbers go stale between writing the PRD and implementing it, and anything stale here propagates into every issue split from this PRD.

## What To Capture

Enough that a new session never has to guess:

- Problem, user-visible goal, and intended system outcome
- Scope, non-goals, and behavior that must stay unchanged
- Current and target owner of the behavior
- Primary routes, screens, APIs, components, services, jobs, adapters, or data models
- Existing helpers, patterns, and prior tests to reuse
- Compatibility paths: legacy routes, redirects, shims, flags, permissions, blocked states, downstream consumers
- Data shape, migration, cleanup, rollback, and risk notes when they apply
- Minimum manual and automated validation

## Maintainability Guidance

Steer implementation toward small, boring code:

- Prefer local changes in the existing owner over new abstractions.
- Add an abstraction only when it removes real duplication or centralizes a rule that already has multiple callers.
- Reuse established repo patterns, helpers, components, and test style.
- Split work into end-to-end slices, not layers.
- State what should be deleted, left in place, or kept temporarily as a compatibility shim, redirect, or wrapper.

## Template

Core sections always appear. Conditional sections appear only when they carry content.

```markdown
## Problem And Solution

What is wrong today, and the shape of the fix. Two short paragraphs at most.

## Scope And Non-Goals

## Behavior Contract

What must be true after the change, and what must still be true that was true before.

## Implementation Plan

End-to-end slices in execution order. Name the current and target owner, the surfaces touched, and the existing helpers, patterns, and tests to reuse.

## Compatibility And Cleanup   <!-- conditional -->

## Risks   <!-- conditional -->

## Validation

The cheapest checks that prove the behavior contract.

## Open Decisions   <!-- conditional -->

Decisions still owed by a human, each naming who or what resolves it.
```

Omit `Open Decisions` when there are none. When there are any, the section is mandatory — `prd-to-issues` reads only the file, and an unresolved decision that lives solely in the chat becomes an issue misclassified `AFK` and implemented on a guess.

## Session Sizing

Close the PRD by recommending one of two paths.

**One session** when all of these hold:

- one end-to-end slice, or two that cannot be usefully separated
- a single primary owner
- no data migration, no destructive cleanup, no auth or permission surface
- no unresolved decision that blocks any part of the work

**Run `prd-to-issues`** when any of these hold:

- three or more independently shippable slices
- a slice that must land and be verified before another can start
- any slice classified `HITL` — blocked on a human decision, credential, or environment
- migration, destructive cleanup, or a security boundary that deserves isolated review

When it is close, prefer issues. A queue that turns out to be one session costs a little ceremony; a session that turns out to be three loses context partway through.

If the PRD still carries an unresolved decision, record it under `Open Decisions` and say so plainly in your reply. Do not write around it.
