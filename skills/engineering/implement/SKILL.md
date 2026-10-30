---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

## Branch gate

Resolve the current branch before building:

```bash
git rev-parse --abbrev-ref HEAD
```

If it is the default branch (`main` or `master`), branch off it first so the
work never lands on the default branch. Ask one `AskUserQuestion` to name it:
`Auto: <proposal>` (default) or `Type my own`. The proposal is a short
kebab-case slug from the ticket, the spec title, or the latest commit subject,
whichever is nearest. Then `git switch -c <name>`.

Implement the work described by the user in the spec or tickets.

Use /tdd where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Once done, use /code-review to review the work.

Commit your work to the current branch.
