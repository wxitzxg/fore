## What it does

`worktree` opens a git worktree, an isolated copy of the repository on its own branch, and names it from a short slug of what you're about to do.

The defining constraint is where the name comes from. It is not a fixed template (`work-1`, `scratch`) and not your prompt pasted verbatim: the agent reads what you said and collapses it to a short, lowercase, kebab-case slug, a verb-noun pair at most. `fix-login-timeout` from "the login sometimes times out after ten seconds", not `login-sometimes-times-out-after-ten-seconds`.

## When to reach for it

You invoke this by typing `/worktree <what you're doing>`; the agent won't reach for it on its own. Pass a few words about the task and the name is derived from them; pass nothing and the agent asks what the worktree is for before opening one.

Reach for it when you want a clean room for a task without disturbing the checkout you're in.

| Situation | Why a worktree |
| --- | --- |
| A side task you don't want touching your current branch | It gets its own branch and checkout |
| Two in-flight tasks that would collide in one tree | Each lives in its own directory |
| A scratch experiment you might throw away | Deleting the worktree discards it cleanly |

For moving work between *sessions or harnesses* rather than between checkouts, use [handoff](./handoff.md). For a throwaway program that answers a design question, use [prototype](../engineering/prototype.md).

## Isolation is the job

The worktree is not a lighter branch switch. It is a second, complete checkout of the repo in its own directory, so the working tree you left behind stays exactly as it was. That is what "isolated copy" buys: you can start dirty work in the new tree while the original tree, the one holding your uncommitted changes or a long-lived `main`, is untouched.

The name is the second half of it. A slug that says what the worktree is *for* is what keeps a pile of worktrees legible later: when the list is `fix-login-timeout`, `add-billing-endpoint` and `prune-stale-index`, each row reads as a task, not as a number you have to go look up.

## Common questions

**Worktree or just branch?** A branch change swaps the working tree in place; a worktree gives you a second tree alongside it. Reach for a worktree when you want both trees live at once, a branch when only one is needed. This skill is the worktree side; it is not a general branch helper.

**Why a skill rather than `git worktree add`?** The agent turning your prompt into the slug and opening the tree is the whole point. Typing the command yourself means naming it yourself, which is exactly the step the skill takes off you.

**Where does the worktree go?** Into the harness's worktree directory (under `.claude/worktrees/` in Claude Code), on a branch named after the slug. It is a scratch checkout that disappears when the task is done, not a durable copy of the repo.

## It's working if

- The worktree name is a short verb-noun slug that says what the task is, not a template or an index.
- You open the link and are in a fresh checkout, with the tree you came from untouched.
- The branch name matches the worktree name, so the pair reads as one task.
- You passed no prompt and it asked what the worktree was for rather than guessing.

## Where it fits

`worktree` is a **reach-for-it-anytime standalone**, a clean-room opener before work rather than a step inside a build chain. Its nearest neighbours are both about getting a separate place to work: [prototype](../engineering/prototype.md), which is a throwaway program in its own directory, and [handoff](./handoff.md), which moves context rather than a checkout. [guide](../engineering/guide.md) carries the full map and routes you from the opening of a task to the rest of the set.