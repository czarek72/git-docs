# Git Internals: Hands-On Tutorial with Plumbing Commands

## Introduction

This tutorial teaches Git internals by having you build a repository from scratch using low-level "plumbing" commands. Follow along step-by-step, executing each command to see how Git actually works under the hood.

By the end, you'll understand blobs, trees, commits, branches, and how they all connect to form Git's elegant data model.

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

The Git Book chapter on "Git Internals" provides additional depth: https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain

```
.git/
├── HEAD                    # Points to current branch
├── config                  # Repository configuration
├── description            # Repository description (for GitWeb)
├── index                  # Staging area (binary file)
├── hooks/                 # Client/server side hook scripts
├── info/                  # Global exclude patterns
├── objects/               # All content (commits, trees, blobs)
│   ├── [0-9a-f][0-9a-f]/  # Object database (first 2 chars of SHA-1)
│   ├── pack/              # Packfiles for efficient storage
│   └── info/              # Object database info
├── refs/                  # References (pointers to commits)
│   ├── heads/             # Local branches
│   ├── remotes/           # Remote branches
│   └── tags/              # Tags
└── logs/                  # Reflog - history of ref updates
    ├── HEAD               # HEAD movement history
    └── refs/              # Branch movement history
```

## Git Objects: The Foundation

Git is fundamentally a **content-addressable filesystem**. Everything is stored as objects identified by SHA-1 hashes.

### Four Types of Objects

#### 1. Blob (Binary Large Object)
Stores file content WITHOUT filename or metadata.

```bash
# Create a blob
echo "Hello Git" | git hash-object -w --stdin
# Output: e965047ad7c57865823c7d992b1d046ea66edf78

# View blob content
git cat-file -p e965047ad7c57865823c7d992b1d046ea66edf78
# Output: Hello Git
```

**Storage location:** `.git/objects/e9/65047ad7c57865823c7d992b1d046ea66edf78`

#### 2. Tree
Stores directory structure - maps filenames to blobs and subtrees.

```bash
# View a tree object
git cat-file -p master^{tree}
# Output:
# 100644 blob a906cb2a4a904a152...   README.md
# 100644 blob 8ab686eafeb1f44702...   main.py
# 040000 tree 99f1a6d12cb4b6f19d...   src
```

**Storage location:** `.git/objects/[first-2-chars]/[remaining-38-chars]`

#### 3. Commit
Points to a tree and contains metadata (author, message, timestamp, parent commits).

```bash
git cat-file -p HEAD
# Output:
# tree 99f1a6d12cb4b6f19d3881dd...
# parent 8ab686eafeb1f44702acb4f8...
# author John Doe <john@example.com> 1707648234 +0000
# committer John Doe <john@example.com> 1707648234 +0000
#
# Commit message here
```

**Storage location:** `.git/objects/[first-2-chars]/[remaining-38-chars]`

#### 4. Tag (Annotated)
Points to a commit with additional metadata.

```bash
git cat-file -p v1.0
# Output:
# object 9a0c65f2c...
# type commit
# tag v1.0
# tagger John Doe <john@example.com> 1707648234 +0000
#
# Version 1.0 release
```

## Git Plumbing Commands: A Practical Tutorial

Git provides "plumbing" commands (low-level) alongside "porcelain" commands (user-facing). These plumbing commands let you inspect and manipulate Git's internal data structures directly. Mastering these tools is essential for understanding and recovering from complex situations.

### `git cat-file`: The Universal Object Inspector

`cat-file` is your primary tool for examining Git objects. It can display object content, type, size, and more.

#### Basic Syntax
```bash
git cat-file <options> <object>
```

#### Essential Options

**`-t` - Show object type**
```bash
git cat-file -t HEAD
# Output: commit

git cat-file -t HEAD^{tree}
# Output: tree

git cat-file -t HEAD:README.md
# Output: blob
```

**`-s` - Show object size (in bytes)**
```bash
git cat-file -s HEAD
# Output: 234

git cat-file -s HEAD:large-file.bin
# Output: 1048576
```

**`-p` - Pretty-print object content**
```bash
# View commit
git cat-file -p HEAD
# Output:
# tree 99f1a6d12cb4b6f19d3881dd...
# parent 8ab686eafeb1f44702acb4f8...
# author John Doe <john@example.com> 1707648234 +0000
# committer John Doe <john@example.com> 1707648234 +0000
#
# Commit message

# View tree
git cat-file -p HEAD^{tree}
# Output:
# 100644 blob a906cb...   README.md
# 040000 tree 8ab686...   src
# 100755 blob 3f2d1e...   script.sh

# View blob (file content)
git cat-file -p HEAD:README.md
# Output: (actual file content)
```

**`-e` - Exit with status 0 if object exists**
```bash
git cat-file -e HEAD
echo $?  # 0 = exists

git cat-file -e nonexistent
echo $?  # 128 = doesn't exist
```

**`--batch` - Process multiple objects from stdin**
```bash
# Check multiple objects
echo -e "HEAD\nHEAD~1\nHEAD~2" | git cat-file --batch
# Output: (info for each object)

# Useful for scripting
git rev-list --all | git cat-file --batch-check
```

**`--batch-check` - Show info without content**
```bash
echo "HEAD" | git cat-file --batch-check
# Output: <sha1> commit <size>

echo "HEAD:README.md" | git cat-file --batch-check
# Output: <sha1> blob <size>
```

#### Practical Examples

**Example 1: Trace a file through commits**
```bash
# Find blob SHA for current version
git ls-tree HEAD src/main.py
# Output: 100644 blob 5d3f2a1... src/main.py

# View content
git cat-file -p 5d3f2a1

# Find blob SHA in previous commit
git ls-tree HEAD~1 src/main.py
# Output: 100644 blob 8e9a4b2... src/main.py

# Compare
git cat-file -p 8e9a4b2
```

**Example 2: Inspect unreachable objects**
```bash
# Find dangling blobs
git fsck --unreachable | grep blob

# Output: unreachable blob 3f2d1e4a...

# View content
git cat-file -p 3f2d1e4a
# Determine if it's worth recovering
```

**Example 3: Verify object integrity**
```bash
# Check if specific commit exists
git cat-file -e 9c1f3d2e && echo "Exists" || echo "Missing"

# Get object type and size
git cat-file -t 9c1f3d2e
git cat-file -s 9c1f3d2e
```

### `git ls-tree`: Inspect Tree Objects

Lists the contents of a tree object (directory structure).

#### Basic Usage
```bash
# List files in current commit
git ls-tree HEAD
# Output:
# 100644 blob 5d3f2a1...    README.md
# 040000 tree 8e9a4b2...    src
# 100755 blob 3f2d1e4...    script.sh

# Recursive listing (show all nested files)
git ls-tree -r HEAD
# Output:
# 100644 blob 5d3f2a1...    README.md
# 100755 blob 3f2d1e4...    script.sh
# 100644 blob 9a8b7c6...    src/main.py
# 100644 blob 2d1e0f3...    src/utils.py

# Show only specific subdirectory
git ls-tree HEAD src/
# 100644 blob 9a8b7c6...    src/main.py
# 100644 blob 2d1e0f3...    src/utils.py

# Include file sizes and names only
git ls-tree -r -l --name-only HEAD
```

#### Options
- `-r` - Recurse into subtrees
- `-l` - Show object size
- `-t` - Show tree entries (not just blobs)
- `--name-only` - Show only filenames
- `--name-status` - Show filenames with status
- `-z` - Null-terminate entries (for scripting)

#### Practical Examples

**Example 1: Find large files in history**
```bash
# List all files with sizes
git ls-tree -r -l HEAD | sort -k 4 -n -r | head -10
# Shows 10 largest files in HEAD

# Check historical commits
for commit in $(git rev-list HEAD); do
    git ls-tree -r -l $commit | awk '{if($4 > 10000000) print $3, $4, $5}'
done
```

**Example 2: Compare directory structures**
```bash
# Compare two commits
diff <(git ls-tree -r HEAD) <(git ls-tree -r HEAD~5)

# Find added files
git ls-tree -r --name-only HEAD > current.txt
git ls-tree -r --name-only HEAD~5 > previous.txt
diff previous.txt current.txt
```

**Example 3: Extract specific file version**
```bash
# Get blob SHA
BLOB=$(git ls-tree HEAD path/to/file.txt | awk '{print $3}')

# Extract content
git cat-file -p $BLOB > recovered-file.txt
```

### `git ls-files`: Inspect the Index (Staging Area)

Shows files in the index and working tree.

#### Basic Usage
```bash
# Show all tracked files
git ls-files
# Output: (list of tracked files)

# Show with stage information
git ls-files --stage
# Output:
# 100644 5d3f2a1... 0	README.md
# 100644 8e9a4b2... 0	src/main.py
# Format: <mode> <sha1> <stage> <filename>

# Show untracked files
git ls-files --others
# Shows files not in index

# Show ignored files
git ls-files --ignored --exclude-standard
```

#### Stage Numbers (Merge Conflicts)
- **Stage 0**: Normal file (no conflict)
- **Stage 1**: Common ancestor version
- **Stage 2**: Current branch version (HEAD)
- **Stage 3**: Merged branch version

```bash
# During merge conflict
git ls-files --stage
# 100644 1a2b3c4... 1	conflicted.txt  # ancestor
# 100644 5d6e7f8... 2	conflicted.txt  # ours
# 100644 9a0b1c2... 3	conflicted.txt  # theirs
# 100644 3d4e5f6... 0	normal.txt      # no conflict
```

#### Options
- `--stage` / `-s` - Show staged contents with SHA-1
- `--cached` - Show files in index
- `--deleted` - Show deleted files
- `--modified` - Show modified files
- `--others` / `-o` - Show untracked files
- `--ignored` - Show ignored files
- `--exclude-standard` - Use standard ignore rules
- `-z` - Null-terminate output

#### Practical Examples

**Example 1: Find files changed but not staged**
```bash
# Modified files
git ls-files --modified

# Deleted files
git ls-files --deleted
```

**Example 2: Recover file from index after accidental overwrite**
```bash
# File was staged, then accidentally overwritten
# Get SHA from index
SHA=$(git ls-files --stage myfile.txt | awk '{print $2}')

# Recover content
git cat-file -p $SHA > myfile.txt
```

**Example 3: Analyze merge conflict**
```bash
# Show all conflict versions
git ls-files --stage | grep conflicted.txt

# Extract each version
git cat-file -p $(git ls-files --stage conflicted.txt | awk 'NR==1{print $2}') > ancestor.txt
git cat-file -p $(git ls-files --stage conflicted.txt | awk 'NR==2{print $2}') > ours.txt
git cat-file -p $(git ls-files --stage conflicted.txt | awk 'NR==3{print $2}') > theirs.txt
```

### `git hash-object`: Create Git Objects

Creates blob objects from files or stdin, storing them in the object database.

#### Basic Usage
```bash
# Create blob from stdin (without writing to database)
echo "Hello Git" | git hash-object --stdin
# Output: e965047ad7c57865823c7d992b1d046ea66edf78

# Create and write to database
echo "Hello Git" | git hash-object -w --stdin
# Output: e965047ad7c57865823c7d992b1d046ea66edf78
# Creates: .git/objects/e9/65047ad7c57865823c7d992b1d046ea66edf78

# Hash existing file
git hash-object myfile.txt

# Hash and write file to database
git hash-object -w myfile.txt
```

#### Options
- `-w` - Write object to database (without this, just shows hash)
- `--stdin` - Read from standard input
- `--path <file>` - Hash object as if it were at path (affects .gitattributes)
- `--no-filters` - Don't apply filters

#### Practical Examples

**Example 1: Manually stage a file**
```bash
# Create blob
BLOB=$(git hash-object -w myfile.txt)

# Update index manually
git update-index --add --cacheinfo 100644 $BLOB myfile.txt

# Verify
git ls-files --stage myfile.txt
```

**Example 2: Test if file content exists in Git**
```bash
# Get hash without creating object
HASH=$(git hash-object myfile.txt)

# Check if object exists
git cat-file -e $HASH && echo "Already in Git" || echo "New content"
```

**Example 3: Find duplicate files**
```bash
# Hash all files
for file in $(find . -type f); do
    echo "$(git hash-object "$file") $file"
done | sort | uniq -w 40 -D
# Shows duplicate content
```

### `git rev-parse`: Parse Revision References

Resolves Git references to SHA-1 hashes and performs other parsing operations.

#### Basic Usage
```bash
# Get commit SHA
git rev-parse HEAD
# Output: 9c1f3d2e5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d

git rev-parse HEAD~3
# Output: (SHA of 3 commits ago)

git rev-parse feature-branch
# Output: (SHA of branch tip)

# Get tree SHA
git rev-parse HEAD^{tree}
# Output: (tree SHA)

# Get blob SHA
git rev-parse HEAD:README.md
# Output: (blob SHA)
```

#### Options
- `--short` - Abbreviate SHA (default: 7 characters)
- `--short=<n>` - Abbreviate to n characters
- `--verify` - Verify the reference exists
- `--git-dir` - Show .git directory path
- `--show-toplevel` - Show repository root
- `--abbrev-ref` - Show reference name instead of SHA

#### Practical Examples

**Example 1: Resolve complex references**
```bash
# Get 5th commit before HEAD
git rev-parse HEAD~5

# Get merge base of two branches
git rev-parse $(git merge-base main feature)

# Get parent commits
git rev-parse HEAD^1  # First parent
git rev-parse HEAD^2  # Second parent (merge commit)

# Get commit from tag
git rev-parse v1.0.0^{commit}
```

**Example 2: Short SHAs for display**
```bash
# 7-character short SHA
git rev-parse --short HEAD
# Output: 9c1f3d2

# Custom length
git rev-parse --short=12 HEAD
# Output: 9c1f3d2e5a6b
```

**Example 3: Get repository paths**
```bash
# Repository root
git rev-parse --show-toplevel
# Output: /home/user/project

# .git directory
git rev-parse --git-dir
# Output: /home/user/project/.git

# Current branch name
git rev-parse --abbrev-ref HEAD
# Output: main
```

### `git show`: Display Git Objects

A high-level command that combines functionality of `cat-file` and `diff`.

#### Basic Usage
```bash
# Show commit (diff + metadata)
git show HEAD

# Show specific file version
git show HEAD:README.md

# Show tree
git show HEAD^{tree}

# Show blob
git show 5d3f2a1e

# Show tag
git show v1.0.0
```

#### Options
- `--stat` - Show diffstat
- `--name-only` - Show only names of changed files
- `--name-status` - Show names and status (A/M/D)
- `--pretty=<format>` - Custom format
- `--no-patch` - Suppress diff output

#### Practical Examples

**Example 1: Quick object inspection**
```bash
# Show any object by SHA
git show 9c1f3d2e

# Show file from specific commit
git show HEAD~5:src/old-file.py

# Show what changed in commit
git show --stat HEAD
```

**Example 2: Extract specific file version**
```bash
# Save old version
git show HEAD~10:config.json > config-old.json

# Compare with current
diff config-old.json config.json
```

### `git verify-pack`: Inspect Packfiles

Packfiles store objects efficiently. This command lets you inspect them.

#### Basic Usage
```bash
# List all objects in packfile
git verify-pack -v .git/objects/pack/pack-*.idx

# Show statistics
git verify-pack -v .git/objects/pack/pack-*.idx | tail -n 3
# Shows count, size, and size-in-pack
```

#### Practical Examples

**Example 1: Find large objects in packfiles**
```bash
# List largest objects
git verify-pack -v .git/objects/pack/*.idx | 
  sort -k 3 -n -r | 
  head -20
# Shows 20 largest objects

# Get more info about large object
git cat-file -t <sha>
git cat-file -s <sha>
```

### `git count-objects`: Repository Statistics

Shows count and disk usage of unpacked objects.

#### Basic Usage
```bash
# Basic count
git count-objects
# Output: 123 objects, 456 kilobytes

# Verbose output
git count-objects -v
# Output:
# count: 123
# size: 456
# in-pack: 5000
# packs: 2
# size-pack: 12345
# prune-packable: 0
# garbage: 0
# size-garbage: 0
```

#### Options
- `-v` / `--verbose` - Show detailed statistics
- `-H` / `--human-readable` - Show sizes in human format

### `git fsck`: File System Check

Verifies connectivity and validity of objects in the database.

#### Basic Usage
```bash
# Check repository integrity
git fsck

# Find unreachable objects
git fsck --unreachable

# Find dangling objects (unreachable by any ref)
git fsck --lost-found
# Creates: .git/lost-found/commit/ and .git/lost-found/other/

# Full verification
git fsck --full --strict
```

#### Options
- `--unreachable` - Show unreachable objects
- `--lost-found` - Write dangling objects to .git/lost-found/
- `--full` - Check all object directories
- `--strict` - Enable stricter checking
- `--no-reflogs` - Don't consider reflog entries

#### Practical Examples

**Example 1: Find recoverable data**
```bash
# Find all unreachable objects
git fsck --unreachable | grep -E 'commit|blob|tree'

# Find just commits
git fsck --unreachable | grep commit
# Output: unreachable commit 9c1f3d2e...

# Inspect found commit
git show 9c1f3d2e
```

**Example 2: Recover lost commits**
```bash
# Find all dangling commits
git fsck --lost-found | grep commit

# View each one
for commit in $(git fsck --unreachable --no-reflogs | grep commit | awk '{print $3}'); do
    echo "=== $commit ==="
    git show --stat $commit
done
```

### Command Combination Workflows

#### Workflow 1: Complete Object Archaeology
```bash
# 1. Find object
git fsck --unreachable | grep blob
# Output: unreachable blob 3f2d1e4a...

# 2. Determine type and size
git cat-file -t 3f2d1e4a
git cat-file -s 3f2d1e4a

# 3. View content
git cat-file -p 3f2d1e4a

# 4. Save if needed
git cat-file -p 3f2d1e4a > recovered-file.txt
```

#### Workflow 2: Trace File History Through Blobs
```bash
# 1. Get current blob
git ls-tree HEAD path/to/file.txt
# Output: 100644 blob 5d3f2a1... path/to/file.txt

# 2. Find all commits touching this file
git log --all --pretty=format:"%H" -- path/to/file.txt

# 3. For each commit, get blob SHA
for commit in $(git log --all --pretty=format:"%H" -- path/to/file.txt); do
    echo "=== $commit ==="
    git ls-tree $commit path/to/file.txt
done

# 4. Compare specific versions
git cat-file -p 5d3f2a1 > version1.txt
git cat-file -p 8e9a4b2 > version2.txt
diff version1.txt version2.txt
```

#### Workflow 3: Diagnose Repository Bloat
```bash
# 1. Overall statistics
git count-objects -vH

# 2. Find large packfiles
ls -lh .git/objects/pack/

# 3. List largest objects
git verify-pack -v .git/objects/pack/*.idx | 
  sort -k 3 -n -r | 
  head -20

# 4. Identify large objects
for sha in $(git verify-pack -v .git/objects/pack/*.idx | sort -k 3 -n -r | head -10 | awk '{print $1}'); do
    echo "=== $sha ==="
    git cat-file -t $sha
    git cat-file -s $sha
    git rev-list --all --objects | grep $sha
done
```

#### Workflow 4: Manual Cherry-Pick Using Plumbing
```bash
# 1. Get commit details
git cat-file -p <commit-sha>
# Note the tree SHA

# 2. Get tree content
git ls-tree <tree-sha>

# 3. Extract specific file blob
BLOB=$(git ls-tree <tree-sha> path/to/file | awk '{print $3}')

# 4. Stage it manually
git hash-object -w <file>  # If you have the file
# OR
git cat-file -p $BLOB > temp-file
git add temp-file

# 5. Commit
git commit -m "Manual cherry-pick"
```

### Quick Reference: Command Selection Guide

| Task | Command |
|------|---------|
| View any object content | `git cat-file -p <sha>` |
| Check object type | `git cat-file -t <sha>` |
| Check object size | `git cat-file -s <sha>` |
| List directory contents | `git ls-tree HEAD` |
| Show all files recursively | `git ls-tree -r HEAD` |
| Show staged files | `git ls-files --stage` |
| Show merge conflicts | `git ls-files --stage \| grep "^[123]"` |
| Convert ref to SHA | `git rev-parse <ref>` |
| Get short SHA | `git rev-parse --short HEAD` |
| Find repository root | `git rev-parse --show-toplevel` |
| Create blob object | `git hash-object -w <file>` |
| Display object with diff | `git show <sha>` |
| Extract file version | `git show <sha>:path > file` |
| Check repository | `git fsck --full` |
| Find lost objects | `git fsck --unreachable` |
| Repository statistics | `git count-objects -vH` |
| Inspect packfiles | `git verify-pack -v .git/objects/pack/*.idx` |

### Pro Tips

1. **Use tab completion** - Most shells can complete Git SHA-1 hashes
2. **Pipe to less** - `git cat-file -p <sha> | less` for large objects
3. **Combine with grep** - `git cat-file -p HEAD | grep "pattern"`
4. **Script with xargs** - `git rev-list --all | xargs -n 1 git cat-file -t`
5. **Use aliases** - Create shortcuts for common operations:
```bash
git config alias.type 'cat-file -t'
git config alias.dump 'cat-file -p'
git config alias.size 'cat-file -s'
```

With these plumbing commands, you can dissect and understand every aspect of Git's internal structure, making you nearly unstoppable when it comes to recovery and debugging operations.

## What Is a Branch REALLY?

A branch is simply a **41-byte file** containing a SHA-1 hash (40 characters) plus a newline.

### File Location
```bash
# The master branch is stored here:
cat .git/refs/heads/master
# Output: 8ab686eafeb1f44702acb4f8d8e2e5a5f7e5b3c1
```

### Creating a Branch
```bash
git branch feature-x
# This creates: .git/refs/heads/feature-x
# Containing the same commit hash as your current branch
```

**That's it!** A branch is just a lightweight movable pointer to a commit.

### Moving a Branch
When you commit on a branch:
1. Git creates new commit object
2. Updates the branch file with new commit's SHA-1
3. Updates reflog

```bash
# Before commit
cat .git/refs/heads/master
# 8ab686eafeb1f44702acb4f8d8e2e5a5f7e5b3c1

# After commit
cat .git/refs/heads/master
# 9c1f3d2e5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d
```

## HEAD: Where You Are

HEAD is a **symbolic reference** pointing to your current branch.

```bash
cat .git/HEAD
# Output: ref: refs/heads/master
```

### Detached HEAD State
When HEAD points directly to a commit (not a branch):

```bash
git checkout 8ab686ea

cat .git/HEAD
# Output: 8ab686eafeb1f44702acb4f8d8e2e5a5f7e5b3c1
# (Direct commit hash, not a ref)
```

## The Index (Staging Area)

The index is a **binary file** at `.git/index` that contains:
- List of files to include in next commit
- Their SHA-1 hashes
- Metadata (timestamps, permissions, file sizes)

### How It Works

```bash
# Working directory → Index (staging)
echo "content" > file.txt
git add file.txt
# This:
# 1. Creates blob object in .git/objects/
# 2. Updates .git/index with blob SHA-1 and metadata

# View index content
git ls-files --stage
# Output: 100644 d95f3ad14dee633a758d2e331151e950dd13e4ed 0	file.txt
```

### Index vs Working Directory vs Repository

```
Working Directory  →  Index (Staging)  →  Repository (.git/objects)
     (files)           (.git/index)         (commits, trees, blobs)
                            ↓
                       git add file
                            ↓
                       git commit
```

## Reflog: The Safety Net

Reflog records **every update to branch tips and HEAD**, even if commits become "unreachable."

### Storage Location
```bash
# HEAD reflog
cat .git/logs/HEAD

# Branch reflog
cat .git/logs/refs/heads/master
```

### Reflog Format
```
<old-sha1> <new-sha1> <author> <timestamp> <action>: <message>

Example:
8ab686ea 9c1f3d2e John Doe <john@example.com> 1707648234 +0000 commit: Add feature
9c1f3d2e 5d4e3f2a John Doe <john@example.com> 1707648345 +0000 reset: moving to HEAD~1
```

### Viewing Reflog
```bash
git reflog
# Output:
# 5d4e3f2 HEAD@{0}: reset: moving to HEAD~1
# 9c1f3d2 HEAD@{1}: commit: Add feature
# 8ab686e HEAD@{2}: commit: Initial commit

git reflog show master
# Shows reflog for specific branch
```

## When Is Data REALLY Deleted?

### Data Is Recoverable In These Scenarios

#### 1. After `git reset --hard`
```bash
git reset --hard HEAD~3
# Commits appear lost, but are still in objects/

# Recover using reflog
git reflog
# Find lost commit SHA-1
git reset --hard <lost-commit-sha>

# Or create new branch pointing to lost commit
git branch recovery-branch 9c1f3d2e
```

**Why recoverable?** 
- Commit objects still exist in `.git/objects/`
- Reflog still references them

#### 2. After Deleting a Branch
```bash
git branch -D feature-branch
# Branch pointer deleted, but commits remain

# Find commit
git reflog
# or
git fsck --lost-found

# Recover
git branch feature-branch <commit-sha>
```

**Why recoverable?**
- Only the branch file (`.git/refs/heads/feature-branch`) is deleted
- Commit objects remain untouched

#### 3. After Amending or Rebasing
```bash
git commit --amend
# Old commit replaced, but still in objects/

git rebase -i HEAD~5
# Rewritten commits still exist

# Recover original commits
git reflog
git reset --hard HEAD@{1}
```

**Why recoverable?**
- New commits are created
- Old commits remain in object database
- Reflog tracks the changes

#### 4. Uncommitted Changes in Index
```bash
git add file.txt
git reset --hard
# file.txt changes lost from working directory
# BUT blob object still exists!

# Find blob
git fsck --lost-found
# Objects are in .git/lost-found/other/

# View blob content
git show <blob-sha>
```

### Data Is TRULY Lost In These Scenarios

#### 1. After Garbage Collection
Git periodically runs garbage collection (`git gc`), which:
- Removes objects unreachable for **90 days** (default)
- Prunes reflog entries older than **90 days** (default)

```bash
# Manual garbage collection
git gc --prune=now
# Immediately removes unreachable objects

# After this, unreferenced commits are GONE
```

**Configuration:**
```bash
# Check GC settings
git config gc.reflogExpire         # Default: 90 days
git config gc.reflogExpireUnreachable  # Default: 30 days
git config gc.pruneExpire          # Default: 2 weeks ago
```

#### 2. Changes Never Committed or Staged
```bash
echo "important data" > file.txt
# NOT git add or git commit
rm file.txt
# Data is GONE - never entered Git's object database
```

**Why truly lost?**
- Git only tracks what you `git add` or `git commit`
- Working directory changes are not in `.git/objects/`

#### 3. After Reflog Expires
```bash
# Reflog entries expire after 90 days (default)
# Once expired and GC runs, commits become unreachable

git reflog expire --expire=now --all
git gc --prune=now
# All unreferenced commits are now DELETED
```

#### 4. Stash Dropped and GC'd
```bash
git stash
git stash drop
# Wait 90 days or force GC
git gc --prune=now
# Stash is GONE
```

## Practical Examples: Recovery Operations

### Example 1: Recover Deleted Branch

```bash
# Create and delete branch
git branch important-feature
git checkout important-feature
echo "feature" > feature.txt
git add feature.txt
git commit -m "Add feature"
git checkout master
git branch -D important-feature  # Oops!

# Recovery
git reflog  # Find commit SHA
# Output: 7f3e2a1 HEAD@{1}: commit: Add feature

git branch important-feature 7f3e2a1  # Recovered!
```

### Example 2: Recover After Hard Reset

```bash
# Make commits
git commit -m "Commit 1"
git commit -m "Commit 2"
git commit -m "Commit 3"

# Accidentally reset
git reset --hard HEAD~3  # Lost all 3 commits!

# Recovery
git reflog
# Output:
# abc1234 HEAD@{1}: commit: Commit 3
# def5678 HEAD@{2}: commit: Commit 2
# ghi9012 HEAD@{3}: commit: Commit 1

git reset --hard abc1234  # Back to Commit 3
```

### Example 3: Recover Staged but Uncommitted Files

```bash
# Stage file
echo "important work" > important.txt
git add important.txt

# Accidentally hard reset
git reset --hard

# File gone from working directory, BUT...
# Find dangling blob
git fsck --lost-found
# Output: dangling blob 3a2f1e...

# View blob
git show 3a2f1e
# Output: important work

# Restore to file
git show 3a2f1e > important.txt
```

### Example 4: Find Lost Commits

```bash
# Find all unreachable commits
git fsck --unreachable --no-reflogs | grep commit

# Output:
# unreachable commit 5d4e3f2a4b3c2d1e0f1a2b3c4d5e6f7a8b9c0d1e
# unreachable commit 9c1f3d2e5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d

# View commit
git show 5d4e3f2a
# Create branch to recover
git branch recovered 5d4e3f2a
```

## Understanding Undo Commands Internally

### `git reset --soft HEAD~1`
1. Moves branch pointer back one commit
2. Updates `.git/refs/heads/<branch>` file
3. Leaves index and working directory unchanged
4. Records move in reflog

### `git reset --mixed HEAD~1` (default)
1. Moves branch pointer back one commit
2. Updates `.git/refs/heads/<branch>` file
3. Updates `.git/index` to match new HEAD
4. Leaves working directory unchanged
5. Records move in reflog

### `git reset --hard HEAD~1`
1. Moves branch pointer back one commit
2. Updates `.git/refs/heads/<branch>` file
3. Updates `.git/index` to match new HEAD
4. Updates working directory to match new HEAD
5. Records move in reflog
6. **Original commit still exists in `.git/objects/`**

### `git revert <commit>`
1. Creates NEW commit object
2. Commit's tree is inverse of reverted commit
3. Does NOT delete original commit
4. Updates branch pointer
5. Records in reflog

### `git checkout <commit>`
1. Updates `.git/HEAD` to commit SHA (detached) or branch ref
2. Updates `.git/index` to match commit's tree
3. Updates working directory
4. Records in HEAD's reflog

## Visual: The Life of a Commit

```
1. You make changes
   └─ Files in working directory (NOT in Git yet)

2. git add
   ├─ Creates blob objects in .git/objects/
   └─ Updates .git/index with blob references

3. git commit
   ├─ Creates tree object (from index) in .git/objects/
   ├─ Creates commit object in .git/objects/
   ├─ Updates .git/refs/heads/<branch> with new commit SHA
   └─ Records in .git/logs/refs/heads/<branch> (reflog)

4. git reset --hard HEAD~1
   ├─ Updates .git/refs/heads/<branch> to previous commit
   ├─ Updates .git/index and working directory
   └─ Records in reflog
   └─ Commit object STILL EXISTS in .git/objects/

5. After 90 days + git gc
   ├─ Reflog entry expires
   ├─ Commit becomes unreachable
   └─ git gc DELETES commit object from .git/objects/
       └─ NOW it's truly gone
```

## Key Takeaways

### Data Persistence Rules

1. **Once committed, data is safe** (for at least 90 days)
   - Commits create permanent objects in `.git/objects/`
   - Even "deleted" commits remain until GC

2. **Staged data is partially safe**
   - Blobs created in `.git/objects/`
   - Can be recovered with `git fsck`
   - No easy reference like reflog

3. **Unstaged data is unsafe**
   - Not in Git's object database
   - Standard file recovery only

4. **Reflog is your safety net**
   - Tracks all ref updates for 90 days
   - Use `git reflog` to find lost commits
   - Prevent expiration: `git config gc.reflogExpire never`

5. **Garbage collection is final**
   - After GC, unreachable objects are deleted
   - Delay with: `git config gc.pruneExpire "1 year ago"`
   - Disable auto-GC: `git config gc.auto 0`

### Recovery Strategy

```bash
# 1. Check reflog first
git reflog
git reflog show <branch-name>

# 2. If reflog doesn't help, find unreachable objects
git fsck --unreachable --no-reflogs

# 3. View found objects
git show <sha1>

# 4. Recover commits
git branch recovery-branch <commit-sha>

# 5. Recover blobs/trees
git show <object-sha> > recovered-file.txt

# 6. For extremely old data (beyond reflog)
git fsck --lost-found
# Check .git/lost-found/commit/ and .git/lost-found/other/
```

## Advanced: Preventing Data Loss

### Extend Reflog Retention
```bash
# Keep reflog forever
git config gc.reflogExpire never
git config gc.reflogExpireUnreachable never

# Or extend to 1 year
git config gc.reflogExpire "1 year"
git config gc.reflogExpireUnreachable "1 year"
```

### Disable or Control Garbage Collection
```bash
# Disable automatic GC
git config gc.auto 0

# Extend prune period
git config gc.pruneExpire "1 year ago"

# Manual GC with extreme caution
git gc --prune=1.year.ago
```

### Create Safety Branches
```bash
# Before risky operations
git branch backup-$(date +%Y%m%d-%H%M%S)

# Or tag current state
git tag backup-before-rebase
```

### Use Git Hooks
```bash
# .git/hooks/pre-rebase
#!/bin/bash
git branch backup-pre-rebase-$(date +%Y%m%d-%H%M%S)
```

## Conclusion

Git's internal structure is elegantly simple:
- **Objects** (blobs, trees, commits) store all content
- **Refs** (branches, tags) are pointers to commits
- **Index** stages changes for next commit
- **Reflog** tracks ref movements

Understanding this structure reveals that:
- **Branches are cheap** (41-byte files)
- **Commits are permanent** (until GC)
- **History is safe** (reflog protects you)
- **True deletion requires GC** (90-day default grace period)

With this knowledge, you can confidently experiment with Git, knowing exactly when data is at risk and how to recover it.

## Quick Reference Card

| Scenario | Recoverable? | How Long? | Recovery Method |
|----------|--------------|-----------|-----------------|
| After `git reset --hard` | ✅ Yes | ~90 days | `git reflog` + `git reset` |
| Deleted branch | ✅ Yes | ~90 days | `git reflog` + `git branch` |
| Amended commit | ✅ Yes | ~90 days | `git reflog` |
| Rebased commits | ✅ Yes | ~90 days | `git reflog` |
| Staged then reset | ✅ Yes | ~30 days | `git fsck --lost-found` |
| Dropped stash | ✅ Yes | ~90 days | `git fsck --unreachable` |
| After `git gc --prune=now` | ❌ No | Gone forever | None |
| Never committed/staged | ❌ No | N/A | File system recovery only |
| Reflog expired + GC | ❌ No | Gone forever | None |

Remember: **Git protects committed data aggressively. The main danger is garbage collection after the reflog expires.**
