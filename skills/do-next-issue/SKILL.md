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

After implementation:

1. Review the diff.
2. Confirm the diff contains only changes relevant to the current issue.
3. If the diff is large, risky, touches shared architecture, changes public contracts, changes database behavior, changes authentication/proxy behavior, or affects multiple areas of the app, start a fresh review subagent.
4. Give the review subagent only:
   * the issue filename or issue entry
   * the current diff
   * instructions to verify that the diff is correct, focused, and safe
5. The review subagent must review only. It must not edit files.
6. If the review subagent finds a real issue, fix it directly in the current session, then review the diff again.
7. Run the relevant tests/build.
8. If tests/build fail, stop. Do not commit. Do not mark the issue as completed. Explain the failure and what changed.
9. If tests/build pass, commit the changes with the message "Issue <filename>" where <filename> is the issue file's base name without the `.md` extension (e.g. `006-add-status-confirmation-and-data-group-lock`).
10. Mark the issue as completed in the index file. If any issues are unblocked, update their status as well. This file is gitignored progress-tracking state and must not be staged or committed.
11. Confirm the tracked working tree is clean.
12. Stop after this one issue.

Do not begin another issue.

Do not mix multiple issues in the same implementation pass.

Do not let later issues influence this issue unless the codebase has already changed because of a completed earlier issue.

If the issue depends on a later issue, conflicts with an earlier change, requires unclear product decisions, or cannot be completed safely, stop and explain the problem instead of guessing.
