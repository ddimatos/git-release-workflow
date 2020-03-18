| [Home](README.md) ▸ **Git Commands** |
|-----|
# Git Commands

Occasionally I come across a command only later to forget it, so I am
sharing some interesting commands and my often used ones. 

# Rebase

Learn about the two;![Merge vs Rebase](https://www.atlassian.com/git/tutorials/merging-vs-rebasing) also
another good ![resource](https://coderefinery.github.io/git-branch-design/01-rebase/).

Credit to these rebase commands comes from this ![repo](https://github.com/Junjie-Chen/git-rebase/blob/master/README.md).

How is `git rebase`
* `git rebase master` - Rebase the current branch to the tip(end) of the `master` branch
* `git rebase master new-feature` - Rebase the `new-feature` branch to the tip of the `master` branch
* `git rebase --onto newbase upstream branch` - Rebase the `upstream` branch to the tip of the `newbase` branch
* `git rebase --onto 9291f0c88 master new-branch` - Undo a rebase by rebasing to a former `merge-base` SHA
* `git rebase --interactive master new-feature` - Modify commits as they are being replayed
* `git rebase --interactive HEAD~3` - Rebase last 3 commits onto the same branch but with the opportunity to modify them
* `git rebase --continue` - Record changes after resolving rebase conflicts and keep going down the chain of commits that are waiting to be rebased
* `git rebase --skip` - Bypass this current commit and keep going with the rest of the commits in the chain that are waiting to be rebased
* `git rebase --abort` - Cancel rebasing the current branch

Useful commands for `git rebase`
* `git log --graph --all --decorate --oneline` - Visualize branch commits
* `git merge-base master new-feature` - Return the commit where the topic branch diverges
* `git reset --hard ORIG_HEAD` - Undo a rebase unless `ORIG_HEAD` has changed again that is a temporary variable used by Git to keep track of where things were when it's doing a rebase, a reset or a merge
* `git pull --rebase` - Fetch from remote then rebase instead of merging
* `git pull --rebase=preserve` - Preserve locally created merge commits and not flatten them
* `git pull --rebase=interactive` - Modify commits

# Remove Untracked Files in Git
Tracked files are files that have been added and committed, others in a 
working directory are untracked. To avoid deleting them, either track them with
git add <file> or add them to .gitignore.

Check before you remove files or directories with a 'dry run' using `-n`, `-d`
for directories, `-f` for force and `-i` for interactive:

```
git clean -d -f -n
```

Limit the scope of `git clean -d` with a target directory by passing the
directory to scan.
```
git clean -d -n directory
```

# Changing a commit message

## Local amend
If the commit is only in your local repository (most recent) and has not been 
pushed to GitHub, you can amend the commit message.

In text editor, edit the commit message, and save the commit
```
git commit --amend
```

Optionally, you can avoid the editor and use `-m`:
```
git commit --amend -m "commit message"
```

## Remote amend
If you pushed a commit to GitHub, you will have to force push a commit 
with an amended message. Note, many times protected branches disable forced
pushing. 

```
git commit --amend
git push --force
```

# Multiple Amend
If you need to amend messages for multiple commits, use interactive rebase,
and force push to change the commit history. Note, rewriting history of a 
branch that has been checked out by others can be problematic. 

`n` is the last n commits , ie 3
```
git rebase -i HEAD~n
```

You will be displayed something like so in an editor: 
```
pick a123456 Some commit
pick b123456 Another commit
pick c123456 More commits

# Rebase a123456..c123456 onto d123456
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

Replace the word `pick` with `reword` on each commit you want to change.
Save and close the commit list file; what follows is subsequent edits after closing.
In each commit file, edit the commit message, save and close the file. 
Force push the amended commits:
```
git push --force
```

# Amend the last commit without changing commit message

The `--no-edit` allows you to add commits to last commit message without 
changing it and without launching an editor.

```
git commit --amend --no-edit
```
# How to Squash Commits
When submitting a pull request, you must squash your commits before we merge. Squasing commits can be intimidating at first, once you figture it out you will see how helpful it is and clean it keeps the git commit history.  

## Squashing Options
* If you are using GitHub for a pull request, it provides you an option to squash before submitting a pull request. 
  * TODO: GitHub instructiosn and images
* From the terminal:
    *  Make sure your branch is up to date with the master branch
    * Run git rebase -i master. 
      * This will ensure that your commit is at the top of the branch pulling in the latest changes.
    * You should see a list of commits, each commit starting with the word "pick"
    * Make sure the first commit says "pick" and change the rest from "pick" to "squash". 
      * This will squash each commit into the previous commit, which will continue until every commit is squashed into the first commit.
    * Save and close the editor
    * It will give you the opportunity to change the commit message
    * Save and close the editor again
    * Then you have to force push the final, squashed commit: git push --force-with-lease origin
