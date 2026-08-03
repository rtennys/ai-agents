---
name: to-prd
description: Write a concise, self-contained Product Requirements Document (PRD) from a client brief, grilled design, implementation discussion, or rough PRD draft. Use when the user wants a plan another AI session can implement with minimal extra context, especially before coding or issue-splitting.
---

# To PRD

Write a short PRD that a fresh, cheaper model can use to implement the change cleanly.

## Workflow

1. Use the current chat, brief, grilled notes, draft PRD, and repo inspection as source material.
2. Inspect the repo for implementation-critical facts instead of asking the user when possible.
3. Ask only about unresolved choices that change scope, behavior, ownership, data shape, compatibility, or validation.
4. Write the PRD to the user's requested path. If none is given, use `prd.md` beside the source prompt when the prompt is under `artifacts/`; otherwise use `artifacts/prd.md`.
5. Suggest whether this PRD could be implemented in one session, or if I should run the skill `prd-to-issues`.

Do not include code snippets, GitHub issue text, broad brainstorming, or exhaustive file-by-file instructions.

## What To Capture

Include enough detail for a new AI session to avoid guessing:

- Problem, user-visible goal, and intended system outcome
- Scope, non-goals, and behavior that must stay unchanged
- Current owner and target owner of the behavior
- Primary routes, screens, APIs, components, services, jobs, adapters, or data models involved
- Existing helpers, patterns, and prior tests that should be reused
- Compatibility paths such as legacy routes, redirects, shims, flags, permissions, blocked states, or downstream consumers
- Data shape, migration, cleanup, rollback, and risk notes when relevant
- Minimum manual and automated validation needed

## Maintainability Guidance

Make the PRD steer implementation toward small, boring code:

- Prefer local changes in the existing owner over new abstractions.
- Add an abstraction only when it removes real duplication or centralizes a rule that already has multiple callers.
- Reuse established repo patterns, helpers, components, and test style.
- Split work into end-to-end slices, not layers.
- State what should be deleted, left in place, or kept temporarily as a compatibility shim, redirect, or wrapper.
- Require touched implementation files to use CRLF line endings and end with a newline.

## Template

Use these sections, omitting parts that do not add clarity:

## Problem

## Solution

## Scope And Non-Goals

## Behavior Contract

## Implementation Plan

## Ownership And Reuse

## Compatibility And Cleanup

## Risks

## Validation
