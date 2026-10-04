---
name: tidy-diff
description: Aggressively simplify code that was just written or changed, without altering what it does. Use after implementing a ticket, issue, feature, or fix — when the user says "tidy this", "tidy the diff", "clean up my change", "make this smaller", "reduce the abstractions", or asks for a clean-code pass on recent work. Pairs with do-next-issue, whose commits this reviews.
---

# Tidy Diff

Make a just-finished change as small and as plain as it can be while behaving identically, then commit that cleanup as one revertable commit.

The standard is fewer *things* — fewer types, functions, parameters, branches, and indirections — not fewer characters. Measure a cut by what a reader no longer has to hold in their head, never by line count. A statement a reader can pause on is cheaper than an expression they must unpack. Behavior is frozen; only the shape of the code moves.

## Scope The Change

1. Verify the tracked working tree is clean. If it is dirty, stop and ask the user to commit or stash first. The tidy commit must stand alone so it can be reverted without touching the feature.
2. Determine the range under review, in this order:
   - a range or commit the user named;
   - otherwise, if the current branch is not the default branch, `<merge-base with the default branch>..HEAD`;
   - otherwise, `HEAD~1..HEAD`.
   State the range you chose. If it spans more than one commit, treat it as one change.
3. Run the project's tests and build now. If they are already failing, stop and report — a tidy pass starts from green or not at all.
4. Collect `git diff <range>` and the full current text of every file the range touches.

## Review

Dispatch one fresh review subagent. Give it the diff, the full text of the touched files, the cut list below, and any coding rules in the project's own instructions (`CLAUDE.md`, `AGENTS.md`, or equivalent) that bear on the touched code — the reviewer cannot read them itself. Instruct it to inspect only what it was given: no worktree access, no tests or builds, no tools, no edits. Its whole job is to return ranked cuts.

Each cut names its location by symbol or a short greppable expression, says what to do (delete, inline, merge, rename), gives the one-line reason behavior is unchanged, and names the thing removed — a type, a function, a parameter, a branch, a duplicate computation, a comment. Rank by how much a reader no longer has to track. A cut whose only gain is fewer lines is not a cut; do not propose it.

### The cut list

Look for these in lines the change introduced:

- An interface, base class, strategy, factory, registry, or generic parameter with exactly one implementation or one use. Replace it with the concrete thing.
- An options bag, parameter, flag, or config key that only ever receives one value. Delete it and hard-code the value.
- A method or function with one caller whose name says nothing its body does not already say. Inline it.
- A wrapper around a single expression or a single call. Inline it.
- A branch, `else`, default, or fallback no input can reach. Delete it.
- The same lookup or computation performed twice. Do it once.
- A guard that a later step already handles. Delete the guard.
- A comment that restates the code, marks where future code "can be added", or explains what a better name would explain. Delete the comment; rename if the name was the problem.
- `if (x) return true; else return false;` and every cousin of it. Return the expression.
- An import, export, variable, or file that a cut above leaves unused. Delete it — after grepping the repository for consumers of any export you remove.
- Style that differs from the surrounding file: the file's existing conventions win, even where the reviewer would choose otherwise.

## Preserve

The reviewer proposes no cut that does any of the following, and you apply none that does:

- Changes any output, return value, thrown error type, side effect, or the order of a numeric expression. `total * (1 - p / 100)` and `total * (100 - p) / 100` are different code.
- Touches a line the range did not touch, except to delete a line a cut has made dead.
- Adds a test, a validation, a guard, a comment, a type annotation, or documentation.
- Restyles working code — loop to `reduce`, `function` to arrow, quote style, reordering members. The one exception is a form the project's own instructions require: bring every line in the range into line with it, whether or not the reviewer raised it, and name occurrences outside the range in the report without touching them.
- Undoes a form the project's own instructions require, or reports one as drive-by noise.
- Renames or removes a public symbol that anything outside the range imports.
- Merges, splits, or moves files.
- Compresses statements into an expression: a block body into an expression body, a local variable into the expression that uses it, an `if` into a conditional or pattern-match chain, sequential steps into one nested call. A named local that holds an intermediate result is documentation; keep it.

A cut that saves lines by making a reader look in two places instead of one is not a cut. Neither is one that saves lines by making them look at one place harder. Reject both.

The line to hold:

```csharp
// Keep. Two named steps, one plain condition.
public static bool IsWithinWindow(DateTime? startedAt, DateTime? endedAt)
{
    var start = Normalize(startedAt);
    var end = Normalize(endedAt);
    if (!start.HasValue || !end.HasValue) return false;
    return end.Value - start.Value <= Window;
}

// Not a cut. Same work, fewer lines, more to unpack per line.
public static bool IsWithinWindow(DateTime? startedAt, DateTime? endedAt) =>
    Normalize(startedAt) is { } start
    && Normalize(endedAt) is { } end
    && end - start <= Window;
```

## Apply

1. Apply every accepted cut directly in the current session, in rank order. Do not start an implementation subagent.
2. Run the tests and build once.
3. If they fail, revert the lowest-ranked cut and run again; repeat until green. Record every cut you dropped and why.
4. Review the final diff of the tidy pass against the Preserve list one more time. Anything on that list, revert it.

## Commit

1. Commit everything as one commit using the [Scoped Commits](https://scopedcommits.com/) format: `<scope>: <description>`. Never amend, squash, or rewrite the original commits.
2. Choose the narrowest scope that accurately covers the tidy diff:
   - Follow an established scoped-commit convention in the repository when one exists.
   - In a .NET repository, normally use the project, component, or subsystem that owns the changed code.
   - In a Go repository, normally use the package or `cmd/<name>` area that owns the changed code.
   - When the diff crosses scopes, use their nearest meaningful shared area; if none exists, list the scopes separated by commas.
3. Make the description after `<scope>:` as short and succinct as possible. Use `tidy <subject>` when the subject adds useful context, but derive a shorter description from the diff when the first commit's subject is verbose or vague.
4. Never include a ticket number or issue number anywhere in the commit message, including the subject, body, or trailers. Strip one from any source text used to form the description. Do not add issue-closing syntax.
5. Omit the body and trailers unless they contain essential context unrelated to a ticket or issue.
6. Do not push.
7. Report: the range reviewed, net lines removed, each cut applied, each cut dropped in step 3 of Apply, and each reviewer cut you rejected against the Preserve list. Name code by symbol, never by line number.

## Stop

Stop after the one commit. Do not begin a second review pass on your own tidy commit. If a tempting cleanup lies outside the range — an old file, an adjacent function — name it in the report and leave it alone.
