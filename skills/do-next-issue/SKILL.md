---
name: do-next-issue
description: Implement exactly one issue — the next one in index order — then validate, commit, update the index, and say whether to pause for a manual check or continue in a fresh session. Use when the user says "do the next issue", "work the queue", "run do-next-issue", or points at an issues index and asks for the next slice. Consumes the queue produced by prd-to-issues.
---

# Do Next Issue

Implement the next issue in the queue, prove it works, record it, and stop.

The queue is worked strictly in index order, one issue per session. Never skip an issue because it is blocked, needs a decision, or looks harder than the one after it. Whatever the next issue needs from the user, ask for it and wait.

## Select The Issue

1. Read the index file.
2. Take the first row that is not `Done`. `## Next Issue` should name the same file; if it does not, the row order wins — fix the pointer and carry on.
3. If that row is `Blocked`, the queue order and the `Blocked by` column disagree, or a blocker has not actually landed. Say which issues block it and what each one still needs, then ask the user how to proceed — resolve the blocker, declare it satisfied, or reorder the queue. Do not move on to a later issue.
4. If that issue is `HITL`, read its `Open Questions Or Blockers` section and put each open question, decision, credential, or access need to the user plainly, with the options you see. Wait for the answer. Record what the user supplied in that section of the issue file, replacing the question, so a resumed session does not ask again. Then proceed with this issue in this session. Never implement a `HITL` issue on a guess, and never leave it for later.
5. If every row is `Done`, say the queue is complete and stop.
6. Verify the working tree is clean. If it is dirty and the selected issue is already `In Progress`, a previous attempt failed partway — say so, summarize the uncommitted diff, and ask whether to resume or reset before touching anything. If it is dirty for any other reason, stop and explain what must be resolved first.
7. Mark the issue `In Progress` in the index before you begin.

Read only that issue file — later issue files come into play only at close-out. Inspect only the repository files needed to implement this issue.

## Implement

Implement only that issue, directly in the current session. Do not start an implementation subagent unless the user explicitly asks for one.

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
4. Update the index: mark this issue `Done`; remove it from every `Blocked by` cell that names it, promoting rows to `Todo` once their `Blocked by` is empty; and point `## Next Issue` at the first row that is still not `Done`.
5. If this issue moved, renamed, extracted, or deleted code that a later issue references, fix those references now while you still have the context. Point them at what exists after your change — the new type, method, and file path — naming the component rather than a location. Repair wrong references only; do not rewrite a later issue's scope or decisions.
6. Confirm the tracked working tree is clean.

Issue files and the index are local progress-tracking state, never committed, and nothing reads them once the ticket ships. The index `Notes` cell is for the next session working this queue; write nothing for readers beyond that. They should already be gitignored; if the commit in step 3 would sweep them in, stop, add the gitignore entry, and tell the user — do not stage or commit them.

## Hand-Off

End your final message with exactly one of these verdicts, on its own line, so the user never has to infer it:

- **Pause for a manual check.** Use this when any of the following holds: an acceptance criterion is marked `Manual:` or otherwise names a check you could not run yourself; the issue is `Review` type; the change alters something only a person can judge — a screen, a report, an email, an external system — and no automated test covers it; or a later issue builds on behavior that you could not prove. Then list the exact steps the user should perform and the result each should show, so the check takes minutes. Say that the next issue should wait until the check passes.
- **Continue.** Use this when the automated validation covers the acceptance criteria and nothing above applies. Say that the user can run `do-next-issue` in a fresh session for `<next-issue-file>`.
- **Queue complete.** Every row is `Done`. Name any manual checks still outstanding from earlier issues.

Pick the verdict from what you could and could not verify, not from how confident you feel. When in doubt, pause.

## Stop

Stop after this one issue. Do not begin another. Do not mix issues in one implementation pass. Do not let a later issue influence this one, unless the codebase already changed because of a completed earlier issue.

If the issue depends on a later issue, conflicts with an earlier change, requires a product decision the issue file does not settle, or cannot be completed safely — stop, explain, and ask. Do not guess, and do not pick up a different issue instead.
