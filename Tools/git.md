---
title: Git Notes
description: Notes on more advanced git stuff
---

## [Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

These are a way to nest git repos inside your main repo. Add one with `git submodule add {url}`, or clone a repo with its submodules using `git clone --recurse-submodules`.

`git submodule update --remote` will pull the latest changes from HEAD of each submodule's default branch (branch can be configured).

[Worktree](https://fev.al/posts/git-worktree/)

Gives you a simple command to check out a branch in a whole different directory, so for example you can fix a bug and deploy main while halfway through unsaved changes on a feature branch in another folder

Used like `git worktree add ../my_second_worktree the_other_branch`.
