---
title: Git reroot
date: 2020-02-26T17:47:53Z
draft: false
lang: en-US
tags: ['git']
cover:
  image: banyan-tree.jpg
  caption: Source https://www.pickpik.com/banyan-tree-tree-maui-hawaii-banyon-beach-70064
---

So, you have a pet project in a private repository. It turns out that the project could work as an open source project. When you looked into its history, you find some stuff that must not go public. You want to delete those early commits and keep recent commits.

> Note that this article uses `master` as the default branch name. Recent repositories might use `main` for it.

> Note that the instructions provided in this document might destroy your working repository. Be warned.

---

Let's imagine our repository state like this:

```
master: O-----X--------H
```

- `O` is original first commit
- `X` is the commit you want to be as the new first commit
- `H` is the head, i.e., latest commit

> Note: I recommend to delete all but `master` branch before we start the operation.

We will need to get the commits

- `ORev` is the commit of `O` -- use the command `git rev-list --max-parents=0 HEAD`
- `XRev` is the commit of `X`. You will need to use `git log` or a Git GUI to find the commit.

```shell
$ git checkout -b reroot <XRev>
$ git reset --soft <ORev>
$ git commit --amend -m "My new initial commit"
```

The above commands do the following:

- First, it checks out `X` into a new branch named `reroot`
- Resets the original commit without reseting the files and index
- Creates a new commit with files from commit `X`

---

After running above commands, we are not at the following state.

```
master: O-----X--------H
reroot:       R--------I
```

We are now at branch `reroot` and at commit `R`.

Next step is to get `RRev`, which is the commit `R` -- use the command `git rev-list --max-parents=0 HEAD`. Yes, the same command we used to obtain `ORev`. We use the same command because we are now at commit `R`, so, the first commit will be `R`.

```shell
$ git checkout master
$ git replace <XRev> <RRev>
$ git filter-branch master
$ rm -rf .git/refs/original .git/refs/replace
$ git branch -d reroot
```

They do the following:

- First, it checks out the default branch (`master` in this case)
- Replaces the reference to `XRev` with `RRev`
- Rewrites the history
- Removes temporary references
- Deletes the now unused `reroot` branch

End result will look like this:

```
master:       R--------I
```

---

Please note that at the end of the process, the history is practically rewritten. The new head commit will not be the same as the old head commit and, no, you can't have the same head commit after re-root as it is logically impossible.
