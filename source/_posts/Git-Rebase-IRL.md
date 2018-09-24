---
title: Git Rebase IRL
date: 2018-09-25 01:52:33
tags:
---

## Intro

Many people might already known that there is a command call `git rebase`, but most of people don't know how to use it right.  
This article will tell you some common cases that you can use `git rebase` to make your commit flow more clearly.

> P.S. the man page of `git rebase` also provided a very clear document. if you want to learn more about `git rebase`, just read it :)

## When will I need it?

There is some common cases:

 1. To split one commit to two or more.
 2. To rename/delete one or more commits.
 3. To merge some commit into one.
 4. To append bunch of commits into another branch.

## Split one commit to two or more

Assume we have following commits:

```
ad99c7e initial commit
fb89181 implement a feature and change the style
7c189c8 remove bugs
```

You might want to split your second commit into to commit, so every commit only do one things, that keeps git history atomic.  

First, we need to start a rebase, use `git rebase -i ad99c7e`. `-i` means interactively rebase.
The git rebase will use `ad99c7e` as a new base, think like rebase is to create a new branch, then we do some works on that branch. Once we have done our work, it destroy the original branch and use the new one to replace it. As a result, the hashes starting from `ad99c7e` will be changed.  

```
# git rebase -i ad99c7e
```

Then, git will open the editor, showing up following content:
```
pick fb89181 implement a feature and change the style
pick 7c189c8 remove bugs
```

this file is called `git-rebase-todo`. we can now edit it to tell it how we want to do during rebasing.  

to split a commit, we need to edit it, so let's change the `pick` to `edit` on first line, then save it and exit your text editor:
```
edit fb89181 implement a feature and change the style
pick 7c189c8 remove bugs
```

now, git will stop at `fb89191` and waiting for you. try use `git status` to see what happened.
```
interactive rebase in progress; onto fb89191
You are currently editing a commit while rebasing branch 'master' on 'fb89191'.
  (use "git commit --amend" to amend the current commit)
  (use "git rebase --continue" once you are satisfied with your changes)
```

to edit this commit, first we need to cancel it by using `git reset --soft HEAD`.
you can now use the combination of `git add`, `git rebase`, or `git commit` to create more commit.

once you have done, use `git rebase --continue` to continue.

afterall, you might notice that starting from `ad99c7e`, the hashes are different now, just like I explained above:
```
ad99c7e initial commit
fb89181 implement a feature 
4434f2c change the style
45fdb3a remove bugs
```

# Merge some commit into one

Assume we have following commits:
```
ad99c7e initial commit
fb89181 implement a feature 
4434f2c change the style
45fdb3a remove bugs
```

What if I want to merge `45fdb3a` to `fb89181` to hide my mistake?

Let's starting rebase by `git rebase -i ad99c7e` like we have done erlier:
```
pick ad99c7e initial commit
pick fb89181 implement a feature 
pick 4434f2c change the style
pick 45fdb3a remove bugs
```

This time, we need to use the command `squash`. The `squash` means to merge that commit into previous one:
```
pick ad99c7e initial commit
pick fb89181 implement a feature 
squash 45fdb3a remove bugs
pick 4434f2c change the style
```

By doing this, `45fdb3a` will be merged into `fb89191`. You might noticed that I changed the order of the commits. That's okay to do it. You can always changing it's order when rebasing, just be careful when conflict.
Save it and exit the editor. Then git will prompt the text editor again, to asking you the commit message the merged commit should have:
```
# This is a combination of 2 commits.
# This is the 1st commit message:

implement a feature 

# This is the commit message #2:

remove bugs
```
let's hiding the second message. save and exit the text editor. and that's all! we have successfully merged that two commits.

# To rename/delete one or more commits.
Let's take following rebase todo for example:
```
pick ad99c7e initial commit
pick fb89181 implement a feature 
pick 4434f2c change the style
pick 45fdb3a remove bugs
```

To rename a commit, just use `reword`.
To delete a commit, just delete that line ;)

# Append a bunch of commits into another branch
That it a very common case if you are using some git workflow. When using gitflow workflow, for example, to fix a bug, there might be a branch `bugfix/fix-some-bugs` that borned from `master`. Then, the `master` keeps adding commits, `bugfix/fix-some-bugs` will have some commits too. Thus, there will be several commits ahead and also several commits behide `master`.  
If we want to move the commits that only on `bugfix/fix-some-bugs` but now on `master` before we merge it to `master`, simply do `git rebase master` on `bugfix/fix-some-bugs`.  
That's very useful if you want to keep your history graph more clear. Keeping rebase to `master` can also avoiding make that branch too far away from `master`, because it might cause some conflict.

# Append bunch of commits into another branch - Part 2

At first, we made a branch `branchA` after `A`, then we committed `B` on master.  
Continue, we made commit `C` and `D` on `branchA`. Then, we want to do a new feature based on `branchA`'s work, so wo created new branch `branchB` just after `D`, then committed `E` on that branch.
To keep sync with `master`, so we did a `git rebase` on `branchA`.
Considering following diagram:
```
         /---\   /---\
---------| A |---| B |---
  master \---/ | \---/  |         /---\   /---\
               |        \---------| C |---| D |
               |          branchA \---/   \---/
               |         
               |        /----\   /----\   /---\
               \--------| C' |---| D' |---| E |
                branchB \----/   \----/   \---/
```

What if we want to keep sync `branchB` with `branchA`?
Since `branchA` has been rebased, the hashed of `C`, `D` will be different on `branchB`'s, we marked it as `C'` and `D'`.
If we did `git rebase branchA` on `branchB`, the `C'` and `D'` will also be append to `branchA`.

For this cases, we need to use `git rebase onto`:
```
# # on branch `branchB`
# git rebase --onto branchA <D''s hash> HEAD
```

This way, only `E` will be rebase onto `branchA`.

```
         /---\   /---\
---------| A |---| B |---
  master \---/   \---/  |         /---\   /---\
                        \---------| C |---| D |---\
                          branchA \---/   \---/   |
                                                  |         
                                                  |        /---\
                                                  \--------| E |
                                                   branchB \---/
```

