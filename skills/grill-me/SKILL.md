---
name: grill-me
description: Stress-test a plan through a focused design interview before a PRD is written. Use when the user says "grill me", "grill this", "poke holes in this", "stress-test this plan", or hands over a rough prompt or brief and wants the design interrogated before implementation. Pairs with to-prd, which consumes this interview's output.
---

# Grill Me

Act as a pre-implementation design reviewer. Interrogate the plan until it is decision-complete, then hand off a written summary that `to-prd` can consume.

This is an interactive review conversation. Do not implement anything, do not enter plan mode, and do not treat the review as a standing objective that survives the user's next request.

## Workflow

### 1. Resolve the input

The source is whatever the user just handed over: a pasted prompt, a file path, an attached brief, or the conversation so far. If they named a path, read it. If nothing was supplied, ask for the plan in one line and stop.

### 2. Answer what you can from the repo

Before asking the user anything, find the answers yourself. Dispatch parallel read-only search agents for independent questions — current owner of the behavior, affected surfaces, existing helpers and patterns, prior tests, live callers. Never ask the user something the repo already answers; every such question spends their attention and buys nothing.

Classify the change while you are in there, because it determines which areas below are worth raising:

- **Modifying existing behavior** — ownership, invariants, compatibility, and cleanup all matter.
- **New behavior on existing surfaces** — ownership and invariants matter; compatibility and cleanup usually do not.
- **Greenfield** — skip ownership, compatibility, and cleanup entirely. Ask about data shape and integration points instead.

Note the helpers, patterns, and prior tests worth reusing as you go. These are discoveries, not questions — you carry them to the handoff without spending a question on them.

### 3. Round one — grouped questions

Post one grouped markdown list, organized by decision area. For each question, state your recommended answer and the reasoning in a clause, not a paragraph.

Cap round one at roughly eight questions. If you have more, you have not done enough repo inspection.

Tell the user to answer only where:

- your recommended answer is wrong
- the decision is genuinely uncertain
- their answer would change scope, ownership, routing, compatibility, data shape, validation, or cleanup

Silence on a question means your recommendation stands. Say so explicitly.

### 4. Follow-ups — one decision at a time

After their answers, ask follow-ups only when an answer changes the implementation path, creates a dependency, reveals a contradiction, or leaves an implementation-critical gap. A follow-up that merely confirms something already settled is noise — skip it.

Ask follow-ups as multiple-choice questions when the tool for that is available in this environment, putting your recommendation first and marking it. Fall back to prose for questions that need a free-form answer.

### 5. Stop and hand off

Stop as soon as the checklist below is resolved, not when you run out of curiosity. Then write out a **Grilled Design** block — the artifact `to-prd` reads:

```markdown
## Grilled Design

**Problem:** what is wrong today, and who it hurts.
**Outcome:** what changes for the user, and the observable signal that it worked.
**Scope:** what is in.
**Non-goals:** what is explicitly out.
**Ownership:** where the behavior lives now; where it lands. (Omit for greenfield.)
**Surfaces:** routes, screens, controllers/actions, components, APIs, services, adapters, jobs, data models.
**Reuse:** existing helpers, patterns, and prior tests the implementation should follow.
**Invariants:** behavior that must not change, and any code that must not be touched.
**Data:** data shape, migration, integration points, rollback. (Omit when nothing persistent or external is involved.)
**Compatibility:** legacy routes, redirects, shims, flags, permissions, blocked states, downstream consumers. (Omit when nothing existing is affected.)
**Cleanup:** what gets deleted, what stays as a temporary shim. (Omit for greenfield.)
**Risks:** what could go wrong that the invariants do not already cover, or `None`.
**Validation:** the cheapest manual and automated checks that prove the outcome.
**Unresolved:** decisions the user deferred, or `None`.
```

Keep each line to a sentence or two. This is a handoff, not the PRD.

Close by offering to run `to-prd`.

## Checklist

Resolve every area that applies to the change class. Skip the rest — do not manufacture a question just to fill a heading.

| Area | Resolved when you can state |
| --- | --- |
| Problem | What is wrong today and who it hurts |
| Outcome | What changes for the user and how you would observe it |
| Scope | What is in, and what is explicitly out |
| Ownership | Where the behavior lives now and where it will live |
| Surfaces | Every route, screen, controller, component, API, service, adapter, job, or model touched |
| Invariants | Behavior that must not change, and code that must not be touched |
| Data | Shape, migration, integration points, and whether a rollback path is needed |
| Compatibility | Legacy paths, redirects, shims, flags, permissions, downstream consumers |
| Cleanup | What is deleted now, what stays temporarily, and what unblocks deleting it later |
| Validation | The cheapest manual and automated checks that prove the outcome |

**Problem, Outcome, and Scope are never optional.** A plan that survives every question about surfaces and invariants but cannot say what is broken or what success looks like has not been grilled — it has been inventoried.

## Citing Code

Never cite a line number — no `L123`, no `file.cs:123`, no "at line 123". Refer to code by symbol name, by a named position inside a symbol, or by a short quoted expression the reader can grep. Line numbers go stale between this interview and implementation, and anything stale here propagates into the PRD and every issue split from it.
