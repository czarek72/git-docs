# Git Range Notation and Revision Specifications

A comprehensive guide to Git's powerful syntax for specifying commits, ranges, and revisions.

## Table of Contents

1. [Basic Commit References](#basic-commit-references)
2. [Range Notation](#range-notation)
3. [Multiple Parent References](#multiple-parent-references)
4. [Exclusion and Inclusion Patterns](#exclusion-and-inclusion-patterns)
5. [Special Notations](#special-notations)
6. [Date and Time Based Ranges](#date-and-time-based-ranges)
7. [Path and Pattern Specifications](#path-and-pattern-specifications)
8. [Practical Examples](#practical-examples)
9. [Edge Cases and Gotchas](#edge-cases-and-gotchas)

---

## Basic Commit References

### Single Commit References

Git provides multiple ways to reference a specific commit:

#### 1. Full SHA-1 Hash
```bash
git show 3a8f7b9c2d1e6f4a5b8c9d2e1f3a4b5c6d7e8f9a
```
The complete 40-character SHA-1 hash uniquely identifies a commit.

#### 2. Short SHA-1 Hash
```bash
git show 3a8f7b9
```
Git accepts abbreviated hashes (minimum 4 characters, typically 7+ for uniqueness).

#### 3. Branch Names
```bash
git show main
git show feature/user-authentication
```
References the commit that the branch currently points to.

#### 4. Tag Names
```bash
git show v1.0.0
git show release-2023-12
```
References the commit that the tag points to (or the tag object itself for annotated tags).

#### 5. HEAD
```bash
git show HEAD
```
References the current commit (the tip of the current branch or detached HEAD state).

### Relative References Using ^ and ~

Git provides two operators for navigating commit ancestry:

#### The Tilde Operator (~)

The tilde operator moves back through **first parents** in a linear fashion:

```bash
# Go back 1 commit from HEAD
git show HEAD~1    # Same as HEAD~

# Go back 2 commits from HEAD
git show HEAD~2

# Go back 5 commits from main
git show main~5
```

**Visualization:**
```
HEAD~0 = HEAD
HEAD~1 = parent of HEAD
HEAD~2 = grandparent of HEAD
HEAD~3 = great-grandparent of HEAD
```

#### The Caret Operator (^)

The caret operator selects a specific parent of a commit:

```bash
# First parent of HEAD (same as HEAD~1)
git show HEAD^

# For merge commits - select different parents
git show HEAD^1    # First parent (same as HEAD^)
git show HEAD^2    # Second parent
git show HEAD^3    # Third parent (for octopus merges)
```

**Merge commit example:**
```
    A---B---C---M     (main branch, M is a merge commit)
             \ /
              D        (feature branch)

M^1 = C  (first parent, the commit on main)
M^2 = D  (second parent, the commit from feature branch)
```

#### Key Differences Between ^ and ~

| Notation | Meaning | Use Case |
|----------|---------|----------|
| `HEAD^` | First parent of HEAD | Same as HEAD~1 |
| `HEAD^2` | Second parent of HEAD | Access merged branch in merge commit |
| `HEAD~2` | Grandparent of HEAD | Go back 2 generations linearly |
| `HEAD^^` | Grandparent via two single-step moves | Same as HEAD~2 |

#### Combining ^ and ~

You can chain these operators:

```bash
# Go back 3 commits, then get second parent of that commit
git show HEAD~3^2

# Get first parent's first parent
git show HEAD^^     # Same as HEAD~2

# Complex navigation
git show main~2^2~1
```

**Example Visualization:**
```
    A---B---C---D---E---F---M       (main)
                 \         /
                  G---H---I         (feature)

M     = merge commit
M^1   = F (first parent)
M^2   = I (second parent)
M~1   = F (same as M^1)
M~2   = E
M^2~1 = H (second parent, then back one)
M~2^2 = would be the second parent of E (if E were a merge)
```

### Practical Examples

```bash
# Show the last 3 commits
git log HEAD~3..HEAD

# Show what was merged
git show HEAD^2

# Compare before and after a merge
git diff HEAD~1 HEAD

# See the parent of a specific commit
git show abc123^
```

---

## Range Notation

Git's range notation allows you to specify sets of commits. The behavior differs depending on the command used.

### Two-Dot Notation (A..B)

The two-dot notation `A..B` means "commits reachable from B but not from A".

#### With git log

```bash
git log main..feature
```

**Meaning:** Show commits in `feature` that are **not** in `main`.

**Visualization:**
```
    A---B---C         (main)
         \
          D---E---F   (feature)

git log main..feature  shows: D, E, F
```

**Use case:** "What's new in my feature branch?"

```bash
# See commits in feature branch not yet in main
git log main..feature

# See what you're about to push
git log origin/main..HEAD

# See commits you don't have yet
git log HEAD..origin/main
```

#### With git diff

```bash
git diff main..feature
```

**Meaning:** Show the difference between the **tip of main** and the **tip of feature**.

⚠️ **Important:** This is equivalent to `git diff main feature` (the `..` is mostly ignored).

### Three-Dot Notation (A...B)

The three-dot notation `A...B` has **different behavior** depending on the command.

#### With git log

```bash
git log main...feature
```

**Meaning:** Show commits reachable from **either** main or feature, but **not both** (symmetric difference).

**Visualization:**
```
        C---D         (main)
       /
    A---B
       \
        E---F         (feature)

git log main...feature  shows: C, D, E, F
(Excludes A and B, which are in both)
```

**Use case:** "Show all commits that differ between these branches."

#### With git diff

```bash
git diff main...feature
```

**Meaning:** Show changes in `feature` since it **diverged from main** (since the merge base).

**Visualization:**
```
        C---D         (main)
       /
    A---B
       \
        E---F         (feature)

git diff main...feature
Shows: Changes from B to F (not including C and D)
```

**Use case:** "What changed in my feature branch specifically?"

This is extremely useful for reviewing feature branch changes without noise from main branch updates.

```bash
# Perfect for code review - shows only your changes
git diff main...feature

# Compare with what would be shown with two-dot
git diff main..feature    # Shows B→D vs B→F (all differences)
git diff main...feature   # Shows B→F only (your changes)
```

### Omitting One Side

#### A.. (everything after A)

```bash
git log main..
```

**Meaning:** Commits reachable from HEAD but not from main.

Equivalent to: `git log main..HEAD`

**Use case:** "What have I done since branching from main?"

```bash
# Before pushing
git log origin/main..

# What's new in current branch
git log main..
```

#### ..B (everything before B, excluding your current position)

```bash
git log ..origin/main
```

**Meaning:** Commits reachable from origin/main but not from HEAD.

Equivalent to: `git log HEAD..origin/main`

**Use case:** "What new commits are on remote that I don't have?"

```bash
# See incoming changes after fetch
git fetch
git log ..origin/main

# What would be pulled
git log ..@{u}
```

### Command-Specific Behavior Summary

| Notation | git log | git diff |
|----------|---------|----------|
| `A..B` | Commits in B not in A | Changes from A to B |
| `A...B` | Commits in A or B, not both | Changes from merge-base to B |
| `A B` | Commits in A or B | Not applicable |
| `^A B` | Commits in B not in A | Not applicable |

---

## Multiple Parent References

When working with merge commits, you often need to reference specific parents.

### Merge Commit Structure

A merge commit has two or more parents:

```
    A---B---C         (main)
         \   \
          \   M       (merge commit)
           \ /
            D---E     (feature)

M^1 = C  (first parent - main branch)
M^2 = E  (second parent - feature branch)
```

### Parent Selection with ^n

```bash
# First parent (always exists)
git show HEAD^1    # or just HEAD^

# Second parent (exists for merge commits)
git show HEAD^2

# Third parent (for octopus merges)
git show HEAD^3
```

### Navigating from Merge Parents

You can chain operators to navigate complex histories:

```bash
# Second parent, then go back one commit
git show HEAD^2~1

# First parent, go back two commits
git show HEAD^1~2    # or HEAD~2 (same thing)

# Second parent's parent (if it was also a merge)
git show HEAD^2^1
```

### Practical Merge Commit Examples

#### See what was merged

```bash
# Show the branch that was merged in
git log HEAD^2

# Show commits unique to the merged branch
git log HEAD^1..HEAD^2

# Show the diff of what was merged
git diff HEAD^1..HEAD^2
```

#### Compare merge strategies

```bash
# Show differences between what was in main vs what was merged
git diff HEAD^1 HEAD^2

# Show combined effect of the merge
git diff HEAD^1 HEAD
```

#### Find merge commits

```bash
# List all merge commits
git log --merges

# List merge commits in a range
git log --merges main..feature

# Show only first parents (skips merge commits)
git log --first-parent
```

### Complex History Navigation

```
        A---B---C---M1---M2        (main)
             \     /     /
              D---E     /          (feature1)
                   \   /
                    F-G            (feature2)

M1^1 = C   (main before first merge)
M1^2 = E   (tip of feature1)
M2^1 = M1  (state after first merge)
M2^2 = G   (tip of feature2)

M2~1 = M1       (go back one commit on main)
M2~2 = C        (go back two commits on main)
M2^2~1 = F      (second parent, back one)
M2~1^2 = E      (back one, then second parent)
```

Commands:
```bash
# Show what feature1 added
git log M1^1..M1^2

# Show what feature2 added
git log M2^1..M2^2

# Show all changes since before merges started
git diff M1^ M2

# Show both features combined
git log M1^^..M2 --first-parent --no-merges
```

---

## Exclusion and Inclusion Patterns

Git allows complex commit selection using inclusion and exclusion patterns.

### Using ^ or --not for Exclusion

The caret symbol (^) before a commit reference means "exclude this and its ancestors".

```bash
# Commits in feature but not in main
git log ^main feature
# Equivalent to: git log main..feature

# Commits in either branch1 or branch2, but not in main
git log ^main branch1 branch2

# Using --not flag
git log main --not feature
```

### Multiple Inclusion/Exclusion

```bash
# Commits reachable from A or B, but not from C
git log A B ^C

# More complex: in A or B or C, but not in D or E
git log A B C ^D ^E

# Using --not to exclude multiple
git log A B --not C D E
```

**Visualization:**
```
        C---D---E     (main)
       /         \
    A---B         M   (merge)
       \         /
        F---G---H     (feature)

git log ^main feature     shows: F, G, H
git log ^feature main     shows: C, D, E
git log main feature ^A   shows: C, D, E, F, G, H
```

### The --all, --branches, --tags Selectors

These flags select multiple refs at once:

#### --all

```bash
# Show all commits from all refs (branches, tags, remotes)
git log --all

# All commits not in main
git log ^main --all

# All commits, but exclude main and develop
git log --all ^main ^develop
```

#### --branches

```bash
# All local branches
git log --branches

# All branches except main
git log --branches ^main

# Specific branch pattern
git log --branches=feature/*
```

#### --tags

```bash
# All commits reachable from any tag
git log --tags

# Commits in tags but not in branches
git log --tags ^--branches

# Specific tag pattern
git log --tags=v1.*
```

#### --remotes

```bash
# All remote-tracking branches
git log --remotes

# Commits in remote branches not in local branches
git log --remotes ^--branches

# Specific remote
git log --remotes=origin/*
```

### Combining Patterns

```bash
# Everything except main and remote branches
git log --branches --tags ^main ^--remotes

# All local work not pushed
git log --branches ^--remotes

# Find branches containing a commit
git branch --contains <commit>

# Find tags containing a commit
git tag --contains <commit>
```

### Practical Examples

#### Find commits in multiple features but not in main

```bash
git log ^main feature1 feature2 feature3
```

#### Find commits unique to a branch (not in any other branch)

```bash
git log --branches ^main ^feature1 ^feature2 feature3
```

#### Review all recent work across branches

```bash
git log --all --since="1 week ago" ^main
```

#### See what hasn't been merged to main

```bash
git log --branches ^main --oneline
```

---

## Special Notations

Git provides several special shorthand notations for common scenarios.

### @ as Shorthand for HEAD

The `@` symbol is a convenient alias for `HEAD`:

```bash
# These are equivalent
git show @
git show HEAD

# With relative references
git show @~1
git show HEAD~1

# In ranges
git log @~3..@
git log HEAD~3..HEAD

# With upstream
git log @{u}..@
git log HEAD@{upstream}..HEAD
```

**Use case:** Saves typing, especially in complex expressions.

### @{-N} for Previous Branches

The `@{-N}` notation refers to the Nth previous branch you were on (from reflog).

```bash
# The previous branch
git checkout @{-1}
# Common shorthand
git checkout -

# Two branches ago
git checkout @{-2}

# Show what changed since you switched branches
git log @{-1}..@

# Diff against previous branch
git diff @{-1}
```

**Example workflow:**
```bash
git checkout feature
# ... do some work ...
git checkout main
git merge @{-1}    # Merges the feature branch
```

**Use case:** Quick navigation and referencing recently visited branches.

### Reflog References

The reflog tracks where your HEAD and branch tips have been. You can reference these historical positions.

#### HEAD@{N} - Position in HEAD's Reflog

```bash
# Where HEAD was 1 move ago
git show HEAD@{1}

# Where HEAD was 2 moves ago
git show HEAD@{2}

# Show reflog
git reflog
```

**Example reflog:**
```
a1b2c3d HEAD@{0}: commit: Add new feature
e4f5g6h HEAD@{1}: checkout: moving from main to feature
i7j8k9l HEAD@{2}: commit: Fix bug
```

#### Branch@{N} - Position in Branch's Reflog

```bash
# Where main was 1 update ago
git show main@{1}

# Where feature was 3 updates ago
git show feature@{3}

# Show branch reflog
git reflog show main
```

#### Date-Based Reflog References

```bash
# Where HEAD was yesterday
git show HEAD@{yesterday}

# Where main was 2 days ago
git show main@{2.days.ago}

# Where HEAD was at a specific time
git show HEAD@{2024-01-15}
git show HEAD@{2024-01-15.14:30}

# Where HEAD was 1 week ago
git show HEAD@{1.week.ago}
```

**Supported units:** minutes, hours, days, weeks, months, years

**Use cases:**
```bash
# What did I have before that bad merge?
git diff HEAD@{1} HEAD

# Recover lost commits after reset
git reset --hard HEAD@{1}

# See history of a branch
git log main@{1.month.ago}..main
```

### Upstream Tracking Notation

The `@{upstream}` or `@{u}` notation refers to the upstream branch.

```bash
# Show upstream branch for current branch
git log @{u}

# Shorthand
git log @{u}

# What you're about to push
git log @{u}..@

# What you're about to pull/merge
git log @..@{u}

# Diff against upstream
git diff @{u}

# Show upstream for specific branch
git log main@{u}
git log feature@{upstream}
```

**Use cases:**
```bash
# Check if you're ahead or behind
git rev-list --left-right --count @{u}...@

# Push current branch to its upstream
git push

# Rebase onto upstream
git rebase @{u}

# Show what's different
git log --oneline @{u}..@
```

### Push Target Notation @{push}

Refers to where the current branch would be pushed:

```bash
# Compare with push destination
git log @{push}..@

# Diff against push target
git diff @{push}
```

**Difference from @{upstream}:**
- `@{u}` - where you pull from
- `@{push}` - where you push to (can differ in triangular workflows)

### Combining Special Notations

```bash
# Previous branch vs current
git diff @{-1}..@

# Your work since you checked out the branch
git log @{u}..@

# Where you were yesterday vs now
git diff @{yesterday}..@

# What changed in main while you worked
git log @...main@{u}
```

---

## Date and Time Based Ranges

Git provides powerful date and time filtering for commit selection.

### Using --since and --until

These flags filter commits by commit date:

```bash
# Commits since a specific date
git log --since="2024-01-01"

# Commits in the last 2 weeks
git log --since="2 weeks ago"

# Commits between two dates
git log --since="2024-01-01" --until="2024-01-31"

# Relative times
git log --since="3 days ago"
git log --since="1 month ago"
git log --since="2 years ago"
```

### Aliases: --after and --before

```bash
# These are equivalent to --since and --until
git log --after="2024-01-01"
git log --before="2024-12-31"

# Combine them
git log --after="1 week ago" --before="2 days ago"
```

### Date Formats

Git accepts many date formats:

#### Absolute Dates

```bash
# ISO 8601 format
git log --since="2024-01-15"
git log --since="2024-01-15T14:30:00"

# Human-readable
git log --since="January 15, 2024"
git log --since="Jan 15 2024"

# Short format
git log --since="15.01.2024"
```

#### Relative Dates

```bash
# Time ago
git log --since="3 hours ago"
git log --since="5 minutes ago"
git log --since="2 days ago"
git log --since="3 weeks ago"
git log --since="1 month ago"
git log --since="2 years ago"

# Yesterday/today
git log --since="yesterday"
git log --until="today"
```

#### Reflog Date Specifications

When using reflog references, you can specify dates:

```bash
# Where HEAD was 2 days ago
git show HEAD@{2.days.ago}

# Where main was yesterday
git show main@{yesterday}

# Specific time
git show HEAD@{2024-01-15.14:30}

# One month ago
git diff HEAD@{1.month.ago}..HEAD
```

### Practical Examples

#### Find commits by time period

```bash
# All commits this week
git log --since="1 week ago"

# All commits today
git log --since="midnight"

# All commits yesterday
git log --since="yesterday" --until="midnight"

# Last 24 hours
git log --since="24 hours ago"
```

#### Author activity in timeframe

```bash
# What Alice did last week
git log --author="Alice" --since="1 week ago"

# All team activity this month
git log --since="1 month ago" --oneline
```

#### Date ranges with branches

```bash
# Commits in feature branch from last week
git log --since="1 week ago" feature

# Recent commits not yet in main
git log main..feature --since="3 days ago"

# What was added to main this month
git log --since="1 month ago" main ^main@{1.month.ago}
```

#### Finding regressions by date

```bash
# Find commits between when it worked and broke
git log --since="2024-01-10" --until="2024-01-15" --oneline

# With bisect
git bisect start HEAD HEAD@{2.weeks.ago}
```

#### Time-based statistics

```bash
# Commits per day in the last month
git log --since="1 month ago" --pretty=format:"%ad" --date=short | sort | uniq -c

# Most active days
git log --since="3 months ago" --pretty=format:"%ad" --date=short | sort | uniq -c | sort -rn | head -10

# Work patterns
git log --since="1 month ago" --pretty=format:"%ad" --date=format:"%H" | sort | uniq -c
```

### Combining with Other Options

```bash
# Recent merges
git log --merges --since="1 week ago"

# Recent work on specific file
git log --since="1 month ago" -- path/to/file.txt

# Recent commits by pattern
git log --grep="bug" --since="2 weeks ago"

# Recent commits changing specific code
git log -S"function_name" --since="1 month ago"
```

### Notes on Commit Dates vs Author Dates

Git has two timestamps per commit:
- **Author date**: When the change was originally made
- **Commit date**: When the commit was created/applied

```bash
# Filter by author date (default)
git log --since="1 week ago"

# Filter by committer date
git log --since="1 week ago" --committer-date-is-author-date

# Show both dates
git log --pretty=format:"%h %ad %cd %s" --date=short
```

This matters for rebased or cherry-picked commits, where author date and commit date may differ significantly.

---

## Path and Pattern Specifications

Git allows you to limit operations to specific files or directories.

### Separating Revisions from Paths

The `--` separator distinguishes revision specifications from file paths:

```bash
git log [<options>] [<revision range>] -- [<path>...]
```

#### Why -- is Important

```bash
# Ambiguous - is "main" a branch or a file?
git log main

# Clear - main is a revision
git log main --

# Clear - main is a file/directory
git log -- main

# Both revision and path
git log feature -- src/
```

### Basic Path Limiting

```bash
# Commits affecting a specific file
git log -- path/to/file.txt

# Commits affecting a directory
git log -- src/components/

# Multiple paths
git log -- file1.txt file2.txt src/

# All files in directory recursively
git log -- src/
```

### Path with Ranges

```bash
# Commits in feature branch that modified a file
git log main..feature -- src/app.js

# Changes to file since last week
git log --since="1 week ago" -- config.yml

# What changed in this file on main
git log main -- README.md

# File history between two commits
git log abc123..def456 -- important.txt
```

### Glob Patterns

Git supports glob patterns for path matching:

```bash
# All JavaScript files
git log -- '*.js'

# All files in any __tests__ directory
git log -- '**/__tests__/**'

# All markdown files anywhere
git log -- '**/*.md'

# All files in src/, any depth
git log -- 'src/**'

# All .py files in specific directory
git log -- 'app/*.py'
```

⚠️ **Note:** Quote glob patterns to prevent shell expansion.

### Pathspec Magic Signatures

Git supports advanced pathspec features:

#### Top-level Matching

```bash
# Match only in top level, not subdirectories
git log -- ':(top)*.txt'
```

#### Case-Insensitive Matching

```bash
# Case-insensitive
git log -- ':(icase)readme.md'
```

#### Exclusion Patterns

```bash
# All .js files except tests
git log -- '*.js' ':(exclude)**/*test.js'

# Everything except node_modules
git log -- . ':(exclude)node_modules/**'

# Multiple exclusions
git log -- '**/*.py' ':(exclude)**/*test*.py' ':(exclude)build/**'
```

#### Literal Paths (No Glob)

```bash
# Treat as literal path, not a glob
git log -- ':(literal)file[1].txt'
```

### Following File Renames

```bash
# Follow file through renames
git log --follow -- oldname.txt

# Show renames explicitly
git log --follow --name-status -- current-name.txt

# Full history with renames
git log --follow --all -- file.txt
```

### Combining Multiple Constraints

```bash
# File changes by specific author
git log --author="Alice" -- src/

# Recent changes to specific files
git log --since="1 week ago" -- '**/*.js'

# Merges affecting a directory
git log --merges -- src/components/

# Changes not yet in main for specific directory
git log main..feature -- docs/
```

### Diff with Path Limiting

```bash
# Diff specific file between branches
git diff main..feature -- src/app.js

# Changes to directory since last tag
git diff v1.0.0..HEAD -- src/

# Only show diffs for tests
git diff HEAD~5..HEAD -- '**/*test.js'
```

### Advanced Path Examples

#### Find who changed a line

```bash
# Using git log with -L (line range)
git log -L 10,20:path/to/file.txt

# Function history
git log -L :functionName:path/to/file.py
```

#### Find when a file was added

```bash
# First commit introducing the file
git log --diff-filter=A -- path/to/file.txt

# Last commit deleting the file
git log --diff-filter=D -- path/to/file.txt
```

#### Find commits that modified certain files

```bash
# Only show commits that changed these files
git log --oneline -- file1.txt file2.txt

# Commits that modified file in a range
git log abc123..HEAD -- important.txt
```

#### Search for code changes

```bash
# Commits adding or removing specific string
git log -S"search_term" -- '**/*.js'

# Commits matching regex in code
git log -G"regex_pattern" -- src/

# Commits with message containing word, affecting path
git log --grep="bug" -- src/buggy/
```

### Practical Workflow Examples

#### Review changes to documentation

```bash
git log --since="1 month ago" -- docs/ README.md
```

#### Find when configuration changed

```bash
git log --follow -- config/app.yml
```

#### See who modified a critical file

```bash
git log --oneline --author=".*" -- src/security/auth.js
```

#### Check if a file exists in a branch

```bash
git log branch -- path/to/file.txt | head -1
```

#### Compare file versions across branches

```bash
git diff main..feature -- package.json
```

---

## Practical Examples

This section demonstrates real-world scenarios and how to solve them with Git's range notation.

### Scenario 1: Finding Unique Branch Commits

**Question:** "What commits are in my feature branch that aren't in main?"

```bash
# Basic approach
git log main..feature

# More detailed with patches
git log -p main..feature

# One-line summary
git log --oneline main..feature

# With graph visualization
git log --graph --oneline main..feature

# Count the commits
git rev-list --count main..feature
```

**Expected output:**
```
3 commits

a1b2c3d Add user authentication
b2c3d4e Update login form UI
c3d4e5f Add password validation
```

### Scenario 2: Finding Commits in Branch A but Not in Branch B

**Question:** "What's in develop that hasn't been merged to main?"

```bash
# All commits in develop not in main
git log main..develop

# Reverse - what's in main not in develop
git log develop..main

# Both directions (symmetric difference)
git log --left-right main...develop

# Formatted to show which side
git log --left-right --oneline main...develop
```

**Expected output:**
```
< a1b2c3d Commit only in main
< b2c3d4e Another commit only in main
> c3d4e5f Commit only in develop
> d4e5f6g Another commit only in develop
```

### Scenario 3: Reviewing Changes Before Push

**Question:** "What am I about to push?"

```bash
# Commits not yet pushed
git log origin/main..HEAD

# Shorthand
git log @{u}..@

# With diff statistics
git log --stat @{u}..@

# Show actual changes
git diff @{u}..@

# One-line summary
git log --oneline @{u}..@
```

**Expected output:**
```
3 commits ahead of origin/main:

f1e2d3c Fix memory leak in parser
e2d3c4b Update dependencies
d3c4b5a Add unit tests for validator
```

### Scenario 4: Checking Incoming Changes

**Question:** "What will I get if I pull?"

```bash
# First, fetch latest
git fetch

# See incoming commits
git log HEAD..origin/main

# Shorthand
git log ..@{u}

# With statistics
git log --stat ..@{u}

# Compare your code to incoming
git diff HEAD..origin/main
```

**Expected output:**
```
2 commits in origin/main not yet in your branch:

z9y8x7w Hotfix for production bug
y8x7w6v Update version to 2.1.4
```

### Scenario 5: Finding Merge Base

**Question:** "Where did my branch diverge from main?"

```bash
# Find merge base
git merge-base main feature

# Show the merge base commit
git show $(git merge-base main feature)

# Log since divergence
git log $(git merge-base main feature)..feature

# Diff since divergence
git diff $(git merge-base main feature)..feature

# Using three-dot notation (equivalent)
git diff main...feature
```

**Expected output:**
```
Merge base: a1b2c3d

This is where 'feature' branched off from 'main'
```

### Scenario 6: Reviewing Feature Branch Changes

**Question:** "What changes did my feature branch introduce?"

```bash
# Best approach - show only feature changes
git diff main...feature

# This is better than two-dot notation because:
# Two-dot includes changes from both branches
git diff main..feature    # includes main's changes too

# Three-dot shows only feature's changes
git diff main...feature   # only feature's changes

# Log format
git log --oneline main...feature

# With file stats
git log --stat main...feature
```

### Scenario 7: Selecting from Multiple Branches

**Question:** "Show commits in branch1 OR branch2, but not in main"

```bash
# Include both branches, exclude main
git log ^main branch1 branch2

# Equivalent using --not
git log branch1 branch2 --not main

# With visual graph
git log --graph --oneline ^main branch1 branch2

# Count commits
git rev-list --count ^main branch1 branch2
```

**Expected output:**
```
Commits in branch1 or branch2 but not in main:

From branch1:
a1b2c3d Feature 1 commit A
b2c3d4e Feature 1 commit B

From branch2:
c3d4e5f Feature 2 commit A
d4e5f6g Feature 2 commit B
```

### Scenario 8: Finding What Was Merged

**Question:** "What commits were included in this merge?"

```bash
# If you're at a merge commit M
git log M^1..M^2

# Show what the feature branch had
git log --oneline HEAD^1..HEAD^2

# Full details
git log -p HEAD^1..HEAD^2

# Summary statistics
git diff --stat HEAD^1..HEAD^2
```

**Expected output:**
```
Merged commits from feature branch:

f1e2d3c Add new API endpoint
e2d3c4b Add tests for endpoint
d3c4b5a Update documentation

Files changed: 12
Insertions: 234
Deletions: 45
```

### Scenario 9: Time-Based Branch Comparison

**Question:** "What changed in main in the last week while I was working?"

```bash
# Changes to main in last week
git log --since="1 week ago" main

# Changes excluding your branch
git log --since="1 week ago" main ^feature

# If you want to see what you missed
git fetch
git log HEAD..origin/main --since="1 week ago"
```

### Scenario 10: Finding Common Ancestors

**Question:** "What do these branches have in common?"

```bash
# Find merge base of two branches
git merge-base branch1 branch2

# Find merge base of multiple branches
git merge-base --octopus branch1 branch2 branch3

# Show commits common to all branches
git log branch1 branch2 --not $(git merge-base branch1 branch2)^
```

### Scenario 11: Cherry-Pick Range

**Question:** "How do I cherry-pick a range of commits?"

```bash
# Cherry-pick a range (exclusive start, inclusive end)
git cherry-pick A..B    # Excludes A, includes B

# Cherry-pick including start commit
git cherry-pick A^..B

# Cherry-pick everything from branch not in current branch
git cherry-pick ..branch-name

# Dry run - see what would be cherry-picked
git log --oneline A..B
```

### Scenario 12: Rebase Range Inspection

**Question:** "What commits will be replayed during rebase?"

```bash
# See what would be rebased
git log --oneline base-branch..current-branch

# What would be rebased onto main
git log --oneline main..@

# Interactive rebase preview
git log --oneline main..@ --reverse
```

### Scenario 13: Finding Lost Commits

**Question:** "I reset my branch, how do I find the lost commits?"

```bash
# View reflog
git reflog

# Show where HEAD was 3 moves ago
git show HEAD@{3}

# See lost commits
git log HEAD@{1}..HEAD@{0}

# Recover by resetting to old position
git reset --hard HEAD@{1}
```

### Scenario 14: Complex Multi-Branch Analysis

**Question:** "Show me all feature branches' changes not in main or develop"

```bash
# All feature branches except main and develop
git log --branches=feature/* ^main ^develop

# With graph
git log --graph --oneline --branches=feature/* ^main ^develop

# Count by branch
for branch in $(git branch --list 'feature/*' | sed 's/\*//' | xargs); do
  count=$(git rev-list --count main..$branch)
  echo "$branch: $count commits ahead"
done
```

### Scenario 15: Release Range Analysis

**Question:** "What's between v1.0 and v2.0?"

```bash
# All commits in the range
git log v1.0..v2.0

# One-liner per commit
git log --oneline v1.0..v2.0

# With authors
git log --format="%h %an %s" v1.0..v2.0

# Generate changelog
git log --pretty=format:"- %s (%h)" v1.0..v2.0

# Statistics
git diff --stat v1.0..v2.0
```

**Expected output:**
```
Release v2.0 includes 47 commits:

- Add dark mode support (a1b2c3d)
- Improve performance by 40% (b2c3d4e)
- Fix security vulnerability (c3d4e5f)
...

Files changed: 156 files
Insertions: 3,245
Deletions: 1,876
```

---

## Edge Cases and Gotchas

Understanding Git's range notation edge cases helps avoid confusion and unexpected behavior.

### Empty Ranges

#### When A..B is Empty

```bash
# If B is an ancestor of A (or A == B)
git log A..B    # Shows nothing

# Example: main is at B, feature is at A (ahead)
git log feature..main    # Empty (main has nothing feature doesn't)
git log main..feature    # Shows commits (feature is ahead)
```

**Check if range is empty:**
```bash
# Returns 0 if empty
git rev-list --count main..feature

# Exit code 0 if empty, 1 if not
git rev-list --quiet main..feature && echo "empty" || echo "not empty"
```

#### When Branches Are Identical

```bash
# If main and feature point to same commit
git log main..feature    # Empty
git log main...feature   # Empty
git diff main..feature   # No output
```

### Ambiguous References

Git tries to resolve ambiguous names as:
1. Tag
2. Branch
3. Remote branch
4. Partial SHA

#### Ambiguity Example

```bash
# If you have both a branch and tag named "release"
git log release    # Which one?

# Be explicit
git log refs/heads/release    # The branch
git log refs/tags/release     # The tag

# Or use full refs
git log refs/remotes/origin/release
```

#### File vs Branch Name Conflict

```bash
# If you have a branch named "main" and a file named "main"
git log main           # Shows the branch
git log -- main        # Shows history of the file

# Ambiguous without --
git checkout main      # Checks out branch
git checkout -- main   # Restores file
```

**Best practice:** Always use `--` when dealing with files:
```bash
git log -- path/to/file
git diff branch1 branch2 -- file.txt
```

### Parent References on Non-Merge Commits

```bash
# HEAD^2 only exists if HEAD is a merge commit
git show HEAD^2    # Error if HEAD is not a merge

# Safe check
git rev-parse --verify HEAD^2 2>/dev/null
if [ $? -eq 0 ]; then
  echo "HEAD is a merge commit"
else
  echo "HEAD is not a merge commit"
fi
```

### Reflog Edge Cases

#### Reflog Expiration

```bash
# Reflog entries expire (default: 90 days for unreachable, 30 days for reachable)
git show HEAD@{100}    # May not exist if too old

# Check reflog
git reflog show --all

# Prevent expiration (not recommended)
git config gc.reflogExpire never
```

#### Reflog vs Branch Reflog

```bash
# HEAD's reflog (all checkouts and commits)
git reflog

# Branch's reflog (only when branch moved)
git reflog show main

# These can differ significantly
git show HEAD@{1}     # Your last position
git show main@{1}     # Last time main changed
```

### Date Specification Gotchas

#### Time Zone Handling

```bash
# Git uses commit timezone by default
git log --since="2024-01-01"    # Midnight in commit's timezone

# Specify timezone
git log --since="2024-01-01 00:00:00 -0800"
```

#### Author Date vs Commit Date

```bash
# These can differ after rebase or cherry-pick
git log --since="1 week ago"    # Uses author date
git log --since="1 week ago" --date-order    # Still author date

# To see the difference
git log --format="%h %ad %cd" --date=iso
```

#### "Since" is Inclusive

```bash
# Includes commits from exactly "1 week ago"
git log --since="1 week ago"

# To exclude exact boundary
git log --since="1 week ago 1 second ago"
```

### Performance Considerations

#### Large Ranges

```bash
# Can be slow on large repositories
git log --all    # Scans all refs

# More efficient - limit scope
git log --branches --since="1 month ago"

# Much faster with --first-parent
git log --first-parent main
```

#### Complex Exclusions

```bash
# Can be expensive
git log --all ^main ^develop ^release/*

# Consider limiting
git log --all ^main ^develop ^release/* --since="1 month ago"
```

#### Path Filtering Performance

```bash
# Slow on large repositories
git log --all -- some-file.txt

# Faster with limited range
git log main -- some-file.txt

# Much faster with time limit
git log --since="6 months ago" -- some-file.txt
```

### Three-Dot Notation Context Dependency

The three-dot notation behaves differently based on command:

```bash
# These are DIFFERENT
git log main...feature     # Symmetric difference (all unique commits)
git diff main...feature    # Changes from merge-base to feature

# This can be confusing
# log: shows commits from BOTH sides
# diff: shows changes from ONE side
```

**Example:**
```
    A---B---C    (main)
         \
          D---E  (feature)

git log main...feature     # Shows: C, D, E
git diff main...feature    # Shows: B→E (only feature changes)
```

### Merge Commit Parent Order

Parent order matters and isn't always obvious:

```bash
# When you do: git merge feature
# First parent: your current branch (main)
# Second parent: what you merged (feature)

# But if you do: git merge main (while on feature)
# First parent: feature
# Second parent: main

# The order depends on which branch you were on!
```

**Check parent order:**
```bash
git log --pretty=format:"%h %p" -1    # Shows commit and its parents
```

### Double-Caret Gotcha

```bash
# These are different in some contexts
git show HEAD^    # First parent
git show HEAD\^   # May be needed in some shells

# Especially in scripts
git show "HEAD^"  # Safer
```

### Upstream Tracking Not Set

```bash
# Error if no upstream
git log @{u}..@    # Error: no upstream configured

# Check if upstream exists
git rev-parse --abbrev-ref @{u} 2>/dev/null
```

### Octopus Merges

Rarely used, but can exist:

```bash
# Merge with 3+ parents
git show HEAD^3    # Third parent (if it exists)

# Check number of parents
git rev-list --parents -n1 HEAD | wc -w
# Result: N+1 where N is number of parents
```

### Detached HEAD State

```bash
# Some operations behave differently
git log @{-1}    # Works differently in detached HEAD

# Reflog may not capture all movements
git reflog show HEAD    # vs
git reflog show branch-name
```

### Escaping in Different Shells

```bash
# Bash
git show HEAD^2

# Zsh (caret is special)
git show HEAD\^2
# Or
git show "HEAD^2"

# Windows CMD
git show HEAD^^2    # Sometimes needs doubling
```

### Range Notation with Diff

```bash
# Two-dot: both sides are treated as points
git diff A..B    # Effectively: git diff A B

# Three-dot: uses merge base
git diff A...B   # Effectively: git diff $(git merge-base A B) B

# This is opposite of intuition from git log!
```

**Summary Table:**

| Command | A..B | A...B |
|---------|------|-------|
| `git log` | Commits in B not in A | Commits in A or B, not both |
| `git diff` | Changes from A to B | Changes from merge-base to B |

### Commit Count Discrepancies

```bash
# These may differ
git log --oneline main..feature | wc -l    # Counts log lines
git rev-list --count main..feature         # Counts commits

# Why? Merges might appear differently
git log --no-merges --oneline main..feature | wc -l
```

### Best Practices Summary

1. **Always use `--` when specifying files**
2. **Quote special characters** in shells (especially Zsh)
3. **Check if upstream exists** before using `@{u}`
4. **Be explicit with ambiguous names** (use full refs)
5. **Remember command context** for `...` notation
6. **Test complex ranges** with `git rev-list --count` first
7. **Use `--first-parent`** for performance on large repos
8. **Be aware of timezone** in date specifications
9. **Check merge commit parents** with `git log --pretty=format:"%h %p"`
10. **Remember reflog expires** - don't rely on old entries

---

## Conclusion

Git's range notation and revision specification system is incredibly powerful once understood. Key takeaways:

- **Basic references** (SHA, HEAD, branches, tags) form the foundation
- **Relative references** (^ and ~) navigate commit history
- **Two-dot notation** (A..B) shows commits in B but not A
- **Three-dot notation** (A...B) behaves differently in log vs diff
- **Multiple parents** (^1, ^2) navigate merge commits
- **Exclusion patterns** (^A, --not) enable complex selections
- **Special notations** (@, @{-1}, @{u}) provide convenient shortcuts
- **Date-based ranges** filter by time
- **Path limiting** (--) constrains operations to specific files
- **Context matters** - the same notation behaves differently across commands

Mastering these concepts enables efficient Git workflows, powerful queries, and precise commit selection for any operation.

---

## Quick Reference Card

```bash
# Basic References
HEAD            # Current commit
HEAD~1          # Parent
HEAD~2          # Grandparent
HEAD^           # First parent
HEAD^2          # Second parent (merges)
branch-name     # Tip of branch
abc123          # Commit SHA

# Ranges
A..B            # In B, not in A
A...B           # In A or B, not both (log) / changes from merge-base to B (diff)
^A B            # In B, not in A
A B ^C          # In A or B, not in C

# Special
@               # HEAD
@{-1}           # Previous branch
@{u}            # Upstream
@{push}         # Push destination
HEAD@{1}        # Reflog: 1 move ago
HEAD@{yesterday} # Reflog: by date

# Combining
main..feature -- file.txt    # Range with file
git log ^main ^develop feature    # Complex exclusion
git log --since="1 week ago" @{u}..@    # Time + range
```

Use `git help revisions` for the official documentation on revision specifications.
