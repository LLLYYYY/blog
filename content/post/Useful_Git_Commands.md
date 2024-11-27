---
title: Useful Git Commands
date: 2024-11-26
categories:
- General
tags:
- random_notes
---

`git log`: Output git commits history.

`git log --graph `: Output git commits history in graph mode.

`git add <filename>`: Add files to the git tracking and committing system.

`git status`: Check Git Status

`.gitignore`: Input files that will be ignored by Git. **But if files are already inside the git tracking system, it won't delete it from git. We have to use `git rm filename` to remove it.**

`git diff`: check file changed that is not staged. 

`gif diff --staged`: check file changed that is staged.

`git add --all`: add all changes to the git tracking.

`git commit`: Submit changes to git.

`git commit -a`: Submit changes with all changes. Still need to call `git add all`

`git commit --amend`: Revert and re-commit the last git submission.

`git rm –cached < filename >`: Delete file from git submission. But does not delete local file.

`git reset – < file >`: change the current HEAD pointers to previous commits. Depends on --hard or --soft, will or will not change the local file.

`git checkout -- <file>`: Change to local file to specific commits. Does not move HEAD pointer in Git.

`git remote -v`: Output remote server and its URL.

`git remote add <short_name> <url>`:

`git remote rm <short_name>`:

`git fetch <remote-name>`:  Receive the remote server's update. Will put into <remote-name>/branch.

`git pull`: Automatically fetch and then merge into current.

`git push <remote-name> <branch-name>`: push to the remote server.

`git push <remote-name> --tags`: push to the remote server with tags information.

`git tag -d <tagname>`: Delete tags information

`git checkout master`: Checkout the master branch.

`git merge <other_branch_name>`: Git will decide which branch will become its ancestor.

If we have conflicts in merge:

`git mergetool`: GUI tool used to fix conflicts. Then use `git merge` to merge.

`git branch`: Output all branches information.

It is OK to repeatedly merge other branches to one branch. For example, we can repeatedly merge *test_branch* to *master* branch. 

`git push origin --delete <branch_name>` Delete remote branch.

`git rebase -i HEAD~3` Change the last three commits, very useful tool to re-generate commits history.

### Added after March, 2023:

`git worktree add <path> <branch_name>`: Checkout another branch in a separate folder. The current branch folder and the new checkout folder will co-exist, and sync with the updates (for different branches). 

`git worktree list`: List all currently checkout work trees. 

`git worktree remove <worktree>`: Remove the work tree from the file system.

### Signing related:

`gpg --list-keys` && `git config --global user.signingkey <your gpg key id>`: Check you gpg key and import it to your git. Make sure the gpg key email matches your git registered email. 

`git config --global commit.gpgsign true`: Auto sign by default. 

### Just interesting new command: 

`git push --force-with-lease`: Force push if the reference tree is not changed by others. Safer force push.  

`git diff --word-diff`: Word diff instead of line diff. 

`git config --global branch.sort -committerdate`: Sort branch based on commit date instead of alphabetic. 

`git maintenance start`: On your most commonly used commit, auto run git maintenance jobs such as prefetching periodically. 

`git clone --filter=blob:none`: Download the git commit tree and the latest blob, but skipped the history blob at the time. Git will fetch for blob if you ask for history commit data. 

If you are working with super large monorepo, you can use `scalar` command, an alternative `git` command that wraps up some `git` commands optimized for large repos. 

If you have a series of commits, and you want to patch something up in the early commit, you can use: 

```bash
git commit -a --fixup=<old_commit_hash>
# here, it will create a new patch commit on top of the head. 
git rebase --autosquash main
# apply the patch commit to the old commit, and then rebase the commit series. 
# it runs rebase in the background, if it encounters something that cannot fix, that it will prompt you for rebasing. 
```