---
name: submit
description: "Push the current branch to origin and open one pull request or merge request whose body closes the issue(s) you pick. Run it yourself right after implement, in the same session and before clearing context."
disable-model-invocation: true
---

# Submit

Ship what implement just built. Submit puts the work on a reviewable branch in
`origin`, opens one request, and prints its link. You merge it by hand afterward;
submit never merges.

Every decision that is yours lands in one gate, answered in a single turn: commit
the uncommitted work, get it onto the right branch, name the request, pick the
issues it closes, and confirm. Nothing writes before that gate, and nothing is
asked twice.

## The gate

One `AskUserQuestion`, composed from the state you detected on the way in. Each
point appears only when its condition holds:

| Point | Asked when | Options |
| --- | --- | --- |
| Commit | the working tree is dirty | `Commit the changes` (default), `Cancel` |
| Branch | the current branch is `origin`'s default | `New branch` (default), `Worktree` |
| Title | always | `Auto: <proposal>` (default), `Type my own` |
| Closes | an issue candidate exists | `Close #<n> <title>` (one per candidate, first is default), `None` |
| Create | always | `Create` (default), `Cancel` |

`AskUserQuestion` holds four questions. When a dirty tree and the default branch
coincide, the Commit and Branch points merge into one landing question and a
worktree is not offered there, because a worktree does not carry uncommitted
changes, which would stay behind on the default branch.

A custom title or an issue number not listed arrives through `Other` in the same
turn. There is no second prompt.

## Resolve before the gate

Settle everything the gate does not ask, with no interaction:

1. **State.** Dirty tree and current branch:

   ```bash
   git status --porcelain          # non-empty means dirty, untracked files included
   git rev-parse --abbrev-ref HEAD
   ```

2. **Platform and target.** Read `docs/agents/issue-tracker.md`; its heading names
   GitHub, GitLab, or Local Markdown. Missing file, fall back to the `origin`
   hostname: `github.com` is GitHub, `gitlab.com` or a self-hosted GitLab host is
   GitLab. Anything else, or a Local Markdown tracker, stops:

   > This repo has no identifiable GitHub or GitLab tracker, so merging a request cannot close its issues. Run /setup to configure the issue tracker, then run /submit again.

   Refuse any remote but `origin`:

   > submit only targets origin. A request against `<remote>` is refused even when named explicitly.

   Derive `<owner>/<repo>` from the origin URL once. Resolve the base branch
   (normally the default branch) and confirm it exists on the remote before
   anything else:

   ```bash
   git ls-remote --heads origin "<base>"
   ```

   GitHub default branch (REST works with a fine-grained token, GraphQL does not):

   ```bash
   gh api "repos/<owner>/<repo>" --jq .default_branch
   ```

   GitLab default branch (path-encode the project, e.g. `group%2Fproject`):

   ```bash
   glab api "projects/<encoded-path>" --jq .default_branch
   ```

3. **Issue candidates.** Collect them in this order of confidence, stopping at the
   first level that yields anything. Carried forward here, never confirmed: the
   gate settles them.

   1. Explicit arguments to `/submit` (digits, `#123`, issue URLs) win over everything.
   2. This implement session's context: a ticket, a ticket set, or a spec. For a
      spec, include only the tickets produced from it.
   3. Digits in the branch name, carried as a suggested candidate rather than fact.
   4. Nothing: no candidate; the gate's `Other` lets the user type one.

   Verify every candidate exists before the gate:

   ```bash
   # GitHub
   gh api "repos/<owner>/<repo>/issues/<n>"
   # GitLab
   glab issue view <n>
   ```

   Drop or correct a reference that does not resolve, and say so.

4. **Title proposal.** One proposal that seeds the gate's `Auto` option:

   | Session shape | Proposal |
   | --- | --- |
   | One ticket | That ticket's title |
   | Multiple tickets | Their common parent's title, or the spec title |
   | Neither | Latest commit subject: `git log -1 --pretty=%s` |
   | No commits | Empty, filled by hand |

5. **Existing request.** Before any write, check whether this branch already has a
   request, open, merged, or closed:

   ```bash
   # GitHub
   gh api "repos/<owner>/<repo>/pulls?head=<owner>:<branch>&state=all&per_page=1&sort=updated&direction=desc" --jq 'if length > 0 then .[0] | {url: .html_url, state, merged} else empty end'
   # GitLab
   glab mr list --source-branch "<branch>" --state all -F json
   ```

   Empty output means none exists. If one does, print its URL, say whether it is
   open, merged, or closed, and stop: no question is needed and nothing is pushed
   or duplicated.

## Act after the gate

Carry out the answers, in order:

1. **Land the work.** From the Commit and Branch answers:

   - Commit the changes: `git add -A` then `git commit` with a message derived
     from the session (the ticket title, or the work's theme).
   - On the default branch: `git switch -c <name>` first, so the commit lands off
     the default branch; or open a worktree through the `worktree` skill when the
     tree was clean and the user picked `Worktree`, then resume from that branch.

   A `New branch` on a default branch with no commits yet still works: the first
   commit lands on the new branch and the default branch stays unborn.

2. **Body.** Write the request body from the primary sources (the tickets, the
   spec, the commits), keeping prose brief and no preamble, then close with one
   reference per line so each picked issue closes on merge:

   ```markdown
   ## Summary

   <the smallest view that makes the change clear: a diagram, tree, diff-sketch, or pseudocode>

   ## Evidence

   - **Before:** <failing test run / output / screenshot>
     **After:** <passing test run / output / screenshot>

   ## Merge Danger

   **Door:** <one-way or two-way>
   **Blast Radius:** <one word, plus optional ramifications>

   ## Closes

   Closes #3
   Closes #4
   ```

   The summary is a picture, not a paragraph: pick the smallest view that carries
   the point, place each visual next to the short text it supports, and keep only
   the calls, files, props and boundaries the reviewer needs. A one-way door is a
   destructive or hard-to-reverse change; a two-way door is cheap to roll back.
   GitHub and GitLab both read `Closes #<n>` in a request description as a closing
   reference, and the issues live in the origin project, so bare `#n` is correct;
   never invent cross-repo URLs. Write the finished body to a temp file.

3. **Push.** Push the branch to `origin`, no force options:

   ```bash
   git push -u origin HEAD
   ```

   A non-fast-forward rejection stops here:

   > The push was rejected because origin/<branch> has commits this branch does not. Rebase onto origin/<branch>, or merge it into your branch, resolve, commit, and run /submit again. Submit never force pushes.

4. **Open the request.** GitHub (REST, since `gh pr create` uses GraphQL and a
   fine-grained token cannot):

   ```bash
   gh api --method POST "repos/<owner>/<repo>/pulls" \
     -f title="<title>" -f head="<branch>" -f base="<base>" \
     -f body="$(cat <body-file>)" \
     --jq .html_url
   # add -f draft=true for a draft request
   ```

   GitLab:

   ```bash
   glab mr create --source-branch "<branch>" --target-branch "<base>" --title "<title>" --description-file "<body-file>"
   # add --draft for a draft request
   ```

   Print the URL and stop. The tracker closes the issues the user picked, with a
   closed-by record.

## Boundaries

Hard rules that hold for any arguments:

- You push and open; the user merges by hand and keeps control of CI.
- Never force push, and no flag enables it.
- Target `origin` only; the base branch must live inside `origin`.
- The request closes exactly the issues picked in the gate, and nothing else.
- One request per branch. If one already exists, open, merged, or closed, return
  its link and stop.