---
name: do-next-issue
description: Implement exactly one issue — the next one in index order — prove it on the local machine, commit, update the index, and say whether the next issue can start or a human judgment is needed first. Use when the user says "do the next issue", "work the queue", "run do-next-issue", or points at an issues index and asks for the next slice. Consumes the queue produced by prd-to-issues.
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

## Prove It Locally

Every acceptance criterion is proven on this machine before it is called manual or deferred. Exhaust the local options first: run the built app behind its real host instead of the dev server, seed the data or the session a flow needs, host the publish output under IIS Express when a web.config matters, stub the one external call that cannot be made, read a vendor's public artifact instead of asking for a dashboard. The user would rather the session spend an hour reproducing a condition locally than spend a deploy on it.

Never open a pull request, merge, or deploy in order to test. The queue's commits accumulate on one branch, and the ticket ships as one pull request once the last issue lands; opening it is the user's call, with the house PR skill, not a step of this skill. A check genuinely needs the deployed environment only when nothing on this machine can observe it: an external scanner, a vendor service bound to the deployed hostname, data that exists only there. Those checks never gate the queue. Record each one in the index under a `## After The Deploy` heading, below the table, as one line naming the check and the result it should show; they run once, together, after the single merge.

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
4. Update the index: mark this issue `Done`; remove it from every `Blocked by` cell that names it, promoting rows to `Todo` once their `Blocked by` is empty; point `## Next Issue` at the first row that is still not `Done`; and add any deploy-only check from this issue to `## After The Deploy`.
5. If this issue moved, renamed, extracted, or deleted code that a later issue references, fix those references now while you still have the context. Point them at what exists after your change — the new type, method, and file path — naming the component rather than a location. Repair wrong references only; do not rewrite a later issue's scope or decisions.
6. Confirm the tracked working tree is clean.

Issue files and the index are local progress-tracking state, never committed, and nothing reads them once the ticket ships. The index `Notes` cell is for the next session working this queue; write nothing for readers beyond that. They should already be gitignored; if the commit in step 3 would sweep them in, stop, add the gitignore entry, and tell the user — do not stage or commit them.

## Hand-Off

End your final message with exactly one of these verdicts, on its own line, so the user never has to infer it:

- **Pause for a judgment.** Use this only when the next issue cannot safely start until a person has judged something this machine cannot: the issue is `Review` type, or the change alters something only a person can judge — how a screen reads, what a report says, how an email lands — and a later issue builds on it. List the exact steps the user should perform on this machine and the result each should show, so the check takes minutes. A check that needs a deploy or an external system is never a reason to pause; it belongs under `## After The Deploy`.
- **Continue.** The default: the automated and local proof covers the acceptance criteria. Say that the user can run `do-next-issue` in a fresh session for `<next-issue-file>`, and name in one line anything this issue added to `## After The Deploy`.
- **Queue complete.** Every row is `Done`. Repeat the `## After The Deploy` list, since that is what the user runs after merging the one pull request.

Pick the verdict from what you proved, not from how confident you feel. When in doubt, continue: a pause costs the user a round trip and tempts a deploy whose only purpose is to test.

## Stop

Stop after this one issue. Do not begin another. Do not mix issues in one implementation pass. Do not let a later issue influence this one, unless the codebase already changed because of a completed earlier issue.

If the issue depends on a later issue, conflicts with an earlier change, requires a product decision the issue file does not settle, or cannot be completed safely — stop, explain, and ask. Do not guess, and do not pick up a different issue instead.
