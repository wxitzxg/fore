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
- Never guess an issue silently. A guess that is not confirmed in writing stops the flow.
- One request per branch. If an open request already exists, return its link.

## 1. Guards

Run these first, in this order, and fail fast with the exact message given:

1. **Default branch.** Resolve `origin`'s default branch name and the current branch:

   ```bash
   git rev-parse --abbrev-ref HEAD
   ```

   If the current branch is the default branch, stop:

   > Refusing to submit from the default branch `<name>`. Work committed here bypasses review. Create a feature branch first, then run /submit again.

2. **Dirty working tree.** Any output means dirty, including untracked files:

   ```bash
   git status --porcelain
   ```

   If non-empty, stop:

   > The working tree has uncommitted changes, which would be silently left out of the request. Commit, stash, or remove them, then run /submit again.

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

GitHub default branch:

```bash
gh repo view --json defaultBranchRef --jq .defaultBranchRef.name
```

GitLab default branch (encode the project path from the origin URL, e.g. `group%2Fproject`):

```bash
glab api "projects/<encoded-path>" --jq .default_branch
```

## 3. Resolve the issues to close

Find the linked issues in this fixed order of confidence. Stop at the first level that yields anything.

1. **Explicit arguments.** Digits, `#123`, or issue URLs passed to `/submit` win over everything. Resolve each and verify it exists.
2. **Implement session context.** This conversation is an implement session, so use what it was given: a single ticket, a ticket set, or a spec. For a spec, include the tickets produced from it, never other issues mentioning it.
3. **Branch-name digits.** If the branch name contains digits (for example `fix-login-12`), offer that issue as a suggestion only:

   > The branch name suggests #12, `<title>`. Close it when this request merges? Answer yes or give the issue number.

   Do not proceed without an explicit answer.
4. **Nothing found.** Stop and ask:

   > No linked issue was found in the arguments, this session, or the branch name. Give the issue number to close, then run /submit again.

Verify every candidate exists before using it: GitHub `gh issue view <n>`, GitLab `glab issue view <n>`. Drop or correct a reference that does not resolve, and say so.

## 4. Propose a title

Propose one title, always editable in the gate:

| Session shape | Proposed title |
| --- | --- |
| One ticket | That ticket's title |
| Multiple tickets | Their common parent's title, or the spec title |
| Neither | Latest commit subject: `git log -1 --pretty=%s` |
| No commits | An empty title field, to fill in by hand |

## 5. Check for an existing request

Before any network write, check whether the branch already has an open request:

GitHub:

```bash
gh pr list --head "<branch>" --state open --json number,url
```

GitLab:

```bash
glab mr list --source-branch "<branch>" -F json
```

If one exists, print its URL and stop. Nothing is pushed and nothing is duplicated.

## 6. Generate the body

Call the Skill tool with `pr` to generate the pull request body from the primary sources (the tickets, the spec, the commits). Then append a closing section at the end of that body. One reference per line so every issue closes:

```markdown
## Closes

Closes #3
Closes #4
```

GitHub and GitLab both treat `Closes #<n>` in a request description as a closing reference, recorded when the request merges. The issues live in the origin project, so bare `#n` references are correct; never invent cross-repo URLs.

Write the finished body to a temp file for the create command.

## 7. Confirmation gate

Show this gate before any network write, and wait for approval:

```text
Target: origin (<platform>, <owner/repo>)
Base: <base branch>
Title: <proposed title>
Draft: <yes/no>
Closes on merge:
  #3 <title>
  #4 <title>
```

Every field is editable on request, and the title is always offered for editing even when the proposal is strong. Ask whether the request should be a draft; a draft request cannot be merged accidentally.

## 8. Push

Push the branch to `origin` without force options:

```bash
git push -u origin HEAD
```

If the push is rejected as non-fast-forward, stop:

> The push was rejected because origin/<branch> has commits this branch does not. Rebase onto origin/<branch>, or merge it into your branch, resolve, commit, and run /submit again. Submit never force pushes.

## 9. Open the request

GitHub:

```bash
gh pr create --base "<base>" --title "<title>" --body-file "<body-file>"
# add --draft for a draft request
```

GitLab:

```bash
glab mr create --source-branch "<branch>" --target-branch "<base>" --title "<title>" --description-file "<body-file>"
# add --draft for a draft request
```

Print the request URL and stop. Do not watch checks, do not merge, do not edit the tickets. The user reviews CI and merges by hand; the tracker closes the listed issues with a closed-by record.
