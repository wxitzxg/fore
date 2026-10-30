---
name: worktree
description: Open a git worktree, deriving the name from what you want to do.
argument-hint: "What will you work on?"
disable-model-invocation: true
---

Open a git worktree for the task the user describes.

Derive the worktree's name from the prompt: a short, lowercase, kebab-case slug that sums up the task in at most three words, ideally a verb-noun pair (fix-login-timeout, add-billing-endpoint).

If the user gave no prompt, ask what the worktree is for before opening one.

Open it with the EnterWorktree tool, passing the slug as `name`. Done when the session is inside the new checkout at `.claude/worktrees/<slug>`, with the tree you came from untouched.