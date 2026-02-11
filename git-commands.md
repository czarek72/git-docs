**Cheat Sheet for Git Console Commands**

**General**

Git is a version control system (for files). It's like the ability to save in computer games (in Git, the equivalent of a game save is a commit). Important: adding files to a "save" is a two-step process: first, add the file to the index (git add), then "save" (git commit).

Any file in the directory of an existing repository may or may not be under version control (tracked and untracked).

Tracked files can be in 3 states: unmodified, modified, staged (ready to commit).

**Key to Understanding**

The key to understanding Git is knowing about the "three trees":

- Working directory — the project's file system (the files you work with).
- Index — the list of files and directories tracked by Git, an intermediate storage for changes (editing, deleting tracked files).
- .git/ directory — all version control data for this project (the entire development history: commits, branches, tags, etc.).

A commit is a "save" (stores a set of changes made in the working directory since the previous commit). A commit is immutable; it cannot be edited.

All commits (except the very first) have one or more parent commits, since commits store changes from previous states.

**Basic Workflow Cycle**

1. Edit, add, or delete files (actual work).
2. Stage/add files to the index (tell Git which changes to commit).
3. Commit (save changes).
4. Return to step 1 or take a break.

**Pointers**

- HEAD — pointer to the current commit or branch (in any case, to a commit). Points to the parent of the commit that will be created next.
- ORIG_HEAD — pointer to the commit from which you just moved HEAD (with git reset ..., for example).
- Branch (master, develop, etc.) — pointer to a commit. When a commit is added, the branch pointer moves from the parent commit to the new one.
- Tags — simple pointers to commits. They do not move.

**Settings**

Before starting work, you need to set some configurations:

```
git config --global user.name "Your Name" # set the name that will sign commits
git config --global user.email "e@w.com"  # set the email that will appear in the committer description
```

If you are on Windows:

```
git config --global core.autocrlf true # enable conversion of line endings from CRLF to LF
```

**Specifying Untracked Files**

Files and directories that should not be included in the repository are listed in the .gitignore file. Usually, these are installed dependencies (node_modules/, bower_components/), build outputs (build/ or dist/), and similar files created during installation or execution. Each file or directory is listed on a new line; patterns can be used.

**Console**

How to use the Bash console in Windows, basic commands.

**Long Output in Console: Vim**

Some console commands produce very long output (e.g., git log -p fileName.txt). In such cases, the Vim editor starts in the console. It works in several modes, of which you are interested in insert mode (editing text) and normal (command) mode. To exit Vim to the console, enter :q in command mode. To switch to command mode from any other: Esc.

If you need to write something, press i — this switches to insert mode. To save changes, switch to command mode and type :w.

**Vim (Some Commands)**

- ESC — switch to command mode
- i — switch to insert mode
- ZQ (hold Shift, press sequentially) — exit without saving
- ZZ (hold Shift, press sequentially) — save and exit

**In command mode:**
- :q! — exit without saving
- :wq — save file and exit
- :w filename.txt — save file as filename.txt

**Console Commands**

**Create a New Repository**

```
git init             # create a new project in the current directory
git init folder-name # create a new project in the specified directory
```

**Clone a Repository**

```
git clone https://github.com/cyberspacedk/Git-commands.git    # clone remote repo into a directory with the same name
git clone https://github.com/cyberspacedk/Git-commands.git FolderName # clone into "FolderName"
git clone https://github.com:nicothin/web-design.git .        # clone into current directory
```

**View Changes**

```
git status              # show repository status (tracked, modified, new files, etc.)
git diff                # compare working directory and index (untracked files are IGNORED)
git diff --color-words  # compare working directory and index, show word differences (untracked files IGNORED)
git diff index.html     # compare working directory and index for a file
git diff HEAD           # compare working directory and the commit pointed to by HEAD (untracked files IGNORED)
git diff --staged       # compare index and HEAD commit
git diff master feature # see what was done in feature branch compared to master
git diff --name-only master feature # show only file names changed in feature vs master
git diff master...feature # show what was done in feature since it diverged from master
```

**Stage Changes**

```
git add .        # add all new, modified, deleted files from current directory and subdirectories to index
git add text.txt # add specified file to index (modified, deleted, or new)
git add -i       # interactive shell for staging selected files
git add -p       # show new/modified files one by one with changes and ask about staging
```

**Unstage Changes**

```
git reset            # remove all staged changes from index (working directory changes remain)
git reset readme.txt # remove staged changes for specified file (working directory changes remain)
```

**Undo Changes**

```
git checkout text.txt      # DANGEROUS: discard changes in file, revert to index state
git reset --hard           # DANGEROUS: discard changes; revert to HEAD commit (unstaged changes deleted from index and working directory, untracked files remain)
git clean -df              # delete untracked files and directories
```

**Commits**

```
git commit -m "Name of commit"    # commit staged changes with message
git commit -a -m "Name of commit" # stage tracked files (ONLY tracked, NOT new files) and commit with message
```

**Undo Commits and Move Through History**

All commits already pushed to a remote repository should be undone with new commits (git revert) to avoid problems for other contributors.

```
git revert HEAD --no-edit    # create a new commit that undoes the last commit without opening the editor
git revert b9533bb --no-edit # same, but for the commit with the specified hash
```

The following commands should ONLY be used if commits have not yet been pushed to a remote repository.

**WARNING! Dangerous commands, you can lose unstaged changes**

```
git commit --amend -m "Title"  # "recommit" the last commit, replace it with a new one with a different message
git reset --hard @~      # move HEAD (and branch) to previous commit, make working directory and index as they were at that commit
git reset --hard 75e2d51 # move HEAD (and branch) to commit with specified hash, make working directory and index as they were at that commit
git reset --soft @~      # move HEAD (and branch) to previous commit, but keep all changes in working directory and index
git reset --soft @~2     # same, but move back 2 commits
git reset @~             # move HEAD (and branch) to previous commit, keep working directory as is, index as it was at previous commit
git reset --keep @~      # move HEAD (and branch) to previous commit, reset index, but keep changes in working directory if possible (error if file changed between commits)
```

**Temporarily Switch to Another Commit**

```
git checkout b9533bb # switch to commit with specified hash (move HEAD, revert working directory to that commit)
git checkout master  # switch to commit pointed to by master
```

**Switch to Another Commit and Continue Working**

Requires creating a new branch starting from the specified commit.

```
git checkout -b new-branch 5589877   # create new-branch from commit 5589877
```

**Restore Changes**

```
git checkout 5589877 index.html  # restore specified file to state at specified commit (and stage this change)
```

**Copying Commits (Cherry-pick)**

```
git cherry-pick 5589877          # copy changes from specified commit to active branch and commit them
git cherry-pick master~2..master # copy last 2 commits from master to active branch
git cherry-pick -n 5589877       # copy changes from specified commit to active branch, but DO NOT COMMIT
git cherry-pick master..feature  # copy all commits from feature since it diverged from master to active branch and commit them (may cause conflict)
git cherry-pick --abort    # abort conflicted cherry-pick
git cherry-pick --continue # continue conflicted cherry-pick (after resolving conflict)
```

**Delete File**

```
git rm text.txt    # delete tracked, unmodified file and stage this change
git rm -f text.txt # delete tracked, modified file and stage this change
git rm -r log/     # delete all tracked files in log/ directory and stage this change
git rm ind*        # delete all tracked files starting with "ind" in current directory and stage this change
git rm --cached readme.txt # remove indexed file from tracking (FILE REMAINS) (often used for accidentally added files)
```

**Move/Rename Files**

Git does not have a rename operation. Renaming is seen as deleting the old file and creating a new one. The fact of renaming can only be determined after staging the change.

```
git mv text.txt test_new.txt # rename "text.txt" to "test_new.txt" and stage this change
git mv readme_new.md folder/ # move readme_new.md to folder/ (must exist) and stage this change
```

**Commit History**

Exit long log output: q.

```
git log master             # show commits in specified branch
git log -2                 # show last 2 commits in active branch
git log -2 --stat          # show last 2 commits and their change stats
git log -p -22             # show last 22 commits and their diffs
git log --graph -10        # show last 10 commits with ASCII branch graph
git log --since=2.weeks    # show commits from last 2 weeks
git log --after '2018-06-30' # show commits after specified date
git log index.html         # show commit history for index.html (commits only)
git log -5 index.html      # show last 5 commits for index.html (commits only)
git log -p index.html      # show commit history for index.html (commits and diffs)
git log -G'myFunction' -p  # show all commits where lines with myFunction changed (regex in quotes)
git log -L '/<head>/','/<\/head>/':index.html # show changes between specified regexes in index.html
git log --grep fix         # show commits with "fix" in message (case-sensitive, current branch only)
git log --grep fix -i      # show commits with "fix" in message (case-insensitive, current branch only)
git log --grep 'fix(ing|me)' -P # show commits matching regex in message (current branch only)
git log --pretty=format:"%h - %an, %ar : %s" -4 # show last 4 commits with custom formatting
git log --pretty=format:"%h %ad | %s%d [%an]" --graph --date=short # my output format, set as shell alias
git log master..branch_99  # show commits in branch_99 not merged into master
git log branch_99..master  # show commits in master not merged into branch_99
git log master...branch_99 --boundary -- graph # show commits from both branches since their divergence (divergence commit shown)

git show 60d6582           # show changes from commit with specified hash
git show HEAD~             # show previous commit in active branch
git show @~                # same as above
git show HEAD~3            # show commit 3 commits ago
git show my_branch~2       # show commit 2 commits ago in specified branch
git show @~:index.html     # show content of index.html at previous (from HEAD) commit
git show :/"footer"        # show newest commit with specified word in message (from any branch)
```

**Who Wrote a Line**

```
git blame README.md --date=short -L 5,8 # show lines 5-8 of file and the commits that added them
```

**Pointer (Branch, HEAD) History**

```
git reflog -20             # show last 20 HEAD pointer changes
git reflog --format='%C(auto)%h %<|(20)%gd %C(blue)%cr%C(reset) %gs (%s)' -20 # same, with action age
```

**Branches**

```
git branch                 # list branches
git branch -v              # list branches and last commit in each
git branch new_branch      # create new branch at current commit
git branch new_branch 5589877 # create new branch at specified commit
git branch -f master 5589877  # move master branch to specified commit
git branch -f master master~2 # move master branch 2 commits back
git checkout new_branch    # switch to specified branch
git checkout -b new_branch # create and switch to new branch
git checkout -B master 5589877 # move branch to specified commit and switch to it
git merge hotfix           # merge hotfix into current branch
git merge hotfix -m "Hotfix" # merge hotfix with commit message
git merge hotfix --log     # merge hotfix, open commit message editor, add merged commit messages
git merge hotfix --no-ff   # merge hotfix, forbid fast-forward, changes "stay" in hotfix, only merge commit in active branch
git branch -d hotfix       # delete hotfix branch (if already merged)
git branch --merged        # show branches already merged into active
git branch --no-merged     # show branches not merged into active
git branch -a              # show all branches (including remotes)
git branch -m old_branch_name new_branch_name # rename local branch
git branch -m new_branch_name # rename CURRENT branch
git push origin :old_branch_name new_branch_name # apply rename in remote repo
git branch --unset-upstream # finish renaming process
```

**Tags**

```
git tag v1.0.0               # create tag at HEAD
git tag -a -m 'To production!' v1.0.1 master # create annotated tag at master
git tag -d v1.0.0            # delete tag(s)
git tag -n                   # show all tags and 1 line of commit message
git tag -n -l 'v1.*'         # show all tags starting with 'v1.*'
```

**Temporarily Save Changes Without Commit**

```
git stash     # temporarily save uncommitted changes and remove from working directory
git stash pop # return stashed changes to working directory
```

**Remote Repositories**

There are two common ways to link a remote repository: via HTTPS and SSH. If SSH is not set up (or you don't know what it is), link via HTTPS (the repo URL should start with https://).

```
git remote -v              # list remotes linked to local repo
git remote remove origin   # remove remote named origin
git remote add origin https://github.com:nicothin/test.git # add remote named origin with specified URL
git remote rm origin       # remove remote
git remote show origin     # get info about remote named origin
git fetch origin           # fetch all branches from remote origin, but don't merge
git fetch origin master    # fetch only specified branch
git checkout --track origin/github_branch # create local branch github_branch from remote and switch to it
git push origin master     # push local master to remote origin
git pull origin            # pull changes from all branches in remote origin
git pull origin master     # pull changes from specified branch in remote origin
```

**Merge Conflict**

Suppose you have master and feature branches. Both have commits after diverging. You try to merge feature into master (git merge feature), but get a conflict because both changed the same line in index.html.

When a conflict occurs, the repo is in a conflicted merge state. You need to keep only the needed code in conflicting files, stage the changes, and commit.

```
git merge feature                # merge feature into active branch
git merge-base master feature    # show hash of last common commit for two branches
git checkout --ours index.html   # keep state from branch you are merging INTO (here, master)
git checkout --theirs index.html # keep state from branch you are merging FROM (here, feature)
git checkout --merge index.html  # show comparison of merged branches in file for manual editing
git checkout --conflict=diff3  --merge index.html # show comparison plus what was in the conflict at the divergence commit

git reset --hard  # abort conflicted merge, revert working directory and index to HEAD
git reset --merge # abort conflicted merge, keep changes not committed before merge
git reset --abort # same as above
```

**"Rebasing" a Branch**

You can "move" a branch's base to any commit. This is needed to bring changes from the main branch into the feature branch after it diverged.

You cannot rebase a branch that has already been pushed to a remote.

```
git rebase master # rebase active branch onto master (may cause conflicts)
git rebase --onto master feature # rebase active branch onto master, starting from where it diverged from feature
git rebase --abort # abort conflicted rebase, revert working directory and index
git rebase --continue # continue conflicted rebase (after resolving and staging conflict)
```

**How to Undo a Rebase**

```
git reflog feature -2        # view branch movement log, find last commit BEFORE rebase
git reset --hard feature@{1} # move branch pointer back one commit, update working directory and index
```

**Miscellaneous**

```
git archive -o ./project.zip HEAD # create archive of project file structure at HEAD
```

**Examples**

**Getting Started**

Create a new repository, first commit, link remote repo on github.com, push changes.

```
git init                      # create repo in this directory
touch readme.md               # create readme.md
git add readme.md             # add file to index
git commit -m "Start"         # create commit
git remote add origin https://github.com:nicothin/test.git # add empty remote repo
git push -u origin master     # push local repo to remote (master branch)
```

**"Amending" a Commit**

Only if the commit has not been pushed to remote.

```
subl inc/header.html          # edit and save header markup
git add inc/header.html       # stage changed file
git commit -m "Removed phone from header" # commit
# commit not yet pushed to remote
# realize something else needed in this commit
subl inc/header.html          # make changes
git add inc/header.html       # stage changed file (or git add .)
git commit --amend -m "Header: task #34 done" # redo commit
```

**Working with Branches**

You have master (public site version), working on a big task (redesign header), but need to fix a critical bug (wrong contact in footer).

```
git checkout -b new-page-header # create new branch for header changes and switch to it
subl inc/header.html            # edit header markup
git commit -a -m "New header: logo change" # commit (work not finished)
# discover bug in footer contact
git checkout master             # switch to master
subl inc/footer.html            # fix bug and save footer markup
git commit -a -m "Footer contact fix" # commit
git push                        # push critical fix to master in remote
git checkout new-page-header    # switch back to new-page-header
subl inc/header.html            # edit and save header markup
git commit -a -m "New header: navigation change" # commit (header work done)
git checkout master             # switch to master
git merge new-page-header       # merge new-page-header into master
git branch -d new-page-header   # delete new_page_header branch
```

**Branches, Merge, and Revert to Pre-Merge State**

You had a fix branch, fixed a bug, merged fix into master, but the fix broke something. Need to revert master to pre-merge state (bug is less critical than broken functionality).

```
git checkout master            # switch to master
git merge fix                  # merge fix into master
# see problem: some functionality broke
git checkout fix               # switch to fix (can't move it from master)
git branch -f master ORIG_HEAD # move master to commit pointed by ORIG_HEAD (before merging fix)
```

**Branches, Merge Conflict**

You have master, two parallel branches (branch-1 and branch-2) both edited the same place in the same file, branch-1 was merged into master, merging branch-2 causes a conflict.

```
git checkout master           # switch to master
git checkout -b branch-1      # create branch-1 from master
subl .                        # edit and save files
git commit -a -m "Fix 1"      # commit
git checkout master           # switch to master
git checkout -b branch-2      # create branch-2 from master
subl .                        # edit and save files
git commit -a -m "Fix 2"      # commit
git checkout master           # switch to master
git merge branch-1            # merge branch-1 into master (success)
git merge branch-2            # merge branch-2 into master (CONFLICT)
# Automatic merge failed; fix conflicts and then commit the result.
subl .                        # choose what to keep in conflicted files, save
git commit -a -m "Conflict resolved" # commit result
```

**Syncing a Fork with the Master Repository**

You forked a repo on github.com, made changes. The original (master) repo was updated. Task: pull changes from master repo (made after your fork).

```
git remote add upstream https://github.com:address.git # add remote: name — upstream, URL of master repo
git fetch upstream            # fetch all branches from master repo, don't merge yet
git checkout master           # switch to your master
git merge upstream/master     # merge fetched master branch into your master
```

**Committed to Master, But Should Have Committed to a New Branch**

IMPORTANT: works only if commit not yet pushed to remote.

```
git checkout -b new-branch    # create new branch from master
git checkout master           # switch to master
git reset HEAD~ --hard        # move master back 1 commit
git checkout new-branch       # switch back to new branch to continue work
```

**Restore File Content to State in a Commit (known commit hash)**

```
git checkout f26ed88 -- index.html # restore index.html to state at commit f26ed88, stage this change
git commit -am "Navigation fixs"   # commit
```

**Git Asks for Login and Password Every Time**

This is about login + password, not a passphrase. Happens because Git does not save your password for HTTPS by default.

Simple solution: tell Git to cache your password.

**.gitattributes**

```
* text=auto

*.html diff=html
*.css  diff=css
*.scss diff=css
```
