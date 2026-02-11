# Git Index Internals: A Complete Guide

## Table of Contents
1. [What is the Git Index?](#what-is-the-git-index)
2. [Index Internal Structure](#index-internal-structure)
3. [Inspecting the Index](#inspecting-the-index)
4. [Adding Files to the Index](#adding-files-to-the-index)
5. [Removing Files from the Index](#removing-files-from-the-index)
6. [Index States and File Tracking](#index-states-and-file-tracking)
7. [Merge Conflicts and the Index](#merge-conflicts-and-the-index)
8. [Advanced Index Operations](#advanced-index-operations)
9. [Index vs Working Directory vs Repository](#index-vs-working-directory-vs-repository)
10. [Practical Scenarios](#practical-scenarios)

---

## What is the Git Index?

The **Git index** (also called the **staging area** or **cache**) is a critical intermediate layer between your working directory and the Git repository. It's a binary file located at `.git/index` that acts as a proposed next commit.

### Key Concepts
- **Purpose**: The index stores a snapshot of files that will be included in the next commit
- **Location**: `.git/index` (binary file)
- **Role**: Staging area between working directory and repository
- **Nature**: Contains file metadata and references to blob objects

### The Three States of Files

```
Working Directory  →  Index (Staging)  →  Repository (Committed)
     (modified)          (staged)            (committed)
```

---

## Understanding File Locations: Working Directory vs Index vs Repository

Before diving into the index structure, it's crucial to understand **where** files can exist in Git's system and what each location means.

### The Three Locations

Git manages files in three distinct locations:

```
┌─────────────────────────────────────────────────────────┐
│ 1. WORKING DIRECTORY (Your Filesystem)                 │
│    - Files you can see and edit                        │
│    - May or may not be tracked by Git                  │
│    - Location: Your project folder                     │
└─────────────────────────────────────────────────────────┘
                            ↓ git add
┌─────────────────────────────────────────────────────────┐
│ 2. INDEX / STAGING AREA (.git/index)                   │
│    - Snapshot of what will be in next commit           │
│    - Binary file with metadata + object references     │
│    - Location: .git/index                              │
└─────────────────────────────────────────────────────────┘
                            ↓ git commit
┌─────────────────────────────────────────────────────────┐
│ 3. REPOSITORY (.git/objects/)                          │
│    - Permanent commit history                          │
│    - Immutable snapshots (commits, trees, blobs)       │
│    - Location: .git/objects/ + .git/refs/              │
└─────────────────────────────────────────────────────────┘
```

### File State Combinations

A file can exist in one, two, or all three locations simultaneously. Here's what each combination means:

#### State 1: Only in Working Directory (Untracked)
**File exists: Working Directory ONLY**

```bash
# Create a new file
echo "Hello" > newfile.txt

# Status: UNTRACKED
git status
# Untracked files:
#   newfile.txt

# Check: Not in index
git ls-files | grep newfile.txt
# (no output - file not in index)

# Check: Not in repository
git log --all --full-history -- newfile.txt
# (no output - file never committed)
```

**What this means:**
- File is completely invisible to Git's history
- Won't be included in commits
- Won't be tracked by version control
- Can be deleted without `git rm`

---

#### State 2: In Working Directory + Index (Staged but Uncommitted)
**File exists: Working Directory + Index**

```bash
# Add the file to staging
git add newfile.txt

# Status: STAGED (ready to commit)
git status
# Changes to be committed:
#   new file:   newfile.txt

# Check: Now in index
git ls-files --stage | grep newfile.txt
# 100644 e965047ad7c57865823c7d992b1d046ea66edf78 0	newfile.txt

# Check: Still not in repository
git log --all --full-history -- newfile.txt
# (no output - not yet committed)
```

**What this means:**
- File is staged and ready for commit
- Git has created a blob object for the content
- If you delete the file from working directory, you can recover it from the index
- Not yet part of permanent history

---

#### State 3: In All Three Places (Tracked and Committed)
**File exists: Working Directory + Index + Repository**

```bash
# Commit the file
git commit -m "Add newfile.txt"

# Status: CLEAN (all three match)
git status
# nothing to commit, working tree clean

# Check: In index
git ls-files --stage | grep newfile.txt
# 100644 e965047ad7c57865823c7d992b1d046ea66edf78 0	newfile.txt

# Check: In repository
git log --oneline -- newfile.txt
# abc1234 Add newfile.txt

# All three locations have the SAME content
```

**What this means:**
- File is fully tracked
- Changes are preserved in Git history
- Can be recovered from any commit
- Working directory, index, and HEAD all have the same version

---

#### State 4: Modified in Working Directory (Uncommitted Changes)
**File exists: All three, but Working Directory is DIFFERENT**

```bash
# Modify the file
echo "More content" >> newfile.txt

# Status: MODIFIED (not staged)
git status
# Changes not staged for commit:
#   modified:   newfile.txt

# Working directory ≠ Index
git diff newfile.txt
# (shows changes)

# Index = Repository (no staged changes)
git diff --cached newfile.txt
# (no output)
```

**What this means:**
- File exists in all three places
- Working directory has newer changes
- Index and repository still have old version
- Changes not yet staged

---

#### State 5: Modified and Staged (Ready to Commit)
**File exists: All three, Index is DIFFERENT**

```bash
# Stage the changes
git add newfile.txt

# Status: STAGED (ready to commit)
git status
# Changes to be committed:
#   modified:   newfile.txt

# Working directory = Index (both have new changes)
git diff newfile.txt
# (no output)

# Index ≠ Repository (staged changes)
git diff --cached newfile.txt
# (shows changes)
```

**What this means:**
- Working directory and index match (both have new content)
- Repository still has old version
- Next commit will update repository to match index

---

#### State 6: Modified in Both (Staged + Additional Changes)
**File exists: All three, ALL DIFFERENT**

```bash
# Stage changes
echo "Change 1" >> file.txt
git add file.txt

# Make more changes
echo "Change 2" >> file.txt

# Status: Both staged and unstaged changes
git status
# Changes to be committed:
#   modified:   file.txt
# Changes not staged for commit:
#   modified:   file.txt

# Working ≠ Index
git diff file.txt
# (shows "Change 2")

# Index ≠ Repository
git diff --cached file.txt
# (shows "Change 1")
```

**What this means:**
- Three different versions exist simultaneously
- Repository: original version
- Index: version with "Change 1"
- Working Directory: version with "Change 1" + "Change 2"

---

#### State 7: Deleted from Working Directory (Not Staged)
**File exists: Index + Repository ONLY**

```bash
# Delete file from working directory
rm newfile.txt

# Status: DELETED (not staged)
git status
# Changes not staged for commit:
#   deleted:    newfile.txt

# File still in index
git ls-files | grep newfile.txt
# newfile.txt

# File still in repository
git show HEAD:newfile.txt
# (shows content)

# RECOVER: Restore from index
git restore newfile.txt
# File restored!
```

**What this means:**
- File physically deleted from disk
- Still tracked in index and repository
- Easy to recover with `git restore`

---

#### State 8: Deleted and Staged (Removal Ready to Commit)
**File exists: Repository ONLY**

```bash
# Remove file with git rm
git rm newfile.txt

# Status: Deletion staged
git status
# Changes to be committed:
#   deleted:    newfile.txt

# File not in working directory
ls newfile.txt
# ls: cannot access 'newfile.txt': No such file or directory

# File not in index
git ls-files | grep newfile.txt
# (no output)

# File still in repository
git show HEAD:newfile.txt
# (shows content)

# RECOVER: Restore before commit
git restore --staged newfile.txt
git restore newfile.txt
```

**What this means:**
- File deleted from working directory and index
- Still exists in repository (current HEAD)
- Next commit will remove from repository too

---

### Quick Reference: Checking File Locations

```bash
# Is file in WORKING DIRECTORY?
ls filename.txt
# or
test -f filename.txt && echo "EXISTS" || echo "NOT FOUND"

# Is file in INDEX?
git ls-files | grep filename.txt
# or
git ls-files --error-unmatch filename.txt 2>/dev/null && echo "IN INDEX" || echo "NOT IN INDEX"

# Is file in REPOSITORY (current HEAD)?
git cat-file -e HEAD:filename.txt 2>/dev/null && echo "IN REPO" || echo "NOT IN REPO"
# or
git show HEAD:filename.txt >/dev/null 2>&1 && echo "IN REPO" || echo "NOT IN REPO"

# Full status check
check_file() {
    local file=$1
    echo "=== Checking: $file ==="
    
    # Working directory
    if [ -f "$file" ]; then
        echo "✓ Working Directory: EXISTS"
    else
        echo "✗ Working Directory: NOT FOUND"
    fi
    
    # Index
    if git ls-files --error-unmatch "$file" >/dev/null 2>&1; then
        echo "✓ Index: TRACKED"
        git ls-files --stage "$file"
    else
        echo "✗ Index: NOT TRACKED"
    fi
    
    # Repository
    if git cat-file -e HEAD:"$file" 2>/dev/null; then
        echo "✓ Repository: COMMITTED"
    else
        echo "✗ Repository: NOT COMMITTED"
    fi
}

# Usage
check_file myfile.txt
```

### Why This Matters

Understanding these locations is critical because:

1. **Recovery**: You can only recover files that exist in the index or repository
2. **Git commands target different locations**:
   - `git add` → Working Directory → Index
   - `git commit` → Index → Repository
   - `git restore` → Index → Working Directory (or HEAD → both)
   - `git rm` → removes from Index + Working Directory
3. **Different versions can coexist**: You can have three different versions of the same file
4. **Safety**: Once committed, data is safe. Before that, it can be lost

---

## Index Internal Structure

### Binary Format

The `.git/index` file is a binary file with the following structure:

```
Header (12 bytes):
- 4 bytes: Signature "DIRC" (DIRectory Cache)
- 4 bytes: Version number (2, 3, or 4)
- 4 bytes: Number of entries

Index Entries (variable length each):
- ctime/mtime (file change time/modification time)
- dev/ino (device/inode number)
- mode (file permissions)
- uid/gid (user/group ID)
- file size
- SHA-1 hash (40 hex chars → 20 bytes)
- flags (including name length)
- file name
- padding

Extensions (optional):
- Tree cache
- Resolve undo
- Untracked cache
```

### What Gets Stored

For each file, the index stores:
1. **Metadata**: timestamps, permissions, size, inode
2. **Content Hash**: SHA-1 of the file content (blob object)
3. **Stage Number**: 0 for normal files, 1-3 for conflicts
4. **Path**: File path relative to repository root

### Viewing Raw Index Data

```bash
# View index in human-readable format
git ls-files --stage

# Example output:
# 100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0	file1.txt
# 100644 d8649f72e6a648f31fbd3c8d2e5ff5d8baa4c7a3 0	src/main.py

# Format: <mode> <object-SHA> <stage> <filename>
```

---

## Inspecting the Index

### `git ls-files`: The Index Inspector

The `git ls-files` command is your primary tool for inspecting the index.

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

#### Understanding File Modes

The mode field indicates file type and permissions:

```
100644 - Regular file (read/write)
100755 - Executable file
120000 - Symbolic link
040000 - Directory (in tree objects)
160000 - Gitlink (submodule)
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

#### Common Options

- `--stage` / `-s` - Show staged contents with SHA-1
- `--cached` - Show files in index
- `--deleted` - Show deleted files
- `--modified` - Show modified files
- `--others` / `-o` - Show untracked files
- `--ignored` - Show ignored files
- `--exclude-standard` - Use standard ignore rules
- `-z` - Null-terminate output (for scripting)

#### Practical Inspection Examples

**Example 1: Find files changed but not staged**
```bash
# Modified files
git ls-files --modified

# Deleted files
git ls-files --deleted
```

**Example 2: List all file states**
```bash
# Tracked files
echo "=== TRACKED ==="
git ls-files

# Modified but not staged
echo "=== MODIFIED ==="
git ls-files --modified

# Untracked files
echo "=== UNTRACKED ==="
git ls-files --others --exclude-standard

# Ignored files
echo "=== IGNORED ==="
git ls-files --ignored --exclude-standard
```

**Example 3: Recover file from index after accidental overwrite**
```bash
# File was staged, then accidentally overwritten
# Get SHA from index
SHA=$(git ls-files --stage myfile.txt | awk '{print $2}')

# Recover content
git cat-file -p $SHA > myfile.txt
```

---

## Adding Files to the Index

### What Happens During `git add`

When you run `git add`, Git performs these steps:

1. **Hash the content**: Creates SHA-1 hash of file content
2. **Create blob object**: Stores content in `.git/objects/`
3. **Update index**: Adds/updates entry in `.git/index`
4. **Store metadata**: Records file permissions, size, timestamps

```bash
# Adding files
git add file.txt

# What happens internally:
# 1. Read file.txt content
# 2. Create blob: git hash-object -w file.txt → SHA
# 3. Update index with SHA and metadata
```

### Scenarios of Adding Files

#### Scenario 1: Adding a New File

```bash
# Create new file
echo "Hello World" > newfile.txt

# Check status - file is untracked
git status
# Untracked files:
#   newfile.txt

# File not in index yet
git ls-files | grep newfile.txt
# (no output)

# Add to index
git add newfile.txt

# Now in index with stage 0
git ls-files --stage | grep newfile.txt
# 100644 557db03de997c86a4a028e1ebd3a1ceb225be238 0	newfile.txt

# Check object was created
git cat-file -t 557db03de997c86a4a028e1ebd3a1ceb225be238
# blob

git cat-file -p 557db03de997c86a4a028e1ebd3a1ceb225be238
# Hello World
```

#### Scenario 2: Modifying a Tracked File

```bash
# File is already tracked and committed
git ls-files --stage | grep tracked.txt
# 100644 abc123... 0	tracked.txt

# Modify the file
echo "New content" >> tracked.txt

# File is modified but not staged
git status
# Changes not staged for commit:
#   modified:   tracked.txt

# Index still has old version
OLD_SHA=$(git ls-files --stage tracked.txt | awk '{print $2}')
echo $OLD_SHA
# abc123...

# Stage the changes
git add tracked.txt

# Index now has new version
NEW_SHA=$(git ls-files --stage tracked.txt | awk '{print $2}')
echo $NEW_SHA
# def456... (different from old SHA)

# New blob object created
git cat-file -p $NEW_SHA
# (shows new content)
```

#### Scenario 3: Partial Staging (Interactive Add)

```bash
# File has multiple changes
cat file.txt
# Line 1
# Line 2
# Line 3

# Add interactively (patch mode)
git add -p file.txt

# You can stage specific hunks
# Index will contain partial changes
```

#### Scenario 4: Adding with Intent-to-Add

```bash
# Create new file but don't stage content yet
touch newfile.txt

# Mark as tracked without staging content
git add -N newfile.txt
# or
git add --intent-to-add newfile.txt

# File appears in git diff now
git diff
# Shows the file as new

# But not fully staged
git ls-files --stage | grep newfile.txt
# 100644 0000000000000000000000000000000000000000 0	newfile.txt
# Notice: all-zero SHA means content not staged
```

### Adding Multiple Files

```bash
# Add all files in directory
git add .

# Add all files matching pattern
git add '*.txt'

# Add all modified and deleted files (not untracked)
git add -u

# Add all files (modified, deleted, untracked)
git add -A
# or
git add --all
```

### What Does NOT Get Added

```bash
# Ignored files (.gitignore)
echo "*.log" >> .gitignore
touch debug.log
git add debug.log
# (ignored, won't be added)

# Force add ignored file
git add -f debug.log
# Now it's added despite being ignored
```

---

## Removing Files from the Index

### Unstaging Files (`git restore`, `git reset`)

#### Scenario 1: Unstage a File (Keep Working Copy)

```bash
# File is staged
git add file.txt
git ls-files --stage | grep file.txt
# 100644 abc123... 0	file.txt

# Unstage (remove from index, keep in working directory)
git restore --staged file.txt
# or (older syntax)
git reset HEAD file.txt

# File removed from index (or restored to HEAD version)
git status
# Changes not staged for commit:
#   modified:   file.txt

# Working directory unchanged
cat file.txt
# (still has your modifications)
```

#### Scenario 2: Completely Remove File

```bash
# Remove from index AND working directory
git rm file.txt

# File deleted from both
git ls-files | grep file.txt
# (no output - not in index)

ls file.txt
# ls: cannot access 'file.txt': No such file or directory

# Change is staged (deletion)
git status
# Changes to be committed:
#   deleted:    file.txt
```

#### Scenario 3: Remove from Index Only (Keep in Working Directory)

```bash
# Remove from tracking but keep file
git rm --cached file.txt

# File removed from index
git ls-files | grep file.txt
# (no output)

# But file still exists
ls file.txt
# file.txt

# Now it's untracked
git status
# Untracked files:
#   file.txt
```

### When to Use `--cached`

```bash
# Use case: Accidentally tracked a file that should be ignored
echo "secrets.txt" >> .gitignore
git rm --cached secrets.txt
git commit -m "Stop tracking secrets.txt"

# File remains in working directory but won't be tracked
```

---

## Index States and File Tracking

### File Lifecycle in Git

```
Untracked → [git add] → Staged (tracked) → [git commit] → Committed
                              ↓
                        [modify file]
                              ↓
                          Modified
                              ↓
                        [git add]
                              ↓
                          Staged
```

### Checking File State

```bash
# Detailed status
git status

# Short status
git status -s
# Output format:
# ?? untracked.txt    # Untracked
# A  new.txt          # Added to index
# M  modified.txt     # Modified in index
#  M modified2.txt    # Modified in working dir (not staged)
# MM double.txt       # Staged and also modified in working dir
# D  deleted.txt      # Deleted from index
#  D deleted2.txt     # Deleted from working dir
```

### All Possible States

1. **Untracked**: File exists in working directory, not in index
   ```bash
   git ls-files --others --exclude-standard
   ```

2. **Tracked and Unmodified**: File in index matches HEAD
   ```bash
   git ls-files
   ```

3. **Tracked and Modified (Unstaged)**: File modified but changes not in index
   ```bash
   git ls-files --modified
   ```

4. **Staged (Modified and Added)**: Changes in index, different from HEAD
   ```bash
   git diff --cached --name-only
   ```

5. **Deleted (Unstaged)**: File in index but deleted from working directory
   ```bash
   git ls-files --deleted
   ```

6. **Deleted (Staged)**: File removed from index
   ```bash
   git diff --cached --diff-filter=D --name-only
   ```

### Tracking vs Untracking

#### Stop Tracking a File

```bash
# Method 1: Remove from index only
git rm --cached file.txt
git commit -m "Stop tracking file.txt"

# Method 2: Use .gitignore for future
echo "file.txt" >> .gitignore
git rm --cached file.txt
git commit -m "Stop tracking and ignore file.txt"
```

#### Resume Tracking

```bash
# If file was untracked, just add it
git add file.txt

# If file exists in history but was deleted
git checkout HEAD -- file.txt
# This restores from HEAD to both index and working directory
```

---

## Merge Conflicts and the Index

### How Conflicts Appear in Index

During a merge conflict, Git stores three versions in the index:

```bash
# Start merge
git merge feature-branch
# CONFLICT (content): Merge conflict in file.txt

# Index now has 3 stages for conflicted file
git ls-files --stage
# 100644 1a2b3c4... 1	file.txt  # Stage 1: common ancestor
# 100644 5d6e7f8... 2	file.txt  # Stage 2: current branch (ours)
# 100644 9a0b1c2... 3	file.txt  # Stage 3: merged branch (theirs)
# 100644 3d4e5f6... 0	normal.txt  # Stage 0: no conflict
```

### Stage Number Meanings

- **Stage 0**: Normal file (no conflict)
- **Stage 1**: Base version (common ancestor)
- **Stage 2**: "Ours" version (current branch/HEAD)
- **Stage 3**: "Theirs" version (merging branch)

### Inspecting Conflict Versions

```bash
# Show all conflict versions
git ls-files --stage | grep file.txt

# Extract base version (stage 1)
git show :1:file.txt > file.base.txt

# Extract our version (stage 2)
git show :2:file.txt > file.ours.txt

# Extract their version (stage 3)
git show :3:file.txt > file.theirs.txt

# Using git cat-file
BASE_SHA=$(git ls-files --stage file.txt | awk 'NR==1{print $2}')
OURS_SHA=$(git ls-files --stage file.txt | awk 'NR==2{print $2}')
THEIRS_SHA=$(git ls-files --stage file.txt | awk 'NR==3{print $2}')

git cat-file -p $BASE_SHA > file.base.txt
git cat-file -p $OURS_SHA > file.ours.txt
git cat-file -p $THEIRS_SHA > file.theirs.txt
```

### Resolving Conflicts

```bash
# Edit file to resolve conflict
vim file.txt

# Add resolved version (this clears stages 1, 2, 3 and sets stage 0)
git add file.txt

# Verify conflict is resolved
git ls-files --stage | grep file.txt
# 100644 abc123... 0	file.txt  # Only stage 0 remains

# Complete merge
git commit
```

### Choosing a Version

```bash
# Use "ours" version
git checkout --ours file.txt
git add file.txt

# Use "theirs" version
git checkout --theirs file.txt
git add file.txt

# Use base version (ancestor)
git show :1:file.txt > file.txt
git add file.txt
```

### Aborting Merge

```bash
# This clears the index and restores to pre-merge state
git merge --abort

# Or manually
git reset --merge
```

---

## Advanced Index Operations

### Viewing Index Diff

```bash
# Compare working directory to index
git diff
# Shows unstaged changes

# Compare index to HEAD
git diff --cached
# or
git diff --staged
# Shows staged changes

# Compare working directory to HEAD (skip index)
git diff HEAD
# Shows all changes (staged + unstaged)
```

### Manipulating Index Directly

#### Update Index from Tree

```bash
# Read tree into index (dangerous!)
git read-tree HEAD

# This replaces index with HEAD tree
# Working directory unchanged
```

#### Write Index to Tree

```bash
# Create tree object from current index
TREE_SHA=$(git write-tree)
echo $TREE_SHA

# Verify tree
git cat-file -p $TREE_SHA
# Shows tree structure
```

#### Low-Level Adding

```bash
# Create blob manually
echo "content" | git hash-object -w --stdin
# Output: SHA of blob

# Add to index manually
git update-index --add --cacheinfo 100644 <SHA> path/to/file
```

### Refresh Index

```bash
# Update stat information in index
git update-index --refresh

# This checks if working directory matches index
# Updates timestamps if content unchanged
```

### Assume Unchanged

```bash
# Tell Git to ignore changes to tracked file
git update-index --assume-unchanged file.txt

# Now Git ignores modifications
echo "changes" >> file.txt
git status
# (no changes shown)

# Undo
git update-index --no-assume-unchanged file.txt

# List files marked as assume-unchanged
git ls-files -v | grep '^h'
```

### Skip Worktree

```bash
# Similar to assume-unchanged but stronger
git update-index --skip-worktree file.txt

# Use case: local configuration files
# that should remain different from repository

# Undo
git update-index --no-skip-worktree file.txt

# List files with skip-worktree
git ls-files -v | grep '^S'
```

---

## Index vs Working Directory vs Repository

### The Three Trees

```
┌─────────────────────┐
│ Working Directory   │  ← Your files
│  (Current state)    │
└──────────┬──────────┘
           │ git add
           ↓
┌─────────────────────┐
│   Index (Stage)     │  ← Proposed commit
│  (Staged state)     │
└──────────┬──────────┘
           │ git commit
           ↓
┌─────────────────────┐
│   Repository (HEAD) │  ← Committed history
│  (Committed state)  │
└─────────────────────┘
```

### State Synchronization

```bash
# Working → Index → HEAD (normal workflow)
git add file.txt      # Working → Index
git commit -m "msg"   # Index → HEAD (+ working stays same)

# HEAD → Index → Working (restore)
git reset --soft HEAD~1    # HEAD → (Index unchanged, Working unchanged)
git reset --mixed HEAD~1   # HEAD → Index (Working unchanged) [default]
git reset --hard HEAD~1    # HEAD → Index → Working (dangerous!)

# HEAD → Working (skip index)
git checkout HEAD -- file.txt  # HEAD → Working + Index
git restore file.txt          # Index → Working (default)
git restore --source=HEAD file.txt  # HEAD → Working

# Index → Working
git restore file.txt          # Discard working changes
git checkout -- file.txt      # Old syntax
```

### Visualizing States

```bash
# Create test scenario
echo "version 1" > file.txt
git add file.txt
git commit -m "Initial"

echo "version 2" > file.txt
git add file.txt

echo "version 3" > file.txt

# Now we have 3 different versions:
# - HEAD: "version 1"
# - Index: "version 2"  
# - Working: "version 3"

# View HEAD version
git show HEAD:file.txt
# version 1

# View index version
git show :file.txt
# version 2
# or
git diff HEAD --cached file.txt
# Shows version 2 vs version 1

# View working version
cat file.txt
# version 3

# View diffs
git diff --cached file.txt  # Index vs HEAD (version 2 vs 1)
git diff file.txt           # Working vs Index (version 3 vs 2)
git diff HEAD file.txt      # Working vs HEAD (version 3 vs 1)
```

---

## Practical Scenarios

### Scenario 1: Partially Stage a File

```bash
# File has multiple changes
cat calculator.py
# def add(a, b):
#     return a + b  # Fixed typo
#
# def subtract(a, b):
#     return a - b  # Added new feature

# Stage only the bug fix
git add -p calculator.py
# (select the hunk with the bug fix)

# Verify partial staging
git diff --cached calculator.py  # Shows staged changes
git diff calculator.py           # Shows unstaged changes

# Index has partial changes
git ls-files --stage calculator.py
# Shows current index version
```

### Scenario 2: Recover Accidentally Deleted File

```bash
# File was staged, then accidentally deleted from working directory
rm important.txt

# File still in index!
git ls-files | grep important.txt
# important.txt

# Recover from index
git restore important.txt
# or
git checkout -- important.txt

# File restored!
```

### Scenario 3: Compare Different Versions

```bash
# Set up scenario
echo "v1" > data.txt
git add data.txt && git commit -m "v1"

echo "v2" > data.txt
git add data.txt

echo "v3" > data.txt

# Compare all versions
echo "=== HEAD ==="
git show HEAD:data.txt

echo "=== INDEX ==="
git show :data.txt

echo "=== WORKING ==="
cat data.txt

# Extract specific version
git show HEAD:data.txt > data.v1.txt
git show :data.txt > data.v2.txt
cp data.txt data.v3.txt
```

### Scenario 4: Fix Wrong File in Staging

```bash
# Accidentally staged wrong file
git add wrong_file.txt

# Unstage it
git restore --staged wrong_file.txt

# Stage correct file
git add correct_file.txt
```

### Scenario 5: Amend Commit by Modifying Index

```bash
# Last commit is wrong
git log -1 --oneline
# abc123 Wrong commit

# Fix the files
echo "corrected" > file.txt
git add file.txt

# Amend (replace last commit with current index)
git commit --amend --no-edit

# Index became new commit
```

### Scenario 6: Cherry-Pick Specific Files from Commit

```bash
# Want specific files from another commit
# but not merge entire commit

# Get file from specific commit into index
git checkout abc123 -- file1.txt file2.txt

# Files now staged
git status
# Changes to be committed:
#   modified:   file1.txt
#   modified:   file2.txt

# Commit
git commit -m "Cherry-picked files from abc123"
```

### Scenario 7: Stash vs Index

```bash
# You have staged changes
git add file1.txt

# And unstaged changes
echo "more" >> file2.txt

# Stash including staged changes
git stash push

# Both index and working directory clean
git status
# nothing to commit, working tree clean

# Apply stash
git stash pop

# Staged status restored!
git status
# Changes to be committed:
#   modified:   file1.txt
# Changes not staged for commit:
#   modified:   file2.txt
```

### Scenario 8: Interactive Rebase and Index

```bash
# During interactive rebase, each step updates index

git rebase -i HEAD~3

# Edit commits: Git temporarily sets index to each commit
# You can modify index before continuing

# Edit mode example:
git rebase -i HEAD~2
# (mark commit as 'edit')

# Rebase stops at that commit
# Index contains that commit's state
git ls-files --stage

# Modify index
echo "change" >> file.txt
git add file.txt

# Continue with modified index
git rebase --continue
# Index changes are incorporated into commit
```

### Scenario 9: Clean Index After Failed Operation

```bash
# Some operation failed and left index messy
git status
# Shows unexpected staged changes

# Reset index to HEAD (keep working directory)
git reset

# Or more explicitly
git reset --mixed HEAD

# Index now matches HEAD
```

### Scenario 10: Analyze Index Contents

```bash
# Full index analysis script
echo "=== All Tracked Files ==="
git ls-files

echo -e "\n=== Files with Stage Info ==="
git ls-files --stage

echo -e "\n=== Modified Files ==="
git ls-files --modified

echo -e "\n=== Deleted Files ==="
git ls-files --deleted

echo -e "\n=== Untracked Files ==="
git ls-files --others --exclude-standard

echo -e "\n=== Ignored Files ==="
git ls-files --ignored --exclude-standard

echo -e "\n=== Index Statistics ==="
echo "Total tracked files: $(git ls-files | wc -l)"
echo "Modified unstaged: $(git ls-files --modified | wc -l)"
echo "Untracked files: $(git ls-files --others --exclude-standard | wc -l)"
```

---

## Summary

### Key Takeaways

1. **Index is the staging area**: Intermediate layer between working directory and repository
2. **Binary format**: `.git/index` stores metadata and SHA-1 references to blob objects
3. **Three-stage system**: Working directory → Index → Repository
4. **Conflict resolution**: Index uses stage numbers (0-3) to track conflict versions
5. **Inspection tool**: `git ls-files` is essential for understanding index state

### Common Commands Summary

```bash
# Inspection
git ls-files                    # List tracked files
git ls-files --stage            # Show with metadata
git status                      # High-level status
git diff                        # Working vs Index
git diff --cached               # Index vs HEAD

# Adding
git add <file>                  # Stage file
git add -p                      # Interactive staging
git add -A                      # Stage all changes

# Removing/Unstaging  
git restore --staged <file>     # Unstage file
git rm --cached <file>          # Untrack file
git reset HEAD <file>           # Unstage (old syntax)

# Advanced
git update-index --assume-unchanged <file>
git update-index --skip-worktree <file>
git read-tree                   # Read tree to index
git write-tree                  # Write index to tree
```

### Understanding File Flow

```
[Create File]
     ↓
Untracked (not in index)
     ↓ git add
Staged (in index, stage 0)
     ↓ git commit
Committed (in repository)
     ↓ modify file
Modified (working ≠ index)
     ↓ git add
Staged (index updated)
     ↓ git commit
Committed (new version)
```

---

## Further Reading

- `man git-ls-files` - Detailed ls-files documentation
- `man git-update-index` - Low-level index manipulation
- `.git/index` format specification in Git documentation
- Git internals chapter in Pro Git book

---

*This guide covers comprehensive index internals and operations. The index is central to Git's workflow, understanding it deeply will make you more effective with Git.*