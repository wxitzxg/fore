---
name: implement-spec
description: "Implement the result of /to-spec and /to-tickets in code."
disable-model-invocation: true
---

You have been provided a spec. This spec should have tickets associated with it, describing how to implement the spec.

The issue tracker should have been provided to you. If not, tell the user to run `/setup`.

The goal is the entire spec implemented on a single **integration branch**, every ticket's work merged into it. The run stops at the branch; the user runs `submit` afterward to push it and open a request.

The tickets are not a list of steps. They are a **task graph** with blocking relationships between them. The **frontier** is the set of tickets whose blockers have all merged into the integration branch; only these are ready to be grabbed. A blocked ticket stays ungrabbed until that merge lands, so its worktree always branches off an integration branch that already contains its blockers.

Communication to and from subagents should be sparse. Communicate primarily through **context pointers**: to the spec, tickets, research notes, and previous commits. Don't duplicate information already available via pointers.

**Implementer subagents** should be run in the background where possible for maximum concurrency.

## Steps

1. Read the spec and tickets to understand the task graph.

2. (optional) Use an **exploration subagent** to conduct any exploration required by the tickets - relevant codebase files or external documentation. Ensure the exploration subagent can save files - it should save its markdown notes in a directory outside the repo, accessible by all future subagents. This lets **implementer subagents** focus on implementation rather than exploration.

3. Create the integration branch off the default branch, never on it. If you are on `main` (or `master`), name it first: one `AskUserQuestion`, `Auto: <proposal>` (a kebab-case slug from the spec title) or `Type my own`, then `git switch -c <name>`.

4. Use **implementer subagents** to implement each ticket, each in its own worktree on its own branch. Each implementer subagent:
   - confirms its worktree is based on the integration branch before starting, and resets onto it if not;
   - calls the Skill tool with `tdd` to build the ticket;
   - merges the integration branch tip into its own branch before reporting done

5. Once an **implementer subagent** completes, merge its work to the integration branch with a **merger subagent**.

6. If this changes the **frontier** of available tickets, kick off more **implementer subagents** to work on the new tickets. This allows for maximum concurrency.

7. Once all tickets are complete, call the Skill tool with `code-review` on the integration branch. Fix all issues raised by the code review in a single **implementer subagent**.

8. Report the integration branch, then tell the user to run `submit` to push it and open the request, which closes the tickets.

9. Clean up all **implementer subagent** worktrees.
