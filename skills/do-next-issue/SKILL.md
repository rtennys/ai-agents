---
name: do-next-issue
description: Implement the next unresolved issue from the passed index file, then stop.
---

Read the index file.

Find the first unresolved issue listed in the index file.

Before starting, verify the working tree is clean. If it is not clean, stop and explain what must be resolved before continuing.

Read only the issue file or issue entry for the unresolved issue.

Inspect only the repository files needed to understand and implement that issue.

Implement only that issue directly in the current session.

Do not start an implementation subagent unless explicitly instructed by the user.

Prefer SQL projection over calling LoadWith.

Treat any line number in an issue file as untrustworthy. Earlier issues in the queue have already moved the code. Locate the code by the symbol or quoted expression named alongside it and verify you are looking at the right thing before relying on it. If an issue's description and the actual code disagree, the code wins — say so rather than implementing against the stale description.

Never write a line number into an issue file, the index, or a commit message. Reference code by symbol name, by a named position inside a symbol, or by a short quoted expression a reader can grep.

After implementation:

1. Review the diff.
2. Confirm the diff contains only changes relevant to the current issue.
3. Usually review the work directly in the current session. Do not start a review subagent merely because the diff spans several files, changes a straightforward interface, or carries a general compatibility note when focused tests directly exercise the behavior.
4. Start a fresh review subagent only when the implemented diff contains a concrete high-risk concern such as:
   * authentication, authorization, sessions, cross-origin protection, proxy trust, or another security boundary
   * a database schema or migration, possible data loss, transaction correctness, or concurrency
   * destructive operations or consequential external side effects
   * a large cross-cutting change with coupled behavior across independent areas
   * a subtle public-contract change that focused tests cannot adequately protect
5. When a review subagent is warranted, give it only:
   * the issue filename or issue entry
   * the current diff
   * one or more narrow, adversarial questions tied to the concrete risk, such as "Find a way this POST could bypass cross-origin protection."
6. Instruct the review subagent to inspect only the supplied issue and diff. It must not inspect the worktree, run tests or builds, call tools, or edit files. Do not ask for a generic correctness review.
7. If the review subagent finds a real issue, fix it directly in the current session, then review the diff again. Treat repeated no-finding reviews as evidence to tighten the review threshold, not as a ritual to preserve.
8. Run the relevant tests/build.
9. If tests/build fail, stop. Do not commit. Do not mark the issue as completed. Explain the failure and what changed.
10. If tests/build pass, commit the changes with the message "Issue <filename>" where <filename> is the issue file's base name without the `.md` extension (e.g. `006-add-status-confirmation-and-data-group-lock`).
11. Mark the issue as completed in the index file. If any issues are unblocked, update their status as well. This file is gitignored progress-tracking state and must not be staged or committed.
12. If this issue moved, renamed, extracted, or deleted code that a later issue's file references, fix those references now, while you still have the context. Point them at what actually exists after your change — the new type, method, and file path — and name the component rather than a location. Repair references that are now wrong; do not rewrite the later issue's scope or decisions. Issue files are gitignored progress-tracking state and must not be staged or committed.
13. Confirm the tracked working tree is clean.
14. Stop after this one issue.

Do not begin another issue.

Do not mix multiple issues in the same implementation pass.

Do not let later issues influence this issue unless the codebase has already changed because of a completed earlier issue.

If the issue depends on a later issue, conflicts with an earlier change, requires unclear product decisions, or cannot be completed safely, stop and explain the problem instead of guessing.
