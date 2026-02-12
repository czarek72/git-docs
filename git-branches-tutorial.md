# Git Branches Tutorial: Local vs Remote Branches Deep Dive

## Table of Contents
- [Introduction](#introduction)
- [What Is a Branch Under the Hood?](#what-is-a-branch-under-the-hood)
- [Three Types of Branches](#three-types-of-branches)
- [Local Branches](#local-branches)
- [Remote-Tracking Branches](#remote-tracking-branches)
- [The Complete Mental Model](#the-complete-mental-model)
- [Creating Branches](#creating-branches)
- [Fetching Changes: Network Protocol Deep Dive](#fetching-changes-network-protocol-deep-dive)
- [Pulling Changes: The Two-Phase Operation](#pulling-changes-the-two-phase-operation)
- [Tracking Relationships: Upstream Configuration](#tracking-relationships-upstream-configuration)
- [Pushing Changes](#pushing-changes)
- [Deleting Branches](#deleting-branches)
- [Common Workflows](#common-workflows)
- [Troubleshooting Common Issues](#troubleshooting-common-issues)
- [Quick Reference](#quick-reference)

## Introduction

Git branches are one of the most powerful yet misunderstood features of Git. The confusion stems from not understanding the difference between **local branches**, **remote-tracking branches**, and **remote branches**, and how they synchronize through network operations.

### Key Concepts We'll Cover
- What branches actually are (simple text files pointing to commits)
- The critical differences between the three types of branches
- What happens on the wire during fetch/pull/push operations
- Where Git stores objects, refs, and tracking information
- Mental models for reasoning about distributed branches

## What Is a Branch Under the Hood?

Before we dive into local vs remote, let's understand what a branch fundamentally is.

### A Branch Is Just a Text File

A branch is simply a **41-byte text file** containing a 40-character SHA-1 hash (or 65-byte for SHA-256) pointing to a commit, followed by a newline.

```bash
# View the main branch file
cat .git/refs/heads/main
# Output: 9c1f3d2e5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d

# That's it! Just a commit hash pointing to a commit object
```

### Branch Storage Location
```
.git/
├── refs/
│   ├── heads/              # Local branches
│   │   ├── main
│   │   ├── feature-login
│   │   └── bugfix-auth
│   └── remotes/            # Remote-tracking branches
│       └── origin/
│           ├── main
│           ├── feature-login
│           └── develop
├── objects/                # All commits, trees, blobs
│   ├── 9c/
│   │   └── 1f3d2e...       # Commit objects
│   └── pack/               # Packfiles from fetches
│       ├── pack-*.pack     # Compressed objects
│       └── pack-*.idx      # Pack indexes
└── config                  # Contains remote URLs and refspecs
```

### Creating a Branch Is Cheap

```bash
# Create a branch
git branch my-feature

# This creates: .git/refs/heads/my-feature
# Containing: Current commit's SHA-1
# No objects copied, no data duplicated
```

### Branches Are Movable Pointers

When you commit, Git updates the current branch file to point to the new commit:

```bash
# Before commit
cat .git/refs/heads/main
# 8ab686eafeb1f44702acb4f8d8e2e5a5f7e5b3c1

# Make a commit
git commit -m "Add feature"

# After commit
cat .git/refs/heads/main
# 9c1f3d2e5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d  ← Updated to new commit
```

## Three Types of Branches

Understanding Git branches requires recognizing that there are **three distinct types** with different purposes and storage locations:

### 1. Local Branches
- **Location:** `.git/refs/heads/`
- **What they are:** Your working branches
- **Modified by:** Your commits, merges, resets
- **Examples:** `main`, `feature-login`, `bugfix-123`

### 2. Remote-Tracking Branches
- **Location:** `.git/refs/remotes/origin/`
- **What they are:** Read-only **local copies** of remote branches
- **Modified by:** `git fetch` and `git pull`
- **Examples:** `origin/main`, `origin/develop`
- **Critical:** These are LOCAL files, not on the server!

### 3. Remote Branches
- **Location:** On the remote server (GitHub, GitLab, etc.)
- **Modified by:** `git push` (by you or collaborators)
- **Examples:** `main` on the server, `develop` on the server

### Visual Model

```
┌─────────────────────────────────────────────────────────┐
│                   YOUR COMPUTER                         │
│                                                         │
│  Local Branches          Remote-Tracking Branches      │
│  .git/refs/heads/        .git/refs/remotes/origin/     │
│                                                         │
│  main ────────────┐      origin/main                   │
│  feature-x        │      origin/develop                │
│                   │                                     │
│  You work here ───┘      Read-only cache               │
│  (git commit)            (updated by git fetch)        │
└─────────────────────────────────────────────────────────┘
                           │
                           │ Network: fetch/pull/push
                           │
┌─────────────────────────▼─────────────────────────────┐
│              REMOTE SERVER (GitHub, etc.)             │
│                                                       │
│  Remote Branches: refs/heads/*                        │
│  main, develop, feature-y                             │
│                                                       │
│  Shared with all collaborators                        │
└───────────────────────────────────────────────────────┘
```

**Key Understanding:** `origin/main` is a LOCAL file on your computer that caches the state of the remote's `main` branch from the last fetch.

## Local Branches

Local branches are where you do your work. They exist only on your computer until you explicitly push them.

### Viewing Local Branches

```bash
# List all local branches
git branch
# Output:
# * main
#   feature-login
#   bugfix-auth

# The * indicates your current branch (where HEAD points)

# View with commit info
git branch -v
# * main          9c1f3d2 Add user authentication
#   feature-login 5d4e3f2 Implement login form

# Show tracking relationships
git branch -vv
# * main          9c1f3d2 [origin/main] Add auth
#   feature-login 5d4e3f2 [origin/feature-login: ahead 2] Implement login
```

### Creating and Switching Branches

```bash
# Create branch (doesn't switch)
git branch feature-new

# Create and switch (modern command)
git switch -c feature-new

# Create from specific commit
git branch feature-new abc1234

# Create from another branch
git switch -c feature-new develop
```

### Working with Local Branches

```bash
# Switch branches
git switch main

# See current branch
git branch --show-current

# Rename current branch
git branch -m new-name

# Rename different branch
git branch -m old-name new-name

# Delete local branch (safe - only if merged)
git branch -d feature-merged

# Force delete local branch
git branch -D feature-abandoned
```

### Local Branch Storage

Each local branch is stored as a simple text file:

```bash
# List local branch files
ls .git/refs/heads/
# main  feature-login  bugfix-auth

# View branch content
cat .git/refs/heads/main
# 9c1f3d2e5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d
```

## Remote-Tracking Branches

Remote-tracking branches are the most misunderstood concept. They are **local files** that cache the state of remote branches.

### What They Really Are

- **Local files** on your computer in `.git/refs/remotes/origin/`
- **Read-only** references (you can't commit to them directly)
- **Snapshots** of remote branches from your last fetch
- **Updated only** by `git fetch` or `git pull`

### Viewing Remote-Tracking Branches

```bash
# List all remote-tracking branches
git branch -r
#   origin/main
#   origin/develop
#   origin/feature-x

# List all branches (local + remote-tracking)
git branch -a
# * main                    # Local
#   feature-login           # Local
#   remotes/origin/main     # Remote-tracking
#   remotes/origin/develop  # Remote-tracking
```

### Storage and Content

```bash
# View remote-tracking branch files
ls .git/refs/remotes/origin/
# main  develop  feature-x

# View content
cat .git/refs/remotes/origin/main
# 7a5b3c1e9d8f6e4a2b0c3d5e7f9a1b3c5d7e9f1a
# ↑ This is a LOCAL file with a commit hash
```

### You Cannot Commit to Remote-Tracking Branches

```bash
# Attempting to switch to remote-tracking branch
git switch origin/main
# fatal: a branch is expected, got remote branch 'origin/main'

# They are read-only references, not branches!

# To work on it, create a local branch:
git switch -c my-main origin/main
```

### Purpose of Remote-Tracking Branches

**1. Track synchronization status:**
```bash
git status
# Your branch is ahead of 'origin/main' by 2 commits.
# ↑ Compares local main with origin/main (local file)
```

**2. Enable inspection before merging:**
```bash
git fetch origin
git log main..origin/main  # What's on remote
git diff main origin/main   # Actual differences
```

**3. Establish tracking relationships:**
```bash
git switch -c feature --track origin/feature
# Sets up: local feature tracks origin/feature

git branch -vv
# feature  abc1234 [origin/feature] Latest commit
```

## The Complete Mental Model

Understanding how all three branch types interact requires a three-layer model:

```
┌───────────────────────────────────────────────────────────────┐
│                    LAYER 1: YOUR WORK                         │
│                     Local Branches                            │
│                 .git/refs/heads/*                             │
│                                                               │
│  main          feature-x                                      │
│  │             │                                              │
│  ├─ A ─ B ─ C  ├─ A ─ B ─ X ─ Y                              │
│                                                               │
│  Modified by: git commit, git merge, git rebase               │
│                                                               │
└───────────────────────────┬───────────────────────────────────┘
                            │
                     git fetch / git push
                            │
┌───────────────────────────▼───────────────────────────────────┐
│                    LAYER 2: LOCAL CACHE                       │
│                 Remote-Tracking Branches                      │
│              .git/refs/remotes/origin/*                       │
│                                                               │
│  origin/main   origin/feature-x                               │
│  │             │                                              │
│  ├─ A ─ B ─ C  ├─ A ─ B ─ X                                  │
│                                                               │
│  Modified by: git fetch (updates from server)                │
│  Read-only: Can't commit directly                             │
│                                                               │
└───────────────────────────┬───────────────────────────────────┘
                            │
                     Network (HTTP/SSH)
                            │
┌───────────────────────────▼───────────────────────────────────┐
│                    LAYER 3: REMOTE SERVER                     │
│                     Remote Branches                           │
│                refs/heads/* (on server)                       │
│                                                               │
│  main          feature-x                                      │
│  │             │                                              │
│  ├─ A ─ B ─ C  ├─ A ─ B ─ X                                  │
│                                                               │
│  Modified by: git push (by anyone)                            │
│  Shared: Visible to all collaborators                         │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

### Synchronization States

Your local branch relative to remote-tracking branch:

**Up to Date:**
```
Local:  A ─ B ─ C
Remote: A ─ B ─ C
Status: "Your branch is up to date"
```

**Ahead:**
```
Local:  A ─ B ─ C ─ D ─ E
Remote: A ─ B ─ C
Status: "Your branch is ahead by 2 commits"
Action: git push
```

**Behind:**
```
Local:  A ─ B ─ C
Remote: A ─ B ─ C ─ D ─ E
Status: "Your branch is behind by 2 commits"
Action: git pull
```

**Diverged:**
```
Local:  A ─ B ─ C ─ L1 ─ L2
Remote: A ─ B ─ C ─ R1 ─ R2
Status: "Your branch and 'origin/main' have diverged"
Action: git pull (creates merge) or git pull --rebase
```

## Creating Branches

### Creating Local Branches

```bash
# Create branch (doesn't switch)
git branch feature-new

# Create and switch
git switch -c feature-new

# Create from specific commit
git branch feature-new abc1234
git switch -c feature-new abc1234

# Create from another branch
git switch -c feature-new develop

# Create from relative reference
git switch -c feature-new HEAD~3
```

### Creating from Remote-Tracking Branch

```bash
# Automatic tracking setup
git switch feature-x
# If feature-x exists as origin/feature-x but not locally:
# 1. Creates local feature-x
# 2. Points to same commit as origin/feature-x
# 3. Sets up tracking automatically
# 4. Switches to new branch

# Explicit tracking with different name
git switch -c my-feature origin/feature-x

# Explicit tracking with same name
git switch -c feature-x --track origin/feature-x

# Verify tracking
git branch -vv
# feature-x abc1234 [origin/feature-x] Latest commit
```

### Creating Remote Branches (via Push)

You cannot directly create remote branches. You create local then push:

```bash
# Create local branch
git switch -c feature-new
git commit -m "Initial work"

# Push to create remote branch (and set tracking)
git push -u origin feature-new

# Result:
# Local: .git/refs/heads/feature-new
# Remote-tracking: .git/refs/remotes/origin/feature-new
# Remote: refs/heads/feature-new (on server)
```

### Push to Different Remote Branch Name

```bash
# Local: feature-new
# Remote: awesome-feature
git push origin feature-new:awesome-feature

# Creates origin/awesome-feature locally
# Creates awesome-feature on remote
```

### Viewing All Branches

```bash
# Local only
git branch

# Remote-tracking only
git branch -r

# All branches
git branch -a

# With commit info and tracking
git branch -vv
# * main        9c1f3d2 [origin/main] Latest
#   feature-x   5d4e3f2 [origin/feature-x: ahead 2] WIP
```

## Fetching Changes: Network Protocol Deep Dive

`git fetch` downloads commits and refs from a remote repository. Understanding what happens on the wire and where data resides is crucial.

### What Git Fetch Actually Does

```bash
git fetch origin
```

**Phase 1: Reference Discovery**
1. Git connects to remote (via HTTP/SSH)
2. Requests list of refs: `GET /info/refs?service=git-upload-pack`
3. Server responds with all refs and their commit SHAs:
   ```
   9c1f3d2e... refs/heads/main
   5d4e3f2a... refs/heads/develop
   8ab686ea... refs/heads/feature-x
   ```

**Phase 2: Object Negotiation**
1. Git compares remote refs with local refs
2. Determines which commits it needs
3. Sends "have" lines for commits it already has
4. Server responds with commits you're missing

**Phase 3: Packfile Transfer**
1. Server creates a packfile containing:
   - Missing commit objects
   - Missing tree objects  
   - Missing blob objects (file contents)
2. Packfile is compressed and sent over wire
3. Git receives and verifies packfile

**Phase 4: Local Storage**
1. Packfile saved to `.git/objects/pack/pack-*.pack`
2. Index created: `.git/objects/pack/pack-*.idx`
3. Remote-tracking refs updated:
   ```
   .git/refs/remotes/origin/main ← new SHA
   .git/refs/remotes/origin/develop ← new SHA
   ```
4. **Local branches: Unchanged**
5. **Working directory: Unchanged**

### Where Objects Reside After Fetch

```
.git/
├── objects/
│   ├── pack/
│   │   ├── pack-abc123.pack    # Compressed objects from fetch
│   │   └── pack-abc123.idx     # Index for quick lookups
│   └── 9c/
│       └── 1f3d2e...           # Individual commit objects
├── refs/
│   ├── heads/
│   │   └── main                # LOCAL - Unchanged by fetch
│   └── remotes/
│       └── origin/
│           ├── main            # UPDATED - Points to new commit
│           └── develop         # UPDATED - Points to new commit
└── FETCH_HEAD                  # Temporary - Last fetched refs
```

### Fetch Operations

```bash
# Fetch from default remote (origin)
git fetch

# Fetch from specific remote
git fetch origin

# Fetch specific branch only
git fetch origin main
# Only updates origin/main

# Fetch all remotes
git fetch --all

# Fetch and prune deleted branches
git fetch --prune
```

### Understanding Refspecs

Refspecs define what refs to fetch and where to store them locally:

```bash
# View fetch refspec
git config remote.origin.fetch
# +refs/heads/*:refs/remotes/origin/*

# Explanation:
# +                  = Allow non-fast-forward updates
# refs/heads/*       = All branches on remote
# refs/remotes/origin/* = Store as remote-tracking branches
```

**Custom refspec example:**
```bash
# Only fetch main branch
git config remote.origin.fetch "refs/heads/main:refs/remotes/origin/main"

# Fetch multiple specific branches
git config --add remote.origin.fetch "refs/heads/develop:refs/remotes/origin/develop"
```

### Network Traffic Analysis

**What's actually sent over the wire:**

```bash
# Enable trace for detailed protocol info
GIT_TRACE_PACKET=1 git fetch origin 2>&1 | grep "git<"

# Example output shows:
# - Reference advertisement (refs and SHAs)
# - Negotiation (have/want protocol)
# - Packfile transfer (actual objects)
```

**Fetch efficiency:**
- Git only sends missing objects (delta compression)
- If you're up to date: Only refs exchanged (~few KB)
- If behind: Packfile with new objects transferred

### After Fetching: Inspecting Changes

```bash
# Fetch latest
git fetch origin

# See what changed
git log main..origin/main
# Shows commits in origin/main not in main

# View differences
git diff main origin/main

# Graphical view
git log --oneline --graph --all

# Check status
git status
# On branch main
# Your branch is behind 'origin/main' by 3 commits.
```

### Fetch vs Not Fetching

**Without fetch:**
```bash
git branch -r
# origin/main (points to commit from days ago)

# Your knowledge is outdated
# Remote-tracking branches are stale
# git status shows incorrect sync state
```

**With fetch:**
```bash
git fetch origin

git branch -r  
# origin/main (points to latest commit)

# Your knowledge is current
# Remote-tracking branches are fresh
# git status shows accurate sync state
```

### Automatic Pruning Configuration

```bash
# Enable automatic pruning
git config fetch.prune true

# Now git fetch removes stale remote-tracking branches
# (branches deleted on remote)

# Manual prune check
git remote prune origin --dry-run
# Would prune origin/old-feature

# Actually prune
git remote prune origin
```

### FETCH_HEAD Special Reference

After fetch, Git creates `.git/FETCH_HEAD`:

```bash
cat .git/FETCH_HEAD
# 9c1f3d2e... branch 'main' of https://github.com/user/repo
# 5d4e3f2a... branch 'develop' of https://github.com/user/repo

# FETCH_HEAD points to the first branch fetched
# Used by git pull for merging
```

## Pulling Changes: The Two-Phase Operation

`git pull` is a composite command that combines fetch and merge (or rebase). Understanding both phases is essential.

### The Two Phases of Pull

```bash
git pull origin main

# Phase 1: FETCH
# ├─ Network protocol (reference discovery + object negotiation)
# ├─ Download packfile with missing objects
# ├─ Store in .git/objects/pack/
# └─ Update .git/refs/remotes/origin/main

# Phase 2: MERGE (or REBASE)
# ├─ Read remote-tracking ref: .git/refs/remotes/origin/main
# ├─ Find merge base between local and remote-tracking
# ├─ Perform three-way merge
# ├─ Update local branch: .git/refs/heads/main
# └─ Update working directory with merged content
```

### What Happens During Pull

**Step 1: Fetch Phase**
```
1. Git contacts remote via HTTP/SSH
2. Downloads new commits to .git/objects/pack/
3. Updates .git/refs/remotes/origin/main
4. Creates .git/FETCH_HEAD with fetched refs
```

**Step 2: Merge/Rebase Phase**
```
1. Reads FETCH_HEAD or origin/main
2. If merge:
   - Finds common ancestor
   - Creates merge commit
   - Updates .git/refs/heads/main
   - Checks out merged tree to working directory
3. If rebase:
   - Finds commits unique to local branch
   - Reapplies them on top of origin/main
   - Updates .git/refs/heads/main
   - Updates working directory
```

### Pull Commands

```bash
# Pull with merge (default)
git pull origin main
# Equivalent to:
#   git fetch origin
#   git merge origin/main

# Pull with rebase
git pull --rebase origin main
# Equivalent to:
#   git fetch origin
#   git rebase origin/main

# Pull from tracked upstream
git pull
# Uses tracking configuration from .git/config

# Fast-forward only
git pull --ff-only origin main
# Fails if merge would be required
```

### Tracking Configuration for Pull

When you set upstream tracking, pull knows what to fetch and merge:

```bash
# View tracking config
git config --get branch.main.remote
# origin

git config --get branch.main.merge
# refs/heads/main

# These configs tell git pull:
# - Fetch from "origin" remote
# - Merge refs/heads/main into current branch

# Set tracking manually
git branch --set-upstream-to=origin/main
```

### Pull with Merge (Default Strategy)

```bash
git pull origin main
```

**Before:**
```
Local:  A ─ B ─ C ─ L1 ─ L2
Remote: A ─ B ─ C ─ R1 ─ R2
```

**After:**
```
Local:  A ─ B ─ C ─ L1 ─ L2 ─┐
                              M (merge commit)
              ─ R1 ─ R2 ─────┘
```

**What's updated:**
- `.git/refs/heads/main` → points to merge commit M
- Working directory → updated with merged content
- `.git/index` → updated to reflect merged state

### Pull with Rebase

```bash
git pull --rebase origin main
```

**Before:**
```
Local:  A ─ B ─ C ─ L1 ─ L2
Remote: A ─ B ─ C ─ R1 ─ R2
```

**After:**
```
Local:  A ─ B ─ C ─ R1 ─ R2 ─ L1' ─ L2'
```

**What happens:**
1. Local commits L1, L2 are temporarily saved
2. Local branch reset to origin/main (R2)
3. L1, L2 replayed on top with new SHAs (L1', L2')
4. `.git/refs/heads/main` → points to L2'
5. Working directory updated to L2'

### Handling Conflicts During Pull

**Merge conflicts:**
```bash
git pull origin main
# Auto-merging file.txt
# CONFLICT (content): Merge conflict in file.txt

# Git updates:
# ├─ .git/MERGE_HEAD → points to fetched commit
# ├─ .git/index → contains all 3 stages (base, ours, theirs)
# └─ Working directory → files with conflict markers

# Resolve:
vim file.txt    # Fix conflicts
git add file.txt
git commit      # Completes merge
```

**Rebase conflicts:**
```bash
git pull --rebase origin main
# CONFLICT: Fix conflict and run 'git rebase --continue'

# Git state:
# ├─ .git/rebase-merge/ directory created
# ├─ Contains rebase state info
# └─ Working directory has conflicts

# Resolve:
vim file.txt
git add file.txt
git rebase --continue  # Apply next commit
```

### Pull Configuration

```bash
# Set default pull behavior
git config pull.rebase false  # Merge (default)
git config pull.rebase true   # Always rebase
git config pull.ff only       # Only fast-forward

# Per-branch configuration
git config branch.main.rebase true
```

### When Pull Fails

**Uncommitted changes:**
```bash
git pull origin main
# error: Your local changes would be overwritten

# Solutions:
# 1. Commit
git commit -am "WIP"
git pull

# 2. Stash
git stash
git pull
git stash pop

# 3. Discard (dangerous)
git restore .
git pull
```

### Pull vs Fetch + Merge

**Using pull (one command):**
```bash
git pull origin main
# Black box: Does fetch + merge automatically
# Less control over what happens
```

**Using fetch + merge (explicit):**
```bash
git fetch origin
git log main..origin/main  # Inspect what you're getting
git diff main origin/main  # See actual changes
git merge origin/main      # Merge when ready
# More control, better understanding
```

## Tracking Relationships: Upstream Configuration

Tracking relationships (also called "upstream" configuration) connect local branches to remote-tracking branches, enabling simplified commands.

### What is Branch Tracking?

When a local branch "tracks" a remote-tracking branch:
- `git pull` knows where to pull from
- `git push` knows where to push to
- `git status` shows ahead/behind information

### Where Tracking is Stored

Tracking configuration lives in `.git/config`:

```ini
[branch "main"]
    remote = origin
    merge = refs/heads/main

[branch "feature-x"]
    remote = origin
    merge = refs/heads/feature-x
```

**Interpretation:**
- `branch.main.remote = origin` → Use origin remote
- `branch.main.merge = refs/heads/main` → Pull from main branch

### Setting Up Tracking

```bash
# During branch creation
git switch -c feature-x --track origin/feature-x

# For existing branch
git branch --set-upstream-to=origin/main
# or shorter:
git branch -u origin/main

# During first push
git push -u origin feature-new
# Sets up tracking automatically

# Verify tracking
git branch -vv
# * main       9c1f3d2 [origin/main] Latest
#   feature-x  5d4e3f2 [origin/feature-x: ahead 2] WIP
```

### Understanding Refspecs

Refspecs define the mapping between remote and local refs:

```bash
# View refspec
git config --get remote.origin.fetch
# +refs/heads/*:refs/remotes/origin/*

# Breaking it down:
# +                     = Allow non-fast-forward updates
# refs/heads/*          = All remote branches
# :                     = Maps to
# refs/remotes/origin/* = Local remote-tracking branches
```

**Custom refspec examples:**
```bash
# Fetch only main
git config remote.origin.fetch "refs/heads/main:refs/remotes/origin/main"

# Fetch all branches to different location
git config remote.origin.fetch "refs/heads/*:refs/remotes/upstream/*"

# Add additional refspec (don't replace)
git config --add remote.origin.fetch "refs/heads/special:refs/remotes/origin/special"
```

### Push Refspecs

```bash
# View push refspec
git config --get remote.origin.push
# refs/heads/*:refs/heads/*

# Custom push refspec
git config remote.origin.push "refs/heads/main:refs/heads/production"
# Now 'git push origin' pushes main to production
```

### Tracking Status Information

```bash
# View tracking relationship
git branch -vv
# * main       abc1234 [origin/main: ahead 1, behind 2] Message
#   feature    def5678 [origin/feature: ahead 3] Message
#   local-only 123abc4 Message

# Explanation:
# [origin/main: ahead 1, behind 2]
#   ↑ Tracks origin/main
#   ↑ 1 local commit not pushed
#   ↑ 2 remote commits not pulled

# Detailed tracking info
git remote show origin
# * remote origin
#   Fetch URL: https://github.com/user/repo.git
#   Push  URL: https://github.com/user/repo.git
#   HEAD branch: main
#   Remote branches:
#     main    tracked
#     develop tracked
#   Local branch configured for 'git pull':
#     main merges with remote main
#   Local ref configured for 'git push':
#     main pushes to main (up to date)
```

### Benefits of Tracking

**Without tracking:**
```bash
# Must specify remote and branch every time
git pull origin main
git push origin main
git status  # Only shows local status
```

**With tracking:**
```bash
# Simple commands
git pull
git push
git status
# Your branch is ahead of 'origin/main' by 2 commits.
```

### Changing Tracking

```bash
# Switch tracking to different remote-tracking branch
git branch -u origin/develop

# Remove tracking
git branch --unset-upstream

# Verify change
git branch -vv
# * main abc1234 [origin/develop] Message
```

## Pushing Changes

Pushing uploads local commits to the remote repository and updates the remote branch.

### What Happens During Push

```bash
git push origin main
```

**Phase 1: Pre-push Checks**
1. Git checks if local branch is ahead of remote
2. Verifies push would be fast-forward (unless forced)
3. Fails if remote has new commits (non-fast-forward)

**Phase 2: Object Transfer**
1. Git determines which objects remote needs
2. Creates packfile with missing commits/trees/blobs
3. Compresses and sends packfile over network
4. Server receives and verifies objects

**Phase 3: Reference Update**
1. Server updates `refs/heads/main` to new commit
2. Local Git updates `.git/refs/remotes/origin/main`
3. Success confirmation returned

### Push Commands

```bash
# Push to tracked upstream
git push

# Push to specific remote and branch
git push origin main

# Push and set up tracking
git push -u origin feature-new

# Push all branches
git push origin --all

# Push tags
git push origin --tags
```

### Pushing New Branches

```bash
# Create and push new branch
git switch -c feature-awesome
git commit -m "Awesome feature"
git push -u origin feature-awesome

# Now tracking is configured:
git branch -vv
# feature-awesome abc1234 [origin/feature-awesome] Awesome feature
```

### When Push Fails (Non-Fast-Forward)

```bash
git push origin main
# ! [rejected]        main -> main (non-fast-forward)
# error: failed to push some refs

# Cause: Remote has commits you don't have
# Solution: Pull first, then push
git pull --rebase origin main
git push origin main
```

### Force Pushing

```bash
# Force push (DANGEROUS - overwrites remote)
git push --force origin main

# Safer: Force push with lease
git push --force-with-lease origin main
# Only forces if remote matches your last fetch
# Protects against overwriting others' work

# When to force push:
# ✓ Personal feature branch after rebase
# ✓ After amending pushed commits
# ✗ NEVER on shared branches (main, develop)
```

### Push Configuration

```bash
# Set default push behavior
git config push.default simple    # Push current to upstream (default)
git config push.default current   # Push current to same name
git config push.default upstream  # Push current to configured upstream

# Auto-setup tracking on push
git config push.autoSetupRemote true
# Now 'git push' automatically sets up tracking
```

## Deleting Branches

### Deleting Local Branches

```bash
# Safe delete (only if merged)
git branch -d feature-merged

# Force delete (even if not merged)
git branch -D feature-abandoned

# What gets deleted:
# - Deleted: .git/refs/heads/feature-abandoned (pointer file)
# - Kept: All commits (still in .git/objects/)
# - Recoverable: Via reflog for ~90 days

# Recover deleted branch
git reflog
# abc1234 HEAD@{1}: commit: Last commit message
git branch feature-recovered abc1234
```

### Deleting Remote Branches

```bash
# Delete remote branch
git push origin --delete feature-old

# What happens:
# - Remote: refs/heads/feature-old removed from server
# - Remote-tracking: origin/feature-old NOT automatically removed
# - Local branch: Unchanged (if exists)

# Clean up stale remote-tracking branches
git fetch --prune
```

### Complete Deletion Workflow

```bash
# Delete branch everywhere

# 1. Delete local branch
git branch -d feature-done

# 2. Delete remote branch  
git push origin --delete feature-done

# 3. Others should prune their stale remote-tracking branches
git fetch --prune  # Run on other machines
```

### Automatic Pruning

```bash
# Enable automatic pruning
git config fetch.prune true

# Now git fetch automatically removes stale remote-tracking branches

# Check what would be pruned
git remote prune origin --dry-run
# Would prune origin/old-feature

# Actually prune
git remote prune origin
```

### Bulk Deletion

```bash
# Delete all merged local branches
git branch --merged main | grep -v "^\* main" | xargs git branch -d

# Delete multiple remote branches
git push origin --delete branch1 branch2 branch3
```

## Common Workflows

### Workflow 1: Start New Feature

```bash
# Update main
git switch main
git pull origin main

# Create feature branch
git switch -c feature-user-login

# Work and commit
git add login.js
git commit -m "Implement login form"
git add auth.js
git commit -m "Add authentication logic"

# Push with tracking
git push -u origin feature-user-login
```

### Workflow 2: Collaborate on Existing Feature

```bash
# Fetch latest
git fetch origin

# Create local branch tracking remote
git switch feature-api  # Auto-detects origin/feature-api

# Work and sync
git commit -m "Add new endpoint"
git pull --rebase origin feature-api
git push origin feature-api
```

### Workflow 3: Sync Before Pushing

```bash
# Fetch and compare
git fetch origin
git log --oneline main..origin/main  # What others did
git log --oneline origin/main..main  # What you did

# Rebase and push
git rebase origin/main
git push origin main
```

### Workflow 4: Merge Feature to Main

```bash
# Push feature
git switch feature-complete
git push origin feature-complete

# Merge into main
git switch main
git pull origin main
git merge --no-ff feature-complete
git push origin main

# Clean up
git branch -d feature-complete
git push origin --delete feature-complete
```

### Workflow 5: Keep Feature Updated

```bash
# Regularly rebase feature on main
git switch feature-long-running
git fetch origin
git rebase origin/main
git push --force-with-lease origin feature-long-running
```

### Workflow 6: Emergency Hotfix

```bash
# Start from main
git switch main
git pull origin main

# Create hotfix
git switch -c hotfix-critical-bug
git commit -m "Fix critical bug"
git push -u origin hotfix-critical-bug

# Merge and deploy
git switch main
git merge hotfix-critical-bug
git push origin main

# Clean up
git branch -d hotfix-critical-bug
git push origin --delete hotfix-critical-bug
```

### Workflow 7: Work Across Multiple Machines

```bash
# Machine A
git switch -c feature-x
git commit -m "Start feature"
git push -u origin feature-x

# Machine B
git fetch origin
git switch feature-x
git commit -m "Continue feature"
git push origin feature-x

# Machine A (later)
git pull origin feature-x
git commit -m "Finish feature"
git push origin feature-x
```

## Troubleshooting Common Issues

### Issue 1: Branch is Ahead/Behind Remote

```bash
# Problem: "Your branch is ahead of 'origin/main' by 5 commits"
git log origin/main..main  # View local commits

# Solution: Push commits
git push origin main
```

### Issue 2: Branches Have Diverged

```bash
# Problem: "Your branch and 'origin/main' have diverged"
git log --oneline --graph --all main origin/main

# Solution 1: Merge
git merge origin/main
git push origin main

# Solution 2: Rebase (cleaner)
git rebase origin/main
git push origin main
```

### Issue 3: Push Rejected (Non-Fast-Forward)

```bash
# Problem: "! [rejected] main -> main (non-fast-forward)"

# Solution: Pull then push
git pull --rebase origin main
git push origin main
```

### Issue 4: Deleted Remote Branch Still Shows

```bash
# Problem: git push origin --delete old-feature
# But: git branch -r still shows origin/old-feature

# Solution: Prune stale branches
git fetch --prune
```

### Issue 5: Can't Delete Branch "Not Fully Merged"

```bash
# Problem: git branch -d feature-x
# Error: "The branch 'feature-x' is not fully merged"

# Check if actually unmerged
git log main..feature-x

# Solution 1: Merge first if valuable
git switch main
git merge feature-x
git branch -d feature-x

# Solution 2: Force delete if not needed
git branch -D feature-x
```

### Issue 6: Committed to Wrong Branch

```bash
# Problem: Made commits on main instead of feature branch

# Solution: Create branch from current state
git branch feature-new  # Creates branch at current position
git reset --hard origin/main  # Reset main
git switch feature-new  # Commits now only on new branch
```

### Issue 7: Wrong Remote Branch Tracking

```bash
# Problem: Branch tracking wrong remote branch
git branch -vv
# feature abc1234 [origin/wrong-branch] Message

# Solution: Change tracking
git branch -u origin/correct-branch

# Verify
git branch -vv
# feature abc1234 [origin/correct-branch] Message
```

### Issue 8: Can't Switch Branches (Uncommitted Changes)

```bash
# Problem: "Your local changes would be overwritten"

# Solution 1: Commit changes
git commit -am "WIP"
git switch main

# Solution 2: Stash changes
git stash
git switch main
# Later: git stash pop
```

## Quick Reference

### Branch Types Summary

| Type | Location | Modified By | Purpose |
|------|----------|-------------|---------|
| **Local** | `.git/refs/heads/` | Your commits | Where you work |
| **Remote-tracking** | `.git/refs/remotes/origin/` | `git fetch` | Cache of remote state |
| **Remote** | On server | `git push` | Shared branches |

### Essential Commands

**Viewing:**
```bash
git branch              # Local branches
git branch -r           # Remote-tracking branches
git branch -a           # All branches
git branch -vv          # With tracking info
```

**Creating/Switching:**
```bash
git branch name         # Create
git switch -c name      # Create and switch
git switch name         # Switch to existing
```

**Fetching/Pulling:**
```bash
git fetch origin        # Update remote-tracking branches
git pull                # Fetch + merge
git pull --rebase       # Fetch + rebase
git fetch --prune       # Fetch and remove stale branches
```

**Pushing:**
```bash
git push                # Push to tracked upstream
git push origin main    # Push to specific branch
git push -u origin name # Push and set tracking
git push --force-with-lease  # Safe force push
```

**Deleting:**
```bash
git branch -d name      # Delete local (safe)
git branch -D name      # Delete local (force)
git push origin --delete name  # Delete remote
```

**Tracking:**
```bash
git branch -u origin/main      # Set tracking
git branch --unset-upstream    # Remove tracking
```

### Comparison Commands

```bash
git log main..origin/main       # What remote has
git log origin/main..main       # What you have
git diff main origin/main       # Content differences
git log --graph --all --oneline # Visual history
```

### Configuration

```bash
# Auto-prune on fetch
git config fetch.prune true

# Auto-setup tracking on push
git config push.autoSetupRemote true

# Default pull behavior
git config pull.rebase true     # Always rebase
git config pull.ff only          # Only fast-forward
```

### Mental Model

```
┌─────────────────────────┐
│   YOU WORK HERE         │
│   Local Branches        │  git commit
│   .git/refs/heads/*     │
└────────┬────────────────┘
         │ git fetch/push
┌────────▼────────────────┐
│   LOCAL CACHE           │
│   Remote-Tracking       │  Updated by git fetch
│   .git/refs/remotes/*   │  Read-only
└────────┬────────────────┘
         │ Network
┌────────▼────────────────┐
│   REMOTE SERVER         │
│   Remote Branches       │  Updated by git push
│   refs/heads/* (server) │  Shared with all
└─────────────────────────┘
```

### Key Takeaways

1. **Three branch types:**
   - Local: Where you work
   - Remote-tracking: Local cache of remote state
   - Remote: Shared on server

2. **Remote-tracking branches are local files:**
   - `origin/main` is on your computer
   - Updated by `git fetch`
   - Read-only

3. **Operations:**
   - `git fetch`: Updates remote-tracking branches only
   - `git pull`: Fetch + merge/rebase into local branch
   - `git push`: Updates remote branch and remote-tracking branch

4. **Always check state:**
   - `git status` shows ahead/behind
   - `git branch -vv` shows tracking
   - `git fetch` before analyzing

5. **Best practices:**
   - Fetch before pushing
   - Use `--force-with-lease` not `--force`
   - Enable automatic pruning
   - Set up tracking with `-u` flag
