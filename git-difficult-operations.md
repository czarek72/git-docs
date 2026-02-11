# Git Operations That Are Difficult or Problematic

Due to Git's architecture and the way it works, certain operations are inherently difficult or problematic. This document provides an exhaustive list of such operations.

## History Rewriting & Sensitive Data

- **Removing sensitive data from history** (passwords, API keys, tokens)
  - *Why difficult:* Git's immutable history means the sensitive data exists in every commit where it was present. Simply deleting it from the current version leaves it accessible in history. You must rewrite every commit from the point the data was introduced, changing all subsequent commit hashes. This requires tools like `git filter-branch` or BFG Repo-Cleaner, force-pushing the rewritten history, and coordinating with all collaborators to abandon their old clones. Even after rewriting, the data may persist in reflog, unpushed commits, and old clones.

- **Splitting a commit into multiple commits**
  - *Why difficult:* Commits are atomic units in Git's DAG. To split one, you must reset to the commit before it, manually stage subsets of changes, and create new commits. This rewrites history from that point forward, changing all descendant commit hashes. During interactive rebase, merge conflicts may arise because later commits expected the changes to be applied together. The process requires deep understanding of staging, partial commits, and conflict resolution.

- **Changing commit authorship across history**
  - *Why difficult:* Author information is baked into each commit's hash calculation. Changing the author of even one old commit cascades through the entire history, regenerating hashes for all descendant commits. You must use `git filter-branch`, `git rebase`, or `git filter-repo` to rewrite history, then force-push. All collaborators must re-clone or perform complex rebase operations to sync with the rewritten history.

- **Removing large files from history**
  - *Why difficult:* Even if you delete a large file in the latest commit, it remains in repository history, bloating the `.git` folder. Every clone downloads all historical versions. Removal requires rewriting every commit that referenced the file, which changes commit hashes and breaks the commit chain. Tools like BFG Repo-Cleaner or `git filter-repo` are necessary. After rewriting, you must force-push and ensure all collaborators re-clone, as their repositories still contain the old objects.

- **Merging/squashing commits in the middle of history**
  - *Why difficult:* Git commits form an immutable chain where each commit references its parent(s). Squashing commits in the middle requires interactive rebase, which replays all subsequent commits on top of the squashed commit. This changes every commit hash from that point forward and can generate conflicts if later commits depend on the intermediate states you're removing. Shared branches make this especially problematic since all collaborators' work is invalidated.

## Large File & Repository Scale Issues

- **Handling very large binary files**
  - *Why difficult:* Git's snapshot-based storage model keeps full copies of files in each commit where they change. Unlike text files, binaries can't be efficiently delta-compressed. A 500MB video file changed in 10 commits consumes ~5GB in the repository. Git LFS (Large File Storage) helps but requires additional setup, server support, and changes the workflow. Without LFS, clones are slow and disk usage explodes. Binary files also make operations like `git diff` and `git log -p` useless.

- **Working with repositories >1-2GB**
  - *Why difficult:* Git's design optimizes for distributed workflows where each clone has complete history. Large repositories mean every clone, fetch, and pull must transfer massive amounts of data. Git's object database becomes unwieldy, with millions of objects to pack and search. Operations like `git status`, `git log`, and `git gc` slow dramatically. Network transfers take forever, making initial clones prohibitive. Partial clone and sparse checkout help but have limitations and complexity.

- **Tracking frequently changing large binaries**
  - *Why difficult:* Binary files that change often (e.g., compiled artifacts, design files) create a new complete copy in repository history with each commit. Repository size grows exponentially rather than linearly. Git's delta compression is ineffective on binaries, especially encrypted or compressed formats. A 100MB binary changed in 100 commits adds 10GB to repository size. This is a fundamental mismatch with Git's storage model.

- **Splitting a monorepo into multiple repos** while preserving relevant history
  - *Why difficult:* Git repositories are self-contained with their entire history. Extracting a subdirectory with its history requires rewriting the entire commit graph to filter out unrelated changes, using tools like `git filter-repo` or `git subtree`. Cross-directory dependencies become external dependencies. Commits that touched multiple directories must be split or handled specially. References between the new repositories must be managed externally. Ongoing development during the split must be carefully merged.

- **Dealing with thousands of branches**
  - *Why difficult:* Git stores each branch as a reference (pointer to a commit). While lightweight individually, operations that iterate over all branches (like `git fetch`, `git branch -a`, or remote operations) scale poorly. Thousands of branches mean thousands of ref updates, ref comparisons, and potential conflict checks. Remote operations must negotiate which objects are needed across all branches. Tab completion and branch listing become unusable. The refs directory becomes a performance bottleneck.

## Partial/Selective Operations

- **Cloning only specific directories**
  - *Why difficult:* Git's architecture requires the complete history to verify integrity through the commit graph. While sparse checkout and partial clone features exist, they're complex workarounds. Sparse checkout still downloads all objects but only populates specific directories in the working tree. Partial clone delays object fetching but still requires understanding the full commit history. Neither approach truly clones "just a directory" - you're still working with the entire repository's metadata and commit history.

- **Checking out specific files from remote** without full clone
  - *Why difficult:* Git's distributed model assumes you have the full repository to work with. While you can fetch a single file via GitHub's raw URL or Git's archive feature, you can't meaningfully work with it in Git without the repository context. The file's history, its relationships to other files, and the commit information are all part of the repository as a whole. Git doesn't have a "download file at revision X" primitive - it's designed around complete repository operations.

- **Pulling changes from only certain directories**
  - *Why difficult:* Git operates at the repository level, not the directory level. A pull fetches all changes and merges or rebases your branch. You can't say "only merge changes to src/ and ignore changes to docs/." The commit DAG is indivisible - commits reference complete tree states. Attempting directory-selective pulls would break the integrity model. You'd need complex filtering and could easily create inconsistent repository states. Submodules or separate repositories are the proper solution, not selective pulls.

- **Tracking permissions beyond executable bit**
  - *Why difficult:* Git was designed for source code, where only execute permission matters for scripts. It deliberately doesn't track file ownership, group permissions, ACLs, or extended attributes because these are system-specific and meaningless across different machines/OSes. Storing them would break cross-platform compatibility. While you can script permission application with hooks, Git's core storage model simply doesn't include this metadata. Tools like `etckeeper` work around this by committing permission information in separate files.

## Branch & Merge Challenges

- **Renaming a branch and updating all references** everywhere (local, remote, CI/CD, documentation)
  - *Why difficult:* A branch name is used throughout the entire ecosystem: remote repositories, CI/CD pipelines, deployment scripts, pull requests, documentation, team members' local repositories, and protected branch rules. Git itself makes local renaming easy (`git branch -m`), but synchronizing the change requires deleting the old remote branch, pushing the new one, updating the default branch, reconfiguring CI/CD, updating protection rules, and coordinating with all team members to run `git branch -m` locally and update their upstream tracking. There's no atomic "rename everywhere" operation.

- **Moving commits between unrelated branches** without common ancestor
  - *Why difficult:* Git's merge and rebase operations expect branches to share a common ancestor in the commit graph. Unrelated branches have completely separate histories with no shared commits. Moving commits between them requires cherry-picking, which creates duplicate commits (different hashes) rather than moving them. This loses the original context and can create confusion. If the commits depend on files or state that don't exist in the target branch, conflicts arise that are difficult to resolve. The lack of common ancestry means Git can't perform its normal three-way merge.

- **Resolving complex three-way merge conflicts** with multiple conflicting changes
  - *Why difficult:* When multiple branches modify the same code in incompatible ways, Git's automatic merge fails. The three-way merge shows the common ancestor, your changes, and their changes, but understanding which combination is correct requires deep knowledge of both branches' intent. Complex conflicts may span multiple files with interdependencies. Conflict markers obscure code structure. Binary files can't be merged semantically. Incorrect resolution can introduce subtle bugs that tests might miss. Aborting and redoing the merge loses work.

- **Undoing a merge that's been pushed and pulled** by others
  - *Why difficult:* Once a merge is in shared history, it can't be simply removed without rewriting history. Using `git revert` on the merge creates a new commit that undoes the changes, but the merge commit remains in history and can cause confusion in future merges. Using `git reset` to remove the merge requires force-pushing, which breaks everyone else's history and requires coordination. If others have based work on the merge commit, their branches become orphaned. The merge may have been part of a deployment, complicating rollback.

- **Cherry-picking merge commits**
  - *Why difficult:* Merge commits have multiple parents, representing the combination of different branches. Cherry-picking expects a single parent to diff against. Git doesn't know which parent represents the "changes" you want to cherry-pick. You must specify `-m 1` or `-m 2` to choose a parent, but this loses the merge semantics - the resulting commit is a regular commit, not a merge. The relationships between branches are lost. If the merge resolved conflicts in specific ways, those resolutions aren't captured in the cherry-pick.

## Cross-Repository Operations

- **Moving history between repositories** while maintaining relationships
  - *Why difficult:* Git repositories are self-contained universes with their own commit graphs. Moving history means extracting a subset of commits while preserving their relationships, then importing them into another repository's commit graph. Commit hashes change because parent references change. Cross-repository references (like submodules or dependencies) become broken and must be manually fixed. Tools like `git filter-repo` can extract history, but merge commits, tags, and branch relationships complicate the process. There's no standard way to maintain "this commit came from repository X."

- **Synchronizing subtrees bidirectionally**
  - *Why difficult:* `git subtree` allows including one repository as a subdirectory of another, but it's designed for one-way incorporation. Changes in the parent repository can be split and contributed back, but this requires careful tracking of which commits affect the subtree and manual pushing. There's no automatic bidirectional sync. Merge commits from subtree operations complicate history. If both repositories evolve independently, keeping them synchronized requires repeated split/merge cycles that are error-prone. Conflicts must be resolved manually. Unlike submodules, subtrees copy history rather than reference it.

- **Sharing commits between forks** without full merge
  - *Why difficult:* Forks are independent repositories that diverge over time. Sharing specific commits means cherry-picking, which creates new commits with different hashes. This loses the connection to the original commit. If multiple related commits must be shared, each must be cherry-picked in order, and dependencies between them must be maintained. If the commits depend on other changes unique to the fork, they won't apply cleanly. There's no way to say "these commits are the same" across forks - Git sees them as duplicates.

- **Consolidating multiple repositories** with different histories
  - *Why difficult:* Each repository has its own independent commit history rooted at different initial commits. Merging them requires creating artificial connections between unrelated histories. Methods include: 1) `git merge --allow-unrelated-histories` which connects separate histories but creates a messy graph, 2) rewriting all commits from one repository to move files into a subdirectory, then merging. Both approaches require careful handling of conflicting files, tags, and branches. The consolidated repository loses the clean separation between projects. CI/CD and tooling must be reconfigured for the new structure.

## Recovery & Corruption

- **Recovering deleted remote branches** after force push (if no local copies exist)
  - *Why difficult:* When a branch is force-pushed or deleted on the remote, the commits it pointed to become orphaned. Without a local copy, the only recovery is through the remote server's reflog (if enabled) or backups. GitHub/GitLab keep commits for a limited time, but there's no standard interface to find orphaned commits. You must know the commit hash or have server access. If multiple people force-pushed, determining the "correct" state is subjective. Once the remote's garbage collection runs (typically 30-90 days), orphaned commits are permanently lost.

- **Fixing corrupted objects in .git directory**
  - *Why difficult:* Git's object database stores commits, trees, and blobs as compressed files identified by SHA-1 hashes. Corruption (from disk errors, interrupted operations, or bugs) can make the repository unusable. Fixing requires: 1) identifying which objects are corrupted via `git fsck`, 2) attempting to recover them from other clones, 3) understanding the object database structure to manually repair it. Missing commits break the history chain. Corrupted tree objects prevent checkout. Blob corruption loses file content. The process requires deep understanding of Git internals and may result in permanent data loss.

- **Recovering from badly resolved conflicts** that were committed
  - *Why difficult:* When conflicts are resolved incorrectly and committed, the mistake is baked into history. If caught immediately, you can amend or reset, but if other commits have been made on top, recovery requires: 1) identifying which files were incorrectly merged, 2) determining the correct state, 3) creating corrective commits, or 4) rewriting history with interactive rebase. If the broken merge was pushed and others have pulled it, history rewriting affects everyone. The incorrect merge may have cascading effects on later code that assumed the broken state.

- **Undoing force push on shared branches**
  - *Why difficult:* Force push overwrites remote history, replacing commits. If multiple people were working on the branch, their local branches now diverge from the remote. Undoing requires: 1) finding the old commit (via reflog or someone's local copy), 2) force-pushing it back, 3) coordinating with all collaborators to reset their local branches. Meanwhile, some people may have pulled the force-pushed version and based work on it. There's no way to tell Git "this force push was a mistake" - you just force push again, potentially overwriting legitimate work. Each force push creates more divergence and confusion.

## Metadata & Tracking Limitations

- **Tracking empty directories**
  - *Why difficult:* Git's data model tracks file contents and trees (directories), but only as necessary to organize files. An empty directory has no files, so Git has no object to store for it. The convention is to place a dummy file like `.gitkeep` in the directory, but this is a workaround, not a solution. Build systems and applications that expect specific directory structures fail when those directories don't exist after checkout. Git philosophically considers directories as emergent from files, not entities themselves.

- **Preserving file modification timestamps**
  - *Why difficult:* Git intentionally ignores file modification times, using checkout time instead. This is because timestamps are system-specific metadata that breaks reproducibility across machines. What matters for source code is content and history, not when a file was last touched locally. However, build systems and tools that rely on timestamps for incremental compilation or cache invalidation break. There's no standard way to preserve mtimes. Third-party tools like `git-restore-mtime` can reconstruct timestamps from commit history, but this is slow and not part of Git core.

- **Tracking symlinks on Windows** properly
  - *Why difficult:* Symlinks are a Unix concept that Windows handles differently (requiring admin privileges or Developer Mode). Git on Unix stores symlinks as special objects containing the link target. On Windows, Git has multiple modes: storing as text files with the target path, creating actual symlinks (if supported), or using junctions. Cloning a repository with symlinks on Windows often results in broken or converted links. Cross-platform teams face inconsistent behavior. Git can't paper over fundamental OS differences in how symbolic links work.

- **Handling case-sensitive renames** on case-insensitive filesystems (e.g., `file.txt` → `File.txt`)
  - *Why difficult:* Git's object model is case-sensitive - `file.txt` and `File.txt` are different files. However, macOS (default) and Windows use case-insensitive filesystems where these are the same file. Renaming `file.txt` to `File.txt` in Git creates a new tree object, but the filesystem sees no change. Git's status gets confused. The standard workaround is a two-step rename: `file.txt` → `temp.txt` → `File.txt`, but this is error-prone. Cloning such a repository on different OSes can fail. Git's cross-platform design can't fully hide OS filesystem semantics.

- **Storing custom metadata** per file/commit beyond standard fields
  - *Why difficult:* Git's commit object has a fixed structure: tree, parent(s), author, committer, and message. Adding custom fields would break Git's hash-based integrity model and wouldn't be understood by other Git tools. Similarly, file metadata is limited to mode (permissions) and content. Custom metadata like "reviewed-by," "ticket-number," or "priority" must be stored externally (in commit messages, notes, or separate files) and parsed by custom tooling. Git Notes provide a mechanism for adding metadata after-the-fact, but they're not part of commits themselves and don't replicate automatically.

## Advanced Rewriting Scenarios

- **Changing commit messages in pushed history**
  - *Why difficult:* Commit messages are part of the commit object's content used to calculate its hash. Changing a message changes the hash, creating a new commit. All descendant commits must be rewritten with updated parent references, changing their hashes too. For pushed branches, this requires force-push, which overwrites remote history. All collaborators must abandon their old clones or perform complex `git reset` or `git pull --rebase` operations. There's no in-place edit for commits - you must rewrite history from that point forward.

- **Reordering commits in shared branches**
  - *Why difficult:* Commit order is fundamental to Git's DAG - each commit references its parent(s). Reordering requires interactive rebase, which replays commits in a new order, generating entirely new commit hashes. If later commits depend on earlier ones, reordering can create conflicts. For shared branches, reordering breaks everyone's history - their local branches are based on the old order. Everyone must rebase their work onto the new history. Any branches based on the old commits become orphaned. The operation is inherently destructive to shared state.

- **Removing an entire branch from history** as if it never existed
  - *Why difficult:* If a branch was merged into main development branches, its commits are part of the permanent history. Removing them requires rewriting all commits since the merge, effectively creating a parallel universe where the branch never existed. Tools like `git filter-repo` can remove refs and rewrite history, but: 1) merge commits must be converted to regular commits or removed, 2) all subsequent commits get new hashes, 3) tags and branches must be rewritten, 4) everyone must re-clone. If the branch introduced files still used by later commits, removal causes conflicts.

- **Changing parent commits** (grafting/replacing)
  - *Why difficult:* Grafts and `git replace` allow making Git think a commit has different parents without actually rewriting history. This creates a virtual view of history that differs from the actual object database. Grafts are local (in `.git/info/grafts`) and aren't shared - other clones see the original history. Making grafts permanent requires filter-branch or filter-repo to rewrite commits with the new parents. This changes all descendant hashes. Grafts are a dangerous power-user feature that can create inconsistent repository states if misused.

- **Fixing incorrect merge base** after the fact
  - *Why difficult:* When branches are merged, Git finds their common ancestor (merge base) automatically. If Git chooses the wrong merge base (due to complex history), the three-way merge produces incorrect results. Fixing this after the fact requires: 1) identifying the correct merge base, 2) reverting the incorrect merge, 3) using `git merge -s ours` or `git merge-base` manipulation to force the correct base, 4) re-merging. If the incorrect merge is already in shared history, this requires rewriting or corrective commits. There's no "redo merge with different base" command - you must manually recreate the desired history.

## Collaboration Conflicts

- **Resolving divergent force-pushed branches** multiple people worked on
  - *Why difficult:* When multiple people force-push the same branch, each overwrites the others' history, creating divergent timelines. Person A's local branch, Person B's local branch, and the remote branch all have different histories from the same starting point. Git can't automatically reconcile these - they're considered unrelated histories. Resolution requires: 1) manually comparing branches to understand what work each person did, 2) choosing a canonical version, 3) manually cherry-picking or merging the other work into it, 4) getting everyone to agree and reset to the chosen version. Data loss is likely.

- **Synchronizing after conflicting rebases** by team members
  - *Why difficult:* If two team members both rebase the same shared branch onto main, they create incompatible histories with the same commits having different hashes. When one pushes, the other's push is rejected. Pulling creates a merge of the divergent rebased branches, doubling commits in a messy graph. The correct solution is for the second person to abort their rebase, pull the first person's rebase, then rebase their unique work on top. But determining which commits are "unique" vs. "already rebased" is difficult. Multiple conflicting rebases create exponential confusion.

- **Undoing someone else's force push** without losing work
  - *Why difficult:* Force pushes overwrite remote history, but if someone did it by mistake, you need to recover without losing their local work or others' work. You must: 1) find the old commit (from someone's local copy or reflog), 2) verify it's the correct state, 3) check if anyone has commits on top of the force-pushed version, 4) merge or rebase those newer commits, 5) force-push the recovered history back. Coordinating across team members requires communication outside Git. If anyone has local changes, those must be preserved through rebase. There's no "undo" button for force push.

- **Managing conflicts in binary files** (images, documents)
  - *Why difficult:* Git's conflict resolution shows textual differences with conflict markers `<<<<<<<`, `=======`, `>>>>>>>`. This is meaningless for binary files - you can't edit a PNG or PDF file by hand to resolve conflicts. Git can only say "both sides modified this file" and present both versions. Resolution requires: 1) choosing one version completely, 2) manually recreating the desired result in an appropriate editor, 3) staging the result. There's no semantic understanding of image or document structure. Merge tools like `git mergetool` can display both versions side-by-side, but you must manually decide and create the merged result. Binary diffs are uninformative.

## Performance & Scale

- **Searching commit messages/diffs across huge histories** (years of commits)
  - *Why difficult:* Commands like `git log --grep` or `git log -S` must scan through every commit in history, reading commit objects and optionally diffing contents. For repositories with decades of history (50k+ commits), this means processing gigabytes of data. Git's object database is optimized for integrity, not search speed. While packfiles compress objects, searching still requires decompressing and examining each commit. Adding `--all` to search all branches multiplies the work. There's no index of commit message text or code content - searches are always linear. Large monorepos make this practically unusable without external indexing tools.

- **Running operations on repositories with deep history** (50k+ commits)
  - *Why difficult:* Operations like `git log`, `git blame`, `git merge-base`, and `git rebase` must traverse the commit graph. With deep history, graph traversal becomes computationally expensive. `git blame` must follow file history through renames and modifications across thousands of commits. Graph algorithms like finding merge bases require walking possibly exponentially many paths. The `.git` directory contains millions of objects that must be managed. Packfile generation during `git gc` processes all history. Memory usage spikes. Even simple operations like `git status` slow down as Git's index grows.

- **Working with repositories containing millions of files**
  - *Why difficult:* Git's index (staging area) keeps track of every file in the working directory. With millions of files, the index file becomes enormous (hundreds of megabytes). Every `git status` must stat() millions of files to detect changes. Checking out commits must update millions of files, which is I/O bound. Tree objects become huge, and Git must recurse through deep directory hierarchies. Commands like `git add .` or `git reset --hard` take minutes. The working directory consumes massive disk space. Sparse checkout helps but adds complexity and doesn't fully solve the problem.

- **Optimizing repositories with many stale branches/tags**
  - *Why difficult:* Branches and tags are references to commits, and Git keeps the commits they point to plus all ancestors. Hundreds or thousands of stale branches mean Git retains vast amounts of old history that's never used. However, deleting branches/tags is dangerous - you might remove important work. Identifying truly stale branches requires external knowledge (is anyone still using it?). After deletion, running `git gc` to prune unreachable objects is slow and risky. If someone has the old branches locally, they can re-push them. Large numbers of refs also slow remote operations as Git must compare and transfer ref updates.

## Structural Limitations

- **Renaming files while preserving perfect history**
  - *Why difficult:* Git doesn't explicitly track renames - it stores snapshots of directory trees. When you rename a file, Git sees it as deleting the old file and creating a new file with similar content. Commands like `git log --follow` use heuristics (similarity index) to guess that files were renamed. If a file is renamed and modified significantly in the same commit, Git may not detect the rename. If renamed multiple times, tracking becomes unreliable. File history appears broken in GUI tools. The `--follow` flag only works for single files, not directories. This is a fundamental design trade-off for Git's storage model.

- **Tracking file moves across commits reliably**
  - *Why difficult:* Since Git infers moves heuristically rather than tracking them explicitly, reliability depends on the similarity threshold (default 50%). Heavy modification during move can break detection. Moving multiple files simultaneously confuses the heuristic. Git compares deleted and added files' contents, which is computationally expensive for large files or many moves. Cross-directory moves are treated the same as renames, but tools often display them differently. If a file is moved and another file with similar content is created at the old location, Git may incorrectly link them. There's no "this file definitely moved here" metadata.

- **Maintaining bidirectional links** between related commits
  - *Why difficult:* Git's DAG is unidirectional - commits point to their parents, never to children or related commits. Finding all children of a commit requires scanning all commits in the repository. There's no way to store "this commit fixes the bug introduced in commit X" as structured data - it must go in the commit message as text. Git Notes can add metadata after commits, but they're not part of the commit graph and don't replicate by default. Cross-references between branches or related work must be maintained externally. The immutable commit model prevents adding pointers to future commits.

- **Creating dependency graphs** between commits
  - *Why difficult:* Git's model captures temporal history (what was committed when) but not logical dependencies (this commit depends on that commit). The parent-child relationships form a graph based on branch/merge structure, not on actual code dependencies. Determining that "commit B depends on commit A" requires semantic analysis of the changes, not just Git metadata. Tools like `git log` can show which files changed, but not why or what the relationships are. For build systems or deployment pipelines that need dependency information, this must be computed externally or embedded in commit messages/tags.

- **Enforcing pre-commit validation** that can't be bypassed
  - *Why difficult:* Git hooks (like pre-commit) run locally on each user's machine. Users can skip them with `git commit --no-verify`, disable them by removing `.git/hooks`, or simply not install them in the first place. Server-side hooks (pre-receive) can enforce validation on push, but users can still create invalid commits locally, wasting time before discovering issues. There's no way to ensure code meets standards before it's committed without external CI/CD systems. The distributed nature of Git means each clone is autonomous - central enforcement only happens at push time, not commit time.

## Why These Operations Are Problematic

The fundamental issue is that Git's content-addressable storage means any change to history creates entirely new commits with new hashes, breaking the chain for everyone downstream. This is why operations affecting old commits are particularly problematic in collaborative environments.

### Git's Immutable History Model

Git's architecture is based on a directed acyclic graph (DAG) where each commit is identified by a SHA-1 hash of its contents, including:
- The tree of files
- Parent commit(s)
- Author and committer information
- Commit message
- Timestamp

Changing any of these elements changes the hash, which propagates to all descendant commits. This design choice prioritizes data integrity and distributed workflows over flexibility in rewriting history.
