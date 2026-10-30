---
name: submit
description: "Push the current branch and open a pull request or merge request whose body closes the linked issues. Run it yourself after implement, in the same session and before clearing context."
disable-model-invocation: true
---

# Submit

The user has finished an implement session. They run `/submit` themselves, in the same session and before clearing context, because the session knows which tickets were implemented. Submit pushes the current branch to `origin`, opens one pull request (GitHub) or merge request (GitLab), prints the link, and stops.

The request body comes from the `pr` skill; submit appends closing references so merging the request by hand closes the linked issues on the tracker.

## Hard boundaries

These never move, no matter what arguments the user passes:

- Never merge, never wait on checks, never do post-merge cleanup. The user merges by hand and keeps control of CI.
- Never force push, and no flag enables it.
- Target `origin` only. A request naming `upstream` or any other remote is refused, and the base branch must be a branch inside `origin`.
- The request closes exactly the issues the user picks in the gate, and nothing else.
- One request per branch. If a request already exists, open, merged, or closed, return its link and stop.

## 1. Guards

Run these first, in this order, and fail fast with the exact message given:

1. **Dirty working tree.** Any output means dirty, including untracked files:

   ```bash
   git status --porcelain
   ```

   If non-empty, stop:

   > The working tree has uncommitted changes, which would be silently left out of the request. Commit, stash, or remove them, then run /submit again.

2. **Default branch.** Resolve `origin`'s default branch name and the current branch:

   ```bash
   git rev-parse --abbrev-ref HEAD
   ```

   If the current branch is the default branch, ask once with `AskUserQuestion` instead of refusing:

   - `New worktree` (default): hand the task to the `worktree` skill with a name derived from what this session implemented (a ticket or spec title) or the latest non-merge commit subject, falling back to letting the worktree skill ask. The worktree opens and switches, then submit resumes from that branch, which no longer trips this guard.
   - `Cancel`: stop here.

   > Submitting from the default branch `<name>` bypasses review.

## 2. Resolve platform and target

Determine the platform in this order:

1. Read `docs/agents/issue-tracker.md`. Its heading names the platform: GitHub, GitLab, or Local Markdown.
2. If the file is missing, fall back to the `origin` hostname (`git remote get-url origin`): `github.com` means GitHub; `gitlab.com` or a self-hosted GitLab host means GitLab.
3. Everything else stops:

   > This repo has no identifiable GitHub or GitLab tracker, so merging a request cannot close its issues. Run /setup to configure the issue tracker, then run /submit again.

A **Local Markdown** tracker (or any freeform tracker) stops with the same message: markdown files under `.scratch/` cannot receive closed-by records from a code host.

Refuse any remote that is not `origin`, in words:

> submit only targets origin. A request against `<remote>` is refused even when named explicitly.

Resolve the base branch (normally `origin`'s default branch) and confirm it exists on the remote before anything else:

```bash
git ls-remote --heads origin "<base>"
```

Derive `<owner>/<repo>` from the origin URL once and reuse it. GitHub default branch (REST works with fine-grained tokens that cannot use GraphQL):

```bash
gh api "repos/<owner>/<repo>" --jq .default_branch
```

GitLab default branch (encode the project path from the origin URL, e.g. `group%2Fproject`):

```bash
glab api "projects/<encoded-path>" --jq .default_branch
```

## 3. Resolve the issues to close

Collect the linked issue candidates in this fixed order of confidence, stopping at the first level that yields anything. Nothing is asked here: the gate settles every issue, so a candidate is carried forward, not confirmed.

1. **Explicit arguments.** Digits, `#123`, or issue URLs passed to `/submit` win over everything. Resolve each and verify it exists.
2. **Implement session context.** This conversation is an implement session, so use what it was given: a single ticket, a ticket set, or a spec. For a spec, include the tickets produced from it, never other issues mentioning it.
3. **Branch-name digits.** If the branch name contains digits (for example `fix-login-12`), verify the issue exists and carry it forward as a candidate, marked suggested rather than fact.
4. **Nothing found.** Carry forward no candidate; the gate's `Other` choice lets the user type one in the same turn.

Verify every candidate exists before using it:

```bash
# GitHub (REST works with fine-grained tokens that cannot use GraphQL)
gh api "repos/<owner>/<repo>/issues/<n>"
# GitLab
glab issue view <n>
```

Drop or correct a reference that does not resolve, and say so.

## 4. Propose a title

Propose one title as the automatic default in the gate:

| Session shape | Proposed title |
| --- | --- |
| One ticket | That ticket's title |
| Multiple tickets | Their common parent's title, or the spec title |
| Neither | Latest commit subject: `git log -1 --pretty=%s` |
| No commits | An empty title field, to fill in by hand |

## 5. Check for an existing request

Before any network write, check whether the branch already has a request, open or closed:

GitHub:

```bash
gh api "repos/<owner>/<repo>/pulls?head=<owner>:<branch>&state=all&per_page=1&sort=updated&direction=desc" --jq 'if length > 0 then .[0] | {url: .html_url, state, merged} else empty end'
```

An empty output means none exists. Do not treat a `null` line from `.[0]` on an empty array as a found request.

GitLab:

```bash
glab mr list --source-branch "<branch>" --state all -F json
```

If a request exists, print its URL, say whether it is open, merged, or closed, and stop. A merged branch has nothing left to submit, and an open or closed request already exists, so nothing is pushed or duplicated in any case.

## 6. Generate the body

Call the Skill tool with "pr" to generate the pull request body from the primary sources (the tickets, the spec, the commits). Then append a closing section at the end of that body. One reference per line so every issue closes:

```markdown
## Closes

Closes #3
Closes #4
```

GitHub and GitLab both treat `Closes #<n>` in a request description as a closing reference, recorded when the request merges. The issues live in the origin project, so bare `#n` references are correct; never invent cross-repo URLs.

Write the finished body to a temp file for the create command.

## 7. Confirm once

Ask once, with a single `AskUserQuestion`. There is no second prompt and no follow-up gate.

The title is never asked; it uses the step 4 proposal, and a custom title arrives through `Other` or `/submit` args.

When step 3 produced candidates, ask the issues:

- `None` (default where every candidate is only suggested)
- `Close #<n> <title>` (one option per candidate; a candidate from arguments or the session is default)

Always ask last:

- `Submit` (default)
- `Cancel`

With no candidates, skip the issue question and ask only `Submit` versus `Cancel`. A custom issue number is typed into `Other` in the same turn, never a second question. Submit defaults to a non-draft request; a draft is requested through `/submit` args, not through the gate.

## 8. Push

Push the branch to `origin` without force options:

```bash
git push -u origin HEAD
```

If the push is rejected as non-fast-forward, stop:

> The push was rejected because origin/<branch> has commits this branch does not. Rebase onto origin/<branch>, or merge it into your branch, resolve, commit, and run /submit again. Submit never force pushes.

## 9. Open the request

GitHub (REST, since `gh pr create` uses GraphQL and fails under a fine-grained token):

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

Print the request URL and stop. The tracker closes the issues the user picked, with a closed-by record.
