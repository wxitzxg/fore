## What it does

`submit` pushes the current branch to `origin`, opens one pull request on GitHub or merge request on GitLab, prints the link, and stops. It writes the body itself (Summary, Evidence, Merge Danger) and appends `Closes #<n>` references so that merging the request by hand closes the linked issues on the tracker.

You drive every irreversible step. Submit never merges, never waits on checks, never force pushes, and never does post-merge cleanup. It closes exactly the issues you pick in the gate, and nothing else: arguments and session context supply candidates, and branch-name digits are listed but only close when you pick them.

## When to reach for it

You invoke this by typing `/submit`, and the agent will not reach for it on its own.

Run it in the same session that just ran implement, before clearing or compacting context: that session is how submit knows which tickets the branch implements.

| Your situation | Reach for |
| --- | --- |
| Implement finished and committed; push, open the request, link the issues | `submit` |
| The request already exists and you need its link | Run submit again; it returns the link without duplicating |
| Review comments came back on an open request | No fore skill covers this yet |
| Nothing is committed yet | [implement](./implement.md) |

## Prerequisites

- **A configured tracker.** Submit reads `docs/agents/issue-tracker.md` (written by [setup](./setup.md)) to pick GitHub or GitLab, falling back to the origin hostname. A local markdown tracker or an unidentifiable remote fails fast and points you at setup, because a code host cannot close markdown files under `.scratch/`.
- **Committed work on a non-default branch.** A dirty working tree is refused, since uncommitted work would be silently left out. A default branch is not refused outright: submit asks whether to open a worktree and resume from it, since committing straight to the default branch bypasses review.
- The platform CLI: `gh` for GitHub, `glab` for GitLab, authenticated against the origin host.

## How it finds the issues

The linked issues are resolved in a fixed order of confidence:

1. Explicit arguments to `/submit` (numbers or URLs).
2. This implement session's context: one ticket, a ticket set, or a spec.
3. Digits in the branch name, carried forward as a suggested candidate you settle at the gate.
4. Nothing at all: no candidate, and you type one through `Other` or the `/submit` args rather than the skill guessing.

Before any network write you get one confirmation gate. The title is not asked: it uses the proposed value, and a custom title arrives through `Other` or the args. When candidates exist, issues are a none-or-each-candidate choice; always joined by a submit-or-cancel question, which is the only question when there are no candidates. A custom value is typed through `Other` in the same turn, so there is never a second prompt. The proposed title comes from the ticket (one ticket), the common parent or spec (multiple tickets), or the latest commit subject as a fallback.

Origin is the only target, and the base branch must live inside it; a request naming upstream or another remote is refused even when asked for explicitly. A draft is requested through `/submit` args and still opens a request that cannot be merged until you promote it on the code host.

## Common questions

**Why must I run it in the same session, before clearing context?**

Because issue identity comes from session context. When implement was given a ticket or a ticket set, submit reads that directly; after `/clear`, that fact is gone and all that remains is pattern matching on the branch name, which submit treats as a suggestion rather than a fact. Running submit before clearing is what makes the closing references trustworthy.

**Will it merge once CI passes?**

No, and it will not wait on CI at all. It prints the request link and stops immediately. You watch the checks and merge by hand. That moment of human review is the point: auto-merge after green would turn the request into a formality.

**What happens if the push is rejected?**

Submit stops on a non-fast-forward and tells you to rebase onto or merge `origin/<branch>`, resolve, commit, and rerun. It never force pushes, and there is no hidden flag that makes it force push. A force push on a branch under review can quietly remove commits a reviewer already saw.

**Can the issue it closes be wrong?**

No, not silently. The request closes exactly the issues you pick in the gate: arguments and session context supply candidates, branch-name digits are listed but default to `None`, and a missing issue is typed through `Other`. Verify the issue list in the gate, since that exact list is what the tracker closes on merge.

## It's working if

- With a dirty tree it refuses before any network command runs; on the default branch it offers a worktree instead of writing.
- A single confirmation gate uses the proposed title without asking, asks the issues (none or each candidate) and submit-or-cancel, collects any custom value through `Other` with no second prompt, and nothing writes until you pick.
- Exactly one request exists for the branch afterward; rerunning submit returns its link instead of making another.
- The request body contains the Summary, Evidence and Merge Danger sections and one `Closes #<n>` line per linked issue.
- Merging the request by hand closes those issues with a closed-by record, and submit itself did not merge, wait on checks, or clean anything up.

## Where it fits

`submit` is the final user-driven step of the main flow: `grill-with-docs -> to-spec -> to-tickets -> implement -> submit -> manual merge`.

- [implement](./implement.md) is the closest neighbour: it builds and commits what submit pushes, and its session context supplies the linked issues.

[guide](./guide.md) routes across the whole set when you are unsure which skill the situation wants.
