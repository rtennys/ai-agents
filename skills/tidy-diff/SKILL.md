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

Dispatch one fresh review subagent. Give it the diff, the full text of the touched files, and both the cut list and the bit-predicate rule below. Instruct it to inspect only what it was given: no worktree access, no tests or builds, no tools, no edits. Its whole job is to return ranked cuts.

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

### Bit predicates

Separate from the cut list, and regardless of what else the change does: a boolean condition the change introduces inside an `IQueryable` lambda never uses `!`. Write `x.IsVoid == false`.

Negation translates to a `<>` predicate, which the query planner will not match against an index on that column; `== false` translates to `= 0`, which it will. Both select the same rows, so this is never a behavior change — it is the difference between a seek and a scan. A bare truthy reference already translates to `= 1` and needs nothing, so only the negation has to go.

- `Where(x => !x.IsVoid)` becomes `Where(x => x.IsVoid == false)`.
- `Where(x => x.IsActive)` stays exactly as written. Do not add `== true`.
- `Any(x => x.IsActive && !x.IsVoid)` becomes `Any(x => x.IsActive && x.IsVoid == false)` — every negated operand, not just an outermost one.
- A `bool?` is already sargable as `== true` or `== false`. Leave `!= true` alone: it is *not* interchangeable with `== false`, because the two disagree on null rows, and swapping them changes which rows come back.

The rule follows `IQueryable`, not the provider. EF, linq2db, and any repository abstraction over either all translate the expression tree to SQL, so all of them are in scope, as is any `Expression<Func<...>>` the change builds. `IEnumerable` is not: a loaded navigation property, a `List<T>`, anything already `ToList()`ed runs in memory and keeps whatever form reads best. Where you cannot tell which one you are looking at, write `== false` — it costs a reader nothing, and a missed one costs a table scan.

Plain in-memory C# is left alone. `if (!found)` and `while (!done)` stay exactly as written.

This rule is mandatory and not a matter of taste. Apply it to every qualifying condition in the range, whether or not the reviewer raised it. Name occurrences you spot outside the range in the report and leave them alone.

## Preserve

The reviewer proposes no cut that does any of the following, and you apply none that does:

- Changes any output, return value, thrown error type, side effect, or the order of a numeric expression. `total * (1 - p / 100)` and `total * (100 - p) / 100` are different code.
- Touches a line the range did not touch, except to delete a line a cut has made dead.
- Adds a test, a validation, a guard, a comment, a type annotation, or documentation.
- Restyles working code — loop to `reduce`, `function` to arrow, quote style, reordering members. The bit-predicate rule is the sole exception, and it is mandatory rather than optional.
- Rewrites an `== false` comparison back to `!x`, or reports one as drive-by noise. That form is required; a change that introduces it is correct and stays.
- Renames or removes a public symbol that anything outside the range imports.
- Merges, splits, or moves files.
- Compresses statements into an expression: a block body into an expression body, a local variable into the expression that uses it, an `if` into a conditional or pattern-match chain, sequential steps into one nested call. A named local that holds an intermediate result is documentation; keep it.

A cut that saves lines by making a reader look in two places instead of one is not a cut. Neither is one that saves lines by making them look at one place harder. Reject both.

The line to hold:

```csharp
// Keep. Two named steps, one plain condition.
public static bool IsInitialInsert(DateTime? addedAt, DateTime? tradeCreatedOn)
{
    var added = Normalize(addedAt);
    var created = Normalize(tradeCreatedOn);
    if (!added.HasValue || !created.HasValue) return false;
    return added.Value - created.Value <= Tolerance;
}

// Not a cut. Same work, fewer lines, more to unpack per line.
public static bool IsInitialInsert(DateTime? addedAt, DateTime? tradeCreatedOn) =>
    Normalize(addedAt) is { } added
    && Normalize(tradeCreatedOn) is { } created
    && added - created <= Tolerance;
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
