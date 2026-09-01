---
name: do-next-issue
description: Implement exactly one issue — the next unresolved one from an issue index — then validate, commit, update the index, and stop. Use when the user says "do the next issue", "work the queue", "run do-next-issue", or points at an issues index and asks for the next slice. Consumes the queue produced by prd-to-issues.
---

# Do Next Issue

Implement one issue from the queue, prove it works, record it, and stop.

## Select The Issue

1. Read the index file.
2. Take the issue named under `## Next Issue`. If that section is missing or stale, fall back to the first `In Progress` or `Todo` row whose blockers are satisfied; among several ready issues, prefer the earliest `AFK`.
3. Never start a `Blocked` issue, and never start a `HITL` issue without the blocking decision in hand. Say what is blocking and stop.
4. If no issue is ready, say so, name what would unblock the queue, and stop.
5. Verify the working tree is clean. If it is dirty and the selected issue is already `In Progress`, a previous attempt failed partway — say so, summarize the uncommitted diff, and ask whether to resume or reset before touching anything. If it is dirty for any other reason, stop and explain what must be resolved first.
6. Mark the issue `In Progress` in the index before you begin.

Read only that issue file — later issue files come into play only at close-out. Inspect only the repository files needed to implement this issue.

## Implement

Implement only that issue, directly in the current session. Do not start an implementation subagent unless the user explicitly asks for one.

Prefer SQL projection over calling `LoadWith`.

Treat any line number in an issue file as untrustworthy — earlier issues in the queue have already moved the code. Locate code by the symbol or quoted expression named alongside it and verify you are looking at the right thing. **If the issue's description and the actual code disagree, the code wins** — say so rather than implementing against a stale description.

Never write a line number into an issue file, the index, or a commit message. Reference code by symbol name, by a named position inside a symbol, by a short quoted expression a reader can grep, or by file path alone when the file is small.

## Review

1. Review the diff. Confirm it contains only changes relevant to this issue.
2. Review in the current session by default. A diff spanning several files, a straightforward interface change, or a general compatibility note is not on its own a reason to spawn a reviewer — especially when focused tests exercise the behavior directly.
3. Spawn a fresh review subagent only for a concrete high-risk concern in the actual diff:
   - authentication, authorization, sessions, cross-origin protection, proxy trust, or another security boundary
   - a database schema or migration, possible data loss, transaction correctness, or concurrency
   - destructive operations or consequential external side effects
   - a large cross-cutting change with coupled behavior across independent areas
   - a subtle public-contract change that focused tests cannot protect
4. Give a review subagent only the issue filename, the current diff, and one or more narrow adversarial questions tied to the concrete risk — "Find a way this POST could bypass cross-origin protection." Instruct it to inspect only the supplied issue and diff: no worktree access, no tests or builds, no tools, no edits. Never ask for a generic correctness review.
5. Fix any real finding directly in the current session, then review the diff again. Repeated no-finding reviews are evidence to raise the review threshold, not a ritual to preserve.

## Close Out

1. Run the relevant tests and build.
2. If they fail, stop. Do not commit. Do not mark the issue `Done`. Leave it `In Progress`, note the failure in its index `Notes` cell so the next session knows what it is walking into, and explain the failure and what changed.
3. If they pass, commit with the message `Issue <filename>`, where `<filename>` is the issue file's base name without `.md` — for example `Issue 006-add-status-confirmation-and-data-group-lock`.
4. Update the index: mark this issue `Done`; remove it from every `Blocked by` cell that names it, promoting rows to `Todo` once their `Blocked by` is empty; and repoint `## Next Issue`.
5. If this issue moved, renamed, extracted, or deleted code that a later issue references, fix those references now while you still have the context. Point them at what exists after your change — the new type, method, and file path — naming the component rather than a location. Repair wrong references only; do not rewrite a later issue's scope or decisions.
6. Confirm the tracked working tree is clean.

Issue files and the index are local progress-tracking state, never committed. They should already be gitignored; if the commit in step 3 would sweep them in, stop, add the gitignore entry, and tell the user — do not stage or commit them.

## Stop

Stop after this one issue. Do not begin another. Do not mix issues in one implementation pass. Do not let a later issue influence this one, unless the codebase already changed because of a completed earlier issue.

If the issue depends on a later issue, conflicts with an earlier change, requires an unclear product decision, or cannot be completed safely — stop and explain instead of guessing.
