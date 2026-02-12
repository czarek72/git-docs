# Git Internals: Hands-On Tutorial with Plumbing Commands

## Introduction

This tutorial teaches Git internals by having you build a repository from scratch using low-level "plumbing" commands. Follow along step-by-step, executing each command to see how Git actually works under the hood.

By the end, you'll understand blobs, trees, commits, branches, and how they all connect to form Git's elegant data model.

## Setup: Create a Test Repository

```bash
# Create and enter test directory
mkdir git-internals-test
cd git-internals-test

# Initialize empty Git repository
git init

# Verify .git directory was created
ls -la .git/
```

You should see directories like `objects/`, `refs/`, and files like `HEAD`, `config`.

## Step 1: Creating Blobs (File Content)

Git stores file content in **blob** objects. Let's create one manually.

```bash
# Create a blob from text
echo "Hello Git" | git hash-object -w --stdin
```

**Output:** `e965047ad7c57865823c7d992b1d046ea66edf78`

This SHA-1 hash is the blob's unique identifier. The `-w` flag writes it to the database.

### View the Blob

```bash
# View blob content
git cat-file -p e965047
```

**Output:** `Hello Git`

### See Where It's Stored

```bash
# Check the object file location
ls -la .git/objects/e9/
```

**Output:** `65047ad7c57865823c7d992b1d046ea66edf78`

Git splits the SHA-1: first 2 characters become the directory, remaining 38 become the filename.

### Create More Blobs

```bash
# Create blob from a real file
echo "This is file1 content" > file1.txt
git hash-object -w file1.txt

# Create another
echo "This is file2 content" > file2.txt
git hash-object -w file2.txt

# List all objects created
find .git/objects -type f
```

**Key Point:** Blobs only store content, not filenames or metadata.

## Step 2: Creating a Tree (Directory Structure)

A **tree** object represents a directory. It maps filenames to blobs (or other trees).

### Update the Index First

To create a tree, we need to stage files in the index:

```bash
# Add file1.txt to index (stage 0, mode 100644 = regular file)
git update-index --add --cacheinfo 100644 \
  $(git hash-object -w file1.txt) file1.txt

# Add file2.txt to index  
git update-index --add --cacheinfo 100644 \
  $(git hash-object -w file2.txt) file2.txt

# View what's staged
git ls-files --stage
```

**Output:**
```
100644 a0423896973644771497bdc03eb99d5281615b51 0	file1.txt
100644 9d13c8d3e2f6a9b4c14a... 0	file2.txt
```

### Create Tree from Index

```bash
# Write tree object from current index
git write-tree
```

**Output:** `<tree-sha>` (e.g., `d8329fc1cc938780ffdd9f94e0d364e0ea74f579`)

### View the Tree

```bash
# View tree contents (use your actual tree SHA)
git cat-file -p d8329fc1
```

**Output:**
```
100644 blob a0423896973644771497bdc03eb99d5281615b51	file1.txt
100644 blob 9d13c8d3e2f6a9b4c14a...	file2.txt
```

**Key Point:** Trees map filenames to blobs. This is how Git stores directory structure.

## Step 3: Creating a Commit

A **commit** points to a tree and adds metadata: author, message, timestamp, parent commit(s).

### Create Your First Commit

```bash
# Create commit object
# Replace TREE-SHA with your actual tree SHA from previous step
echo "First commit" | git commit-tree d8329fc1
```

**Output:** `<commit-sha>` (e.g., `7f3e2a1b9c8d5f6a7e8b9c0d1e2f3a4b5c6d7e8f`)

### View the Commit

```bash
# View commit object (use your commit SHA)
git cat-file -p 7f3e2a1
```

**Output:**
```
tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
author Your Name <your@email.com> 1707648234 +0000
committer Your Name <your@email.com> 1707648234 +0000

First commit
```

### Make Second Commit with Parent

```bash
# Modify file
echo "Updated content" > file1.txt

# Stage new version
git update-index file1.txt

# Create new tree
TREE2=$(git write-tree)

# Create commit with parent (use your first commit SHA)
echo "Second commit" | git commit-tree $TREE2 -p 7f3e2a1
```

**Output:** `<new-commit-sha>`

### View Second Commit

```bash
# View it (use your second commit SHA)
git cat-file -p <second-commit-sha>
```

**Output:**
```
tree <new-tree-sha>
parent 7f3e2a1b9c8d5f6a7e8b9c0d1e2f3a4b5c6d7e8f
author Your Name <your@email.com> 1707648345 +0000
committer Your Name <your@email.com> 1707648345 +0000

Second commit
```

**Key Point:** Commits link to parents, forming a history chain.

## Step 4: Creating a Branch

A **branch** is simply a pointer to a commit. It's stored as a file containing a commit SHA.

### Create Branch Manually

```bash
# Create master branch pointing to second commit (use your commit SHA)
echo "<second-commit-sha>" > .git/refs/heads/master

# Verify
cat .git/refs/heads/master

# List branches
git branch
```

**Output:** `* master`

### Update HEAD

```bash
# Point HEAD to master branch
echo "ref: refs/heads/master" > .git/HEAD

# Verify
cat .git/HEAD
```

**Output:** `ref: refs/heads/master`

### Check Your Work

```bash
# View commit history
git log --oneline

# View files in working tree
git ls-tree -r HEAD
```

**Key Point:** Branches are just 41-byte files (40-char SHA + newline) in `.git/refs/heads/`.

## Step 5: Creating a Subdirectory Tree

Let's add nested directories to understand tree-in-tree references.

### Create Subdirectory

```bash
# Create subdirectory and file
mkdir src
echo "Source code here" > src/main.py

# Stage the new file
git update-index --add --cacheinfo 100644 \
  $(git hash-object -w src/main.py) src/main.py

# Create tree
TREE3=$(git write-tree)

# View tree structure
git cat-file -p $TREE3
```

**Output:**
```
100644 blob a0423896...	file1.txt
100644 blob 9d13c8d3...	file2.txt
040000 tree <subtree-sha>	src
```

### View Subtree

```bash
# Extract subtree SHA from above output, then view it
git cat-file -p <subtree-sha>
```

**Output:**
```
100644 blob 5c7e2a1b...	main.py
```

**Key Point:** Trees can contain other trees (subdirectories). The mode `040000` indicates a tree object.

## Step 6: Understanding Object Types

Git has 4 object types. Let's examine each:

### Check Object Types

```bash
# Check blob type
git cat-file -t $(git hash-object file1.txt)
```
**Output:** `blob`

```bash
# Check tree type
git cat-file -t $TREE3
```
**Output:** `tree`

```bash
# Check commit type  
git cat-file -t HEAD
```
**Output:** `commit`

### Object Sizes

```bash
# Check blob size
git cat-file -s $(git hash-object file1.txt)

# Check commit size
git cat-file -s HEAD
```

**Key Point:** Use `cat-file -t` for type, `-s` for size, `-p` for content.

## Step 7: The Index (Staging Area)

The index is a binary file at `.git/index` that tracks what goes into the next commit.

### View Index Contents

```bash
# Show staged files with metadata
git ls-files --stage
```

**Output:**
```
100644 a0423896... 0	file1.txt
100644 9d13c8d3... 0	file2.txt
100644 5c7e2a1b... 0	src/main.py
```

### Manual Staging

```bash
# Create new file
echo "Documentation" > README.md

# Stage it manually
git update-index --add --cacheinfo 100644 \
  $(git hash-object -w README.md) README.md

# Verify
git ls-files --stage
```

### Remove from Index

```bash
# Remove from index (but keep file)
git update-index --force-remove README.md

# Verify
git ls-files --stage
```

**Key Point:** `git add` is just a wrapper around `hash-object` and `update-index`.

## Step 8: Making a Proper Commit

Now use the knowledge to make a complete commit:

```bash
# Stage all current files
git add .

# Create tree from index
TREE4=$(git write-tree)

# Create commit with parent
COMMIT3=$(echo "Third commit: added subdirectory" | \
  git commit-tree $TREE4 -p $(cat .git/refs/heads/master))

# Update master branch
echo $COMMIT3 > .git/refs/heads/master

# View log
git log --oneline --graph
```

**Key Point:** `git commit` automates: `write-tree` + `commit-tree` + updating branch ref.

## Step 9: Understanding References

References (refs) are pointers to commits.

### Types of References

```bash
# Local branches
ls .git/refs/heads/

# Remote branches (if any)
ls .git/refs/remotes/

# Tags (if any)
ls .git/refs/tags/
```

### Create a Tag

```bash
# Lightweight tag (just a pointer)
echo $(cat .git/refs/heads/master) > .git/refs/tags/v1.0

# Verify
git tag
cat .git/refs/tags/v1.0
```

### Annotated Tag

```bash
# Create annotated tag object
git mktag << EOF
object $(cat .git/refs/heads/master)
type commit
tag v2.0
tagger Your Name <your@email.com> 1707648456 +0000

Version 2.0 release
EOF
```

This creates a tag object (4th object type) and stores it.

**Key Point:** Lightweight tags are just refs. Annotated tags are full objects with metadata.

## Step 10: Resolving References

Git provides `git rev-parse` to resolve any reference to a SHA.

### Resolve Various References

```bash
# Resolve HEAD
git rev-parse HEAD

# Resolve branch
git rev-parse master

# Resolve tag
git rev-parse v1.0

# Get parent commit
git rev-parse HEAD^

# Get grandparent
git rev-parse HEAD~2

# Get tree from commit
git rev-parse HEAD^{tree}

# Get blob from path
git rev-parse HEAD:file1.txt
```

**Key Point:** `rev-parse` is essential for scripting and understanding Git references.

## Step 11: The Reflog

The reflog tracks all reference updates, providing a safety net.

### View Reflog

```bash
# View HEAD reflog
git reflog

# View branch reflog
git reflog show master
```

### Check Reflog Storage

```bash
# View raw reflog file
cat .git/logs/HEAD

# View branch log
cat .git/logs/refs/heads/master
```

**Output format:**
```
old-sha new-sha Author <email> timestamp action: message
```

**Key Point:** Reflog tracks every change to HEAD and branches for ~90 days.

## Step 12: Porcelain vs Plumbing

Compare what we did manually vs. porcelain commands:

| What We Did | Porcelain Equivalent |
|-------------|---------------------|
| `hash-object -w` + `update-index` | `git add` |
| `write-tree` + `commit-tree` | `git commit` |
| `echo SHA > .git/refs/heads/branch` | `git branch` |
| `echo "ref:" > .git/HEAD` | `git checkout` |
| `cat-file -p` | `git show` |
| `ls-files --stage` | `git status` (partially) |

## Step 13: Complete Workflow Example

Let's do a complete workflow using plumbing commands:

```bash
# 1. Create and stage files
echo "Feature X code" > feature.py
git hash-object -w feature.py
git update-index --add feature.py

# 2. Create tree
TREE=$(git write-tree)

# 3. Create commit
COMMIT=$(echo "Add feature X" | git commit-tree $TREE -p HEAD)

# 4. Update branch
git update-ref refs/heads/master $COMMIT

# 5. Verify
git log --oneline -1
git ls-tree HEAD
```

**Key Point:** This is exactly what `git add` + `git commit` does internally.

## Step 14: Inspecting the Object Database

Let's explore all objects created:

### List All Objects

```bash
# Find all objects
find .git/objects -type f | grep -v pack
```

### Count Objects

```bash
# Get statistics
git count-objects -v
```

**Output:**
```
count: 15
size: 8
in-pack: 0
packs: 0
size-pack: 0
prune-packable: 0
garbage: 0
```

### Verify Repository Integrity

```bash
# Check for corruption
git fsck --full
```

**Key Point:** Git stores each version as a separate blob. Packfiles optimize this later.

## Step 15: Creating a Branch (Properly)

Now create a branch using understanding of refs:

```bash
# Get current commit
CURRENT=$(git rev-parse HEAD)

# Create new branch
git update-ref refs/heads/feature-branch $CURRENT

# List branches
git branch

# Switch to new branch (update HEAD)
git symbolic-ref HEAD refs/heads/feature-branch

# Verify
git branch
cat .git/HEAD
```

**Key Point:** Branch creation is just creating a ref file. Checkout updates HEAD.

## Step 16: Making Changes on New Branch

```bash
# Make change
echo "Feature branch work" > feature-branch-file.txt

# Stage and commit (using plumbing)
BLOB=$(git hash-object -w feature-branch-file.txt)
git update-index --add --cacheinfo 100644 $BLOB feature-branch-file.txt
TREE=$(git write-tree)
COMMIT=$(echo "Work on feature" | git commit-tree $TREE -p HEAD)

# Update feature-branch ref
git update-ref refs/heads/feature-branch $COMMIT

# View branches
git log --all --oneline --graph --decorate
```

## Step 17: Understanding Merge (Conceptually)

A merge commit has multiple parents:

```bash
# Switch back to master
git symbolic-ref HEAD refs/heads/master

# Create merge commit manually (two parents)
TREE=$(git write-tree)
COMMIT=$(echo "Merge feature-branch into master" | \
  git commit-tree $TREE \
  -p $(git rev-parse master) \
  -p $(git rev-parse feature-branch))

# Update master
git update-ref refs/heads/master $COMMIT

# View commit
git cat-file -p $COMMIT
```

**Output:**
```
tree <tree-sha>
parent <master-sha>
parent <feature-branch-sha>
author ...
committer ...

Merge feature-branch into master
```

**Key Point:** Merge commits simply have multiple parent references.

## Step 18: Detached HEAD State

```bash
# Point HEAD directly to commit (not a branch)
echo $(git rev-parse HEAD~2) > .git/HEAD

# Check state
git status
cat .git/HEAD
```

**Output:** `HEAD detached at <commit-sha>`

### Return to Branch

```bash
# Point HEAD back to branch reference
echo "ref: refs/heads/master" > .git/HEAD

# Verify
git status
```

**Key Point:** Detached HEAD means HEAD contains a SHA directly, not `ref: ...`.

## Step 19: Cleanup and Best Practices

### Garbage Collection

```bash
# Manually trigger GC
git gc

# View packfiles created
ls -lh .git/objects/pack/
```

### Verify After GC

```bash
# All objects still accessible
git fsck --full

# Count objects
git count-objects -v
```

**Key Point:** GC packs loose objects efficiently but keeps all reachable data.

## Summary: Git's Data Model

### The Four Object Types

1. **Blob**: File content (no name, no metadata)
2. **Tree**: Directory listing (maps names to blobs/trees)
3. **Commit**: Points to tree + metadata + parent(s)
4. **Tag**: Points to commit with annotation (annotated tags only)

### The Structure

```
Working Directory
      ↓ (git add = hash-object + update-index)
   .git/index
      ↓ (git commit = write-tree + commit-tree)
  .git/objects/
      ↓ (refs point to commits)
.git/refs/heads/
      ↓ (HEAD points to ref)
  .git/HEAD
```

### Key Files and Directories

- `.git/objects/`: All blobs, trees, commits, tags
- `.git/refs/heads/`: Branch pointers
- `.git/refs/tags/`: Tag pointers  
- `.git/HEAD`: Current branch or commit
- `.git/index`: Staging area (binary)
- `.git/logs/`: Reflog entries

### Essential Plumbing Commands

| Command | Purpose |
|---------|---------|
| `git hash-object -w` | Create blob object |
| `git update-index` | Stage files in index |
| `git write-tree` | Create tree from index |
| `git commit-tree` | Create commit object |
| `git update-ref` | Create/update references |
| `git symbolic-ref` | Work with symbolic refs (like HEAD) |
| `git cat-file -t/-s/-p` | Inspect objects |
| `git ls-files --stage` | View index |
| `git ls-tree` | View tree contents |
| `git rev-parse` | Resolve references |
| `git fsck` | Verify repository |
| `git count-objects` | Object statistics |

## Practical Exercise

Try recreating a real repository workflow using only plumbing commands:

```bash
# 1. Initialize
mkdir my-project && cd my-project
git init

# 2. Create initial files
echo "# My Project" > README.md
echo "*.log" > .gitignore

# 3. Stage them (plumbing way)
git hash-object -w README.md .gitignore
git update-index --add README.md .gitignore

# 4. Create first commit
TREE1=$(git write-tree)
COMMIT1=$(echo "Initial commit" | git commit-tree $TREE1)
git update-ref refs/heads/main $COMMIT1
git symbolic-ref HEAD refs/heads/main

# 5. Make changes
echo "More content" >> README.md

# 6. Stage and commit changes
git update-index README.md
TREE2=$(git write-tree)
COMMIT2=$(echo "Update README" | git commit-tree $TREE2 -p $COMMIT1)
git update-ref refs/heads/main $COMMIT2

# 7. Verify
git log --oneline
```

## Conclusion

You've now built a Git repository from scratch using plumbing commands. You understand:

- **Blobs** store file content
- **Trees** store directory structure  
- **Commits** link to trees and form history
- **Branches** are simple pointers to commits
- **HEAD** points to the current branch
- **Index** stages changes for commits
- **Refs** are stored as text files with SHAs

This knowledge makes you fearless with Git operations and able to recover from almost any situation!

## Next Steps

1. Explore packfiles: `git verify-pack -v .git/objects/pack/*.idx`
2. Understand git gc: `man git-gc`
3. Study reflog expiration: `git config --get gc.reflogExpire`
4. Learn about alternate object stores and Git internals documentation
5. Read the Git Book chapter on "Git Internals": https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain
