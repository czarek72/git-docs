# Git Diff Guide: Understanding Changes in Git

## Table of Contents
1. [Introduction to git diff](#introduction-to-git-diff)
2. [Understanding Diff Format](#understanding-diff-format)
3. [Basic git diff Commands](#basic-git-diff-commands)
4. [Advanced Diff Options](#advanced-diff-options)
5. [Related Commands](#related-commands)
6. [Practical Examples](#practical-examples)
7. [Diff Tools and Configuration](#diff-tools-and-configuration)

## Introduction to git diff

`git diff` is one of the most essential Git commands for understanding changes in your repository. It shows the differences between:
- Working directory and staging area
- Staging area and last commit
- Different commits
- Different branches
- And more

Understanding how to read and work with diff output is crucial for effective version control.

## Understanding Diff Format

### The Unified Diff Format

Git uses the **unified diff format**, which is the standard format for showing differences between files. Here's a breakdown of what you'll see:

```diff
diff --git a/file.txt b/file.txt
index 83db48f..bf269f4 100644
--- a/file.txt
+++ b/file.txt
@@ -1,4 +1,5 @@
 This is line 1
-This is line 2
+This is the modified line 2
+This is a new line 3
 This is line 4
 This is line 5
```

### Line-by-Line Explanation

#### 1. Header Lines

```diff
diff --git a/file.txt b/file.txt
```
- Shows which files are being compared
- `a/file.txt` = old version (before)
- `b/file.txt` = new version (after)

#### 2. Extended Header

```diff
index 83db48f..bf269f4 100644
```
- `83db48f` = SHA-1 hash of old blob
- `bf269f4` = SHA-1 hash of new blob
- `100644` = file mode (regular file with permissions)

#### 3. File Markers

```diff
--- a/file.txt
+++ b/file.txt
```
- `---` marks the old version
- `+++` marks the new version

#### 4. Hunk Header

```diff
@@ -1,4 +1,5 @@
```
This is the **chunk header** or **hunk header**:
- `-1,4` = In old file: starting at line 1, showing 4 lines
- `+1,5` = In new file: starting at line 1, showing 5 lines

#### 5. Change Lines

- Lines starting with `-` (red) = removed from old version
- Lines starting with `+` (green) = added in new version
- Lines with no prefix (context lines) = unchanged

### File Status Indicators

When files are added or deleted:

```diff
# New file
diff --git a/newfile.txt b/newfile.txt
new file mode 100644
index 0000000..e69de29

# Deleted file
diff --git a/oldfile.txt b/oldfile.txt
deleted file mode 100644
index e69de29..0000000

# Renamed file
diff --git a/old-name.txt b/new-name.txt
similarity index 100%
rename from old-name.txt
rename to new-name.txt
```

## Basic git diff Commands

### 1. Working Directory vs Staging Area

```bash
git diff
```
Shows changes in working directory that are **not yet staged**.

**Example:**
```bash
# Modify a file
echo "new content" >> file.txt

# See unstaged changes
git diff
```

### 2. Staging Area vs Last Commit

```bash
git diff --cached
# or
git diff --staged
```
Shows changes that are **staged** (ready to commit).

**Example:**
```bash
# Stage a file
git add file.txt

# See staged changes
git diff --staged
```

### 3. Working Directory vs Last Commit

```bash
git diff HEAD
```
Shows **all changes** (staged + unstaged) since last commit.

### 4. Compare Specific Commits

```bash
# Compare two commits
git diff commit1 commit2

# Compare current HEAD with a specific commit
git diff abc123

# Compare with previous commit
git diff HEAD~1 HEAD
```

### 5. Compare Branches

```bash
# Compare current branch with another
git diff main

# Compare two specific branches
git diff main..feature-branch

# Show what's in feature-branch but not in main
git diff main...feature-branch
```

### 6. Compare Specific Files

```bash
# Diff a specific file
git diff file.txt

# Diff specific file between commits
git diff commit1 commit2 -- file.txt

# Diff multiple files
git diff file1.txt file2.txt
```

## Advanced Diff Options

### Formatting and Display

```bash
# Show statistics only
git diff --stat

# Show summary of changes
git diff --summary

# Show shortened diff
git diff --shortstat

# Show word-level diff (instead of line-level)
git diff --word-diff

# Color words (default)
git diff --word-diff=color

# Show only names of changed files
git diff --name-only

# Show names and status (A=added, M=modified, D=deleted)
git diff --name-status
```

### Example Output

```bash
$ git diff --stat
 file1.txt | 10 ++++++++--
 file2.txt |  5 ++---
 file3.txt | 23 +++++++++++++++++++++++
 3 files changed, 33 insertions(+), 5 deletions(-)

$ git diff --name-status
M       file1.txt
M       file2.txt
A       file3.txt
```

### Context Control

```bash
# Show more context lines (default is 3)
git diff -U5  # Show 5 lines of context
git diff --unified=10  # Show 10 lines

# Show function/method names in hunk headers
git diff -p

# Minimal diff
git diff --minimal
```

### Ignoring Changes

```bash
# Ignore whitespace changes
git diff -w
git diff --ignore-all-space

# Ignore whitespace at end of line
git diff --ignore-space-at-eol

# Ignore changes in amount of whitespace
git diff -b
git diff --ignore-space-change

# Ignore blank lines
git diff --ignore-blank-lines
```

### Binary and Special Files

```bash
# Show binary files as binary
git diff --binary

# Don't show binary file diffs
git diff --no-binary

# Treat all files as text
git diff -a
git diff --text
```

## Related Commands

### git show

Display information about Git objects (commits, tags, trees).

```bash
# Show the last commit's changes
git show

# Show a specific commit
git show abc123

# Show specific file from a commit
git show abc123:path/to/file.txt

# Show changes in specific commit for specific file
git show abc123 -- file.txt

# Show commit without diff
git show --no-patch abc123
git show -s abc123
```

### git log with diffs

```bash
# Show commit history with diffs
git log -p

# Show last N commits with diffs
git log -p -2

# Show commits affecting specific file
git log -p -- file.txt

# Show statistics for each commit
git log --stat

# Compact summary
git log --oneline --stat
```

### git difftool

Use external diff tools (vimdiff, meld, kdiff3, etc.).

```bash
# Launch configured difftool
git difftool

# Use specific tool
git difftool --tool=meld

# Compare specific commits
git difftool commit1 commit2
```

### git range-diff

Compare two commit ranges (useful for reviewing rebases).

```bash
# Compare commit ranges
git range-diff main...feature-old main...feature-new

# Compare before and after rebase
git range-diff main...HEAD@{1} main...HEAD
```

## Practical Examples

### Example 1: Reviewing Changes Before Commit

```bash
# Make some changes to files
echo "new feature" >> app.js
echo "update docs" >> README.md

# See what changed
git diff

# Stage some changes
git add app.js

# Review staged changes
git diff --staged

# Review all changes (staged + unstaged)
git diff HEAD

# Commit staged changes
git commit -m "Add new feature"
```

### Example 2: Comparing Branches Before Merge

```bash
# See what's different in feature branch
git diff main...feature-branch

# See statistics
git diff --stat main...feature-branch

# See only file names
git diff --name-only main...feature-branch

# See detailed diff for specific file
git diff main...feature-branch -- src/important-file.js
```

### Example 3: Finding When a Change Was Introduced

```bash
# See who changed what in a file
git log -p -- file.txt

# Find when a specific line was changed
git log -p -S "search string" -- file.txt

# Show commits that added or removed a function
git log -p -G "function.*myFunction" -- file.js
```

### Example 4: Checking Changes in a Commit Range

```bash
# See changes between two tags
git diff v1.0.0 v2.0.0

# See changes in last 5 commits
git diff HEAD~5 HEAD

# See what changed this week
git diff @{1.week.ago} @{now}

# See changes since yesterday
git diff @{yesterday} @{now}
```

### Example 5: Ignoring Whitespace Changes

```bash
# Someone reformatted the code - ignore whitespace
git diff -w

# Compare ignoring all whitespace changes
git diff -b main feature-branch

# Show only actual code changes
git diff --ignore-all-space --ignore-blank-lines
```

### Example 6: Reviewing Merge Conflicts

```bash
# During a merge conflict
git diff

# See changes from merge base
git diff --merge

# See differences against both parents
git diff HEAD
git diff MERGE_HEAD

# See 3-way diff
git diff --cc
```

### Example 7: Creating Patches

```bash
# Create a patch file
git diff > my-changes.patch

# Create patch from specific commits
git diff commit1 commit2 > feature.patch

# Apply a patch
git apply my-changes.patch

# Test patch without applying
git apply --check my-changes.patch
```

### Example 8: Word-Level Diff for Documentation

```bash
# Better for prose/documentation
git diff --word-diff README.md

# Color-coded word diff
git diff --word-diff=color docs/

# Inline word diff
git diff --color-words README.md
```

## Diff Tools and Configuration

### Configuring Default Behavior

```bash
# Set default context lines
git config --global diff.context 5

# Enable rename detection
git config --global diff.renames true

# Set diff algorithm
git config --global diff.algorithm histogram

# Always show file statistics
git config --global diff.stat true
```

### Setting Up External Diff Tools

```bash
# Configure vimdiff
git config --global diff.tool vimdiff

# Configure meld
git config --global diff.tool meld
git config --global difftool.meld.cmd 'meld "$LOCAL" "$REMOTE"'

# Configure VS Code
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'

# Don't prompt before launching difftool
git config --global difftool.prompt false
```

### Using Different Diff Algorithms

Git supports different diff algorithms:

```bash
# Myers (default) - good general purpose
git diff --diff-algorithm=myers

# Minimal - produces smallest possible diff
git diff --diff-algorithm=minimal

# Patience - better for code with indentation
git diff --diff-algorithm=patience

# Histogram - fast and handles indentation well
git diff --diff-algorithm=histogram
```

### Diff Drivers for Specific File Types

```bash
# Configure diff driver for Word documents
git config diff.word.textconv "strings"

# In .gitattributes:
# *.docx diff=word

# Configure diff for images (using exiftool)
git config diff.exif.textconv "exiftool"

# In .gitattributes:
# *.jpg diff=exif
# *.png diff=exif
```

## Quick Reference

### Common Commands

| Command | Description |
|---------|-------------|
| `git diff` | Working directory vs staging area |
| `git diff --staged` | Staging area vs last commit |
| `git diff HEAD` | All changes vs last commit |
| `git diff commit1 commit2` | Compare two commits |
| `git diff branch1..branch2` | Compare branches |
| `git diff --stat` | Show statistics only |
| `git diff --name-only` | Show changed file names |
| `git diff -w` | Ignore whitespace |
| `git show` | Show last commit changes |
| `git log -p` | Show commit history with diffs |

### Useful Flags

| Flag | Description |
|------|-------------|
| `--cached` / `--staged` | Compare staging area with last commit |
| `--stat` | Show change statistics |
| `--summary` | Show summary (new/deleted files) |
| `--name-only` | List changed file names |
| `--name-status` | List files with status (A/M/D) |
| `-w` / `--ignore-all-space` | Ignore whitespace changes |
| `-b` / `--ignore-space-change` | Ignore whitespace amount changes |
| `--word-diff` | Show word-level differences |
| `-U<n>` / `--unified=<n>` | Show n context lines |
| `--color-words` | Color word-level changes |

### Two-Dot vs Three-Dot

```bash
# Two-dot (..) - shows all changes between two points
git diff branch1..branch2
# Equivalent to: git diff branch1 branch2

# Three-dot (...) - shows changes on branch2 since common ancestor with branch1
git diff branch1...branch2
# Useful for seeing what changed in a feature branch
```

## Tips and Best Practices

1. **Review before committing**: Always use `git diff --staged` before committing to review what you're about to commit.

2. **Use specific file diffs**: When working on large changesets, diff specific files to focus on relevant changes.

3. **Leverage word-diff for documentation**: Use `--word-diff` for prose, documentation, and configuration files.

4. **Configure a good difftool**: External diff tools provide better visualization, especially for complex changes.

5. **Use ignore-whitespace wisely**: Use `-w` when reviewing code that's been reformatted, but be careful not to miss important changes.

6. **Check diff before push**: Run `git diff origin/main..HEAD` to see what you're about to push.

7. **Use three-dot for feature branches**: `git diff main...feature` shows only the feature branch changes.

8. **Create patches for sharing**: Use `git diff > file.patch` to share changes without committing.

9. **Understand the diff format**: Taking time to understand unified diff format makes code review much easier.

10. **Combine with log**: Use `git log -p` to see how code evolved over time.

## Conclusion

Mastering `git diff` and understanding its output format is essential for effective version control. Whether you're reviewing your own changes before committing, conducting code reviews, or investigating when bugs were introduced, `git diff` is an indispensable tool in your Git toolkit.

Key takeaways:
- The unified diff format shows context and changes clearly
- Different variants compare different parts of your repository
- Many options exist to customize output to your needs
- Related commands like `git show` and `git log -p` extend diff functionality
- External diff tools can enhance your workflow

Practice reading diffs regularly, and soon it will become second nature to spot changes, understand their impact, and review code effectively.
