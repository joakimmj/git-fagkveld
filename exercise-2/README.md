# Exercise 2 - Changing commits, rebasing and merging

For this exercise, we have created a simple calculator you can run in the
browser. The code is just for having something to work with for the exercises,
and you don't need to do anything with the code, other than what is mentioned in
the tasks.

We will start working on the branch `feat/trigonmetric-functions`, and doing
some changes there. Eventually, we will merge this branch into `master`.

## Part 1. Amend the last commit

For the first part, we want to change the most recent commit in the branch
`feat/trigonmetric-functions`. The change you should do is to make the check for
NaN in `calc.js` use `Number.isNaN` instead of `isNaN`, as this function is more
strict. They take the same argument, so you don't need to change anything else.

```
-    if (!isNaN(result)) {
+    if (!Number.isNaN(result)) {
```

To include this into the previous commit you can run commit with the `--amend`
option:

```
git commit --amend
```

Note that you need to add the changes you want to include, or run commit with
the `-a` option to include all changes, just like you would with a normal
commit.

## Part 2. Fix an earlier commit

But what if the thing you want to change isn't from the most recent commit, but
further back in the log? It's still possible to change these commits, but you
will have to do it a bit differently. Now, you will first have to use the
`--fixup` option to commit, and then do an interactive rebase. This part will
cover the commit.

The change you should do is to hide the second number field by setting display
to none, instead of deactivating it. Like this:

```
-    number2Field.disabled = operationField.value === 'sin';
+    number2Field.style.display = operationField.value === 'sin' ? 'none' : '';
```

We would like this change to be a part of the commit "Deactivate the second
number field when the operation is sin". As mentioned, we do this by running
commit with the `--fixup` option. For this command, we need to specify the
revision of the commit we would like to change. You can specify the revision
however you like, as long as it points to that commit. In this case, we can
provide the revision `:/Deactivate`. This means the most recent commit which
includes `Deactivate` in the commit message.

```
git commit --fixup=:/Deactivate
```

This will not change the original commit immediately though, since that would
change the history, which is not something the commit command should do.
Instead, a special commit that indicates that it belongs to an earlier commit
has been created. To combine these two commits we need to run an interactive
rebase. This will be the next task.

## Part 3. Interactive rebase

Interactive rebase can be used to change previous commits in several different
ways. In this case, we will use it to combine the commit created in the
previous task with an earlier commit. When running an interactive rebase, you
specify a revision and can change all the commits after that revision up to the
commit you started the rebase from.

The new fixup commit created in the previous task will contain the commit
message from the original commit, so the revision `:/Deactivate` now points to
this commit. Therefore, we have to specify the revision differently. To look for
this message after the most recent commit, we can specify `HEAD~^{/Deactivate}`.
This means a commit containing Deactivate which comes after the commit `HEAD~`,
which is the parent of the most recent commit. If you find this impractical or
hard to remember, you can run `git log`, copy the commit hash and use that
instead.

When running the rebase, we need to use the `--autosquash` option. This will
instruct rebase to treat the fixup commit specially so it will be included in
the previous commit. Without this option, it will just be treated as a normal
commit. This option can also be set in the config file with the key
`rebase.autoSquash`, so you don't have to specify it each time.

Now run the rebase command:

```
git rebase --autosquash HEAD~^{/Deactivate}
```

This will open your editor with a list of commits in reverse order. You can see
that since we used the `--autosquash` option, the fixup commit we made is
prefixed with `fixup` instead of `pick` like the others, and it is placed right
after the original commit. This instructs rebase to merge these two commits.

You can change the contents here if you want to make more changes, e.g. edit a
commit or merge more commits, but for now we stick with fixing that one commit.
So all that's left to to is to exit the editor (if you made any changes, you
also need to save of course). If you use the default editor of git, which is
vim, this can be done with :q<Enter>. After exiting the editor, the rebase
operation will run, and the two commits will be merged.

## Part 4. Move the last commit into a new branch

Before we merge this branch into master, we realize that the last commit in the
branch doesn't really belong as part of this feature, so it should be in a
separate branch. Therefore, we would like to split this branch into two
branches, where the current branch, `feat/trigonmetric-functions` keeps all the
commits except the last, and the new branch only contains the last commit on
top of master.

First we start by making the new branch. However, we don't want to check out the
branch yet. This can be done with the branch command.

```
git branch feat/dont-use-nan-result
```

Now we have a new branch, with exactly the same commits as the current branch.
We can therefore remove the last commit of the current branch to get it to the
state we want it. We can do this with the reset command, which changes the
current branch to the specified revision.

By using the `--keep` option, we change the files in the working directory as
well, since we don't want to keep the changes from the commit we remove. If you
have seen the `--hard` flag, that does a similar function. The difference is
that `--hard` will remove all changes in the working directory, while `--keep`
keeps local changes that hasn't been committed.

Since we should only remove the last commit, the revision we want to specify is
the commit before that, i.e. the parent commit of the current commit. The
current commit can be referenced with `HEAD`, and by adding the suffix `~` we
specify the parent. So we end up with:

```
git reset --keep HEAD~
```

Now `feat/trigonmetric-functions` is in the state we want it, so we can move on
to the new branch.

```
git checkout feat/dont-use-nan-result
```

This contains all the commits we previously had. The branch is based out of
master, so we want to remove all the commits between master and the most recent
commit. We can do this using the rebase command.

The option we want to use to make rebase remove these commits is `--onto <rev>`.
This tells rebase to start the operation from that revision. From that revision
it will apply all the commits that comes between the other revision we specify,
and the current commit.

The revision we want to specify for onto is master, since the branch is based
off of master. If it was based off of an older commit from master, this rebase
would make the newer commits included in this branch. To avoid this you can
instead specify the last commit in the branch that was from master, i.e. the
commit after the last commit you want to remove. This commit can be found with
the command `git merge-base master HEAD`. For this exercise, we will just use
master though.

In addition to that revision, you also need to specify a revision for which
commits you want to include in the rebase. Like in the previous task, the
commits which will be included is the ones that comes between this revision and
the current commit. Since we only want to include the last commit, we can
specify `HEAD~` like we did for the reset command. So we end up with:

```
git rebase --onto master HEAD~
```

Since this is not an interactive rebase, no editor will open up. The operation
will run, and you end up with a branch containing only the last commit.

Of course, it's also possible to move multiple commits to another branch by
specifying a different revision to reset and rebase. You can also move other
commits than the last, but to do that you might need to run an interactive
rebase to reorder the commits first, or do it in a completely different way.
This is left as a bonus exercise, and will not be explored further here.

## Part 5. Merge into master and resolve conflicts

The operations we've covered so far in this exercise are the ones you'll most
commonly use to keep your branch history nice and clean. Now that we've cleaned
up the branch a bit, the final thing we'll do with it is to merge it back in to
the master branch.

To do this, switch back to the master branch by running `git checkout master`
and then perform the merge by running `git merge feat/trigonmetric-functions`.

When you perform merges, it's normal to encounter conflicts caused by changes
introduced (usually by other people) on the branch you're merging into. If git
can't figure out how to cleanly combine the changes, you'll have to do it
yourself. In this case, there will be a few small conflicts you'll have to solve
manually, both in calc.js and in index.html.

If you open exercise-2/calc.js, you'll see that git has inserted some markers
for you:
```
<<<<<<< HEAD
    case 'minus':
      return number1 - number2;
=======
    case 'sin':
      return Math.sin(number1);
>>>>>>> feat/trigonmetric-functions
```

This indicates the conflicting sections from the master and the feature branch,
respectively. Since we in this case want both changes to be there, the correct
strategy is simply to remove the conflict markers so all three switch cases
remain. There should be one more similar conflict to solve, in index.html.

After you've solved all the conflicts, use `git add` to add the conflicting
files (calc.js and index.html) to the staging area to include them in the merge
commit. Finish the merge by running `git commit`.
