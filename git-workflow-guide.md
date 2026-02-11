# Git Workflow Guide for Spring Boot/Angular Monorepo

## Table of Contents
1. [Repository Setup](#repository-setup)
2. [Initial Project Structure](#initial-project-structure)
3. [Branching Strategy](#branching-strategy)
4. [Daily Workflow Scenarios](#daily-workflow-scenarios)
5. [Backend Developer Workflows](#backend-developer-workflows)
6. [Frontend Developer Workflows](#frontend-developer-workflows)
7. [Collaboration Scenarios](#collaboration-scenarios)
8. [Conflict Resolution](#conflict-resolution)
9. [Code Review Process](#code-review-process)
10. [Best Practices](#best-practices)
11. [Advanced Scenarios](#advanced-scenarios)
12. [Troubleshooting](#troubleshooting)

---

## Repository Setup

### 1.1 Creating the Repository (Repository Owner)

```bash
# Create a new directory for the project
mkdir my-project
cd my-project

# Initialize git repository
git init

# Create the monorepo structure
mkdir -p backend frontend documents

# Create initial README
cat > README.md << 'EOF'
# My Project

Monorepo containing:
- Backend: Spring Boot application
- Frontend: Angular application
EOF

# Create .gitignore for the monorepo
cat > .gitignore << 'EOF'
# IDE
.idea/
.vscode/
*.iml
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Backend (Spring Boot/Maven)
backend/target/
backend/*.log
backend/.mvn/wrapper/maven-wrapper.jar

# Frontend (Angular/Node)
frontend/node_modules/
frontend/dist/
frontend/.angular/
frontend/npm-debug.log
frontend/yarn-error.log

# Environment files
.env
.env.local
EOF

# Add all files
git add .

# Initial commit
git commit -m "Initial commit: Setup monorepo structure"

# Create remote repository on GitHub/GitLab/Bitbucket first, then:
git remote add origin https://github.com/your-org/your-repo.git

# Push to remote
git branch -M main
git push -u origin main
```

### 1.2 Setting Up Development Branch

```bash
# Create a development branch
git checkout -b develop
git push -u origin develop

# Set develop as default branch (recommended)
# Do this in your Git hosting service settings (GitHub/GitLab/Bitbucket)
```

---

## Initial Project Structure

### 2.1 Setting Up Backend (Spring Boot)

```bash
# On develop branch
git checkout develop

# Navigate to backend folder
cd backend

# Initialize Spring Boot project (using Spring Initializr or manually)
# ... add your Spring Boot files ...

# Commit backend setup
git add .
git commit -m "feat(backend): Initialize Spring Boot project structure"
git push origin develop
```

### 2.2 Setting Up Frontend (Angular)

```bash
# Ensure you're on develop branch
git checkout develop

# Navigate to frontend folder
cd frontend

# Initialize Angular project
# ng new . (if starting fresh)
# ... add your Angular files ...

# Commit frontend setup
git add .
git commit -m "feat(frontend): Initialize Angular project structure"
git push origin develop
```

---

## Branching Strategy

### 3.1 Branch Naming Conventions

```
main              → Production-ready code
develop           → Integration branch for features
feature/*         → New features
bugfix/*          → Bug fixes
hotfix/*          → Emergency production fixes
release/*         → Release preparation
backend/*         → Backend-specific work
frontend/*        → Frontend-specific work
```

### 3.2 Creating Feature Branches

```bash
# Always branch from develop
git checkout develop
git pull origin develop

# For backend features
git checkout -b feature/backend/user-authentication

# For frontend features
git checkout -b feature/frontend/login-page

# For full-stack features
git checkout -b feature/employee-management

# For bug fixes
git checkout -b bugfix/backend/fix-null-pointer

# For hotfixes (branch from main)
git checkout main
git pull origin main
git checkout -b hotfix/critical-security-issue
```

---

## Daily Workflow Scenarios

### 4.1 Starting Your Day

```bash
# Update your local repository
git checkout develop
git pull origin develop

# Check what branches exist
git branch -a

# Check current status
git status

# Create/switch to your working branch
git checkout feature/your-feature-name
# or
git checkout -b feature/new-feature-name

# Sync your feature branch with latest develop
git merge develop
# or
git rebase develop
```

### 4.2 Making Changes

```bash
# Check what files have changed
git status

# See detailed changes
git diff

# See changes for specific file
git diff path/to/file.java

# Stage specific files
git add backend/src/main/java/com/example/Service.java

# Stage all changes in a directory
git add backend/src/

# Stage all changes
git add .

# Review staged changes
git diff --staged

# Commit with meaningful message
git commit -m "feat(backend): Add employee validation service"

# Or commit with detailed message
git commit -m "feat(backend): Add employee validation service

- Implement email validation
- Add phone number format checking
- Include unit tests for validators"
```

### 4.3 Pushing Your Work

```bash
# Push to remote (first time)
git push -u origin feature/your-feature-name

# Push subsequent changes
git push

# Force push (use with caution, only on your own branches!)
git push --force-with-lease
```

### 4.4 Ending Your Day

```bash
# Commit your work (even if incomplete)
git add .
git commit -m "wip: Work in progress on feature X"
git push

# Alternative: Stash your changes
git stash save "WIP: Description of work"
git stash list  # See all stashes

# Next day: Apply stashed changes
git stash pop   # Apply and remove from stash
# or
git stash apply # Apply but keep in stash
```

---

## Backend Developer Workflows

### 5.1 Working on a New Backend Feature

```bash
# Start from develop
git checkout develop
git pull origin develop

# Create feature branch
git checkout -b feature/backend/rest-api-endpoints

# Work on your feature
cd backend/src/main/java/com/example/controller
# ... make changes to EmployeeController.java ...

# Check status frequently
git status
git diff

# Stage and commit
git add backend/src/main/java/com/example/controller/
git commit -m "feat(backend): Add REST endpoints for employee CRUD operations"

# Add service layer
# ... make changes to service files ...
git add backend/src/main/java/com/example/service/
git commit -m "feat(backend): Implement employee service business logic"

# Add tests
git add backend/src/test/
git commit -m "test(backend): Add unit tests for employee service"

# Push to remote
git push -u origin feature/backend/rest-api-endpoints
```

### 5.2 Updating Dependencies (pom.xml)

```bash
# Create branch for dependency updates
git checkout -b chore/backend/update-dependencies

# Update pom.xml
# ... modify backend/pom.xml ...

# Test the changes
cd backend
./mvnw clean test

# If tests pass, commit
git add backend/pom.xml
git commit -m "chore(backend): Update Spring Boot to version 3.2.1"
git push -u origin chore/backend/update-dependencies
```

### 5.3 Backend Hotfix

```bash
# Hotfix from main
git checkout main
git pull origin main
git checkout -b hotfix/backend/sql-injection-fix

# Make the fix
# ... fix the security issue ...

# Test thoroughly
cd backend
./mvnw clean test

# Commit
git add backend/src/
git commit -m "fix(backend): Resolve SQL injection vulnerability in employee query"

# Push
git push -u origin hotfix/backend/sql-injection-fix

# After approval and merge to main:
# Also merge to develop
git checkout develop
git pull origin develop
git merge main
git push origin develop
```

---

## Frontend Developer Workflows

### 6.1 Working on a New Frontend Feature

```bash
# Start from develop
git checkout develop
git pull origin develop

# Create feature branch
git checkout -b feature/frontend/employee-list-component

# Work on your feature
cd frontend/src/app/components
# ... create employee-list component ...

# Check what changed
git status
git diff

# Stage and commit
git add frontend/src/app/components/employee-list/
git commit -m "feat(frontend): Add employee list component"

# Add styling
git add frontend/src/app/components/employee-list/*.css
git commit -m "style(frontend): Style employee list component"

# Add tests
git add frontend/src/app/components/employee-list/*.spec.ts
git commit -m "test(frontend): Add tests for employee list component"

# Push
git push -u origin feature/frontend/employee-list-component
```

### 6.2 Updating NPM Dependencies

```bash
# Create branch
git checkout -b chore/frontend/update-angular

# Update package.json
cd frontend
npm update
# or for major updates
npm install @angular/core@latest

# Test the application
npm test
npm run build

# Commit lock file and package.json
git add frontend/package.json frontend/package-lock.json
git commit -m "chore(frontend): Update Angular to version 17"
git push -u origin chore/frontend/update-angular
```

### 6.3 Working on UI/UX Changes

```bash
# Create branch
git checkout -b feature/frontend/responsive-design

# Make incremental commits for different components
git add frontend/src/app/components/header/
git commit -m "style(frontend): Make header responsive"

git add frontend/src/app/components/sidebar/
git commit -m "style(frontend): Make sidebar responsive"

git add frontend/src/styles.css
git commit -m "style(frontend): Update global responsive breakpoints"

# Push all changes
git push -u origin feature/frontend/responsive-design
```

---

## Collaboration Scenarios

### 7.1 Working on Same Feature (Different Parts)

**Backend Developer:**
```bash
git checkout develop
git pull origin develop
git checkout -b feature/employee-management-backend

# Work on backend
cd backend
# ... implement employee service ...
git add backend/src/
git commit -m "feat(backend): Implement employee management service"
git push -u origin feature/employee-management-backend
```

**Frontend Developer:**
```bash
git checkout develop
git pull origin develop
git checkout -b feature/employee-management-frontend

# Work on frontend
cd frontend
# ... implement employee components ...
git add frontend/src/app/employee/
git commit -m "feat(frontend): Implement employee management UI"
git push -u origin feature/employee-management-frontend
```

### 7.2 Backend Changes Required for Frontend Work

**Frontend Developer needs backend changes:**

```bash
# Frontend dev creates a feature branch
git checkout -b feature/frontend/new-dashboard

# Realizes backend API is needed
# Create issue/ticket for backend team
# Reference backend branch in your commits

git commit -m "wip(frontend): Dashboard UI (waiting for API from feature/backend/dashboard-api)"
git push
```

**Backend Developer:**

```bash
# Backend dev creates branch
git checkout -b feature/backend/dashboard-api

# Implement and push
git add backend/
git commit -m "feat(backend): Add dashboard API endpoints"
git push -u origin feature/backend/dashboard-api

# Notify frontend developer
```

**Frontend Developer continues:**

```bash
# Pull backend branch for local testing
git fetch origin
git checkout feature/backend/dashboard-api
cd backend
./mvnw spring-boot:run  # Run backend locally

# Switch back to frontend work
git checkout feature/frontend/new-dashboard

# Update and complete work
git add frontend/
git commit -m "feat(frontend): Complete dashboard with new API integration"
git push
```

### 7.3 Multiple Developers on Same Backend Feature

**Developer A:**
```bash
git checkout -b feature/backend/employee-crud
# Work on controllers
git add backend/src/main/java/com/example/controller/
git commit -m "feat(backend): Add employee controller"
git push -u origin feature/backend/employee-crud
```

**Developer B needs to join:**
```bash
# Fetch and checkout the branch
git fetch origin
git checkout feature/backend/employee-crud

# Work on different files (service layer)
git add backend/src/main/java/com/example/service/
git commit -m "feat(backend): Add employee service implementation"

# Pull any changes from Developer A first
git pull origin feature/backend/employee-crud

# Then push
git push origin feature/backend/employee-crud
```

### 7.4 Syncing Feature Branch with Develop

```bash
# You're on feature/my-feature
git checkout feature/my-feature

# Get latest develop changes
git fetch origin develop

# Option 1: Merge (preserves commit history)
git merge origin/develop

# Option 2: Rebase (cleaner history, recommended)
git rebase origin/develop

# If conflicts occur during rebase:
# 1. Fix conflicts in files
# 2. Stage resolved files
git add .
git rebase --continue

# If you want to abort rebase
git rebase --abort

# After successful merge/rebase
git push origin feature/my-feature
# or if you rebased
git push --force-with-lease origin feature/my-feature
```

---

## Conflict Resolution

### 8.1 Merge Conflicts During Pull

```bash
# You pull and get conflicts
git pull origin develop

# Output shows:
# CONFLICT (content): Merge conflict in backend/src/main/java/Service.java

# Check which files have conflicts
git status

# Open conflicted file - you'll see markers:
# <<<<<<< HEAD
# Your changes
# =======
# Their changes
# >>>>>>> origin/develop

# Edit the file to resolve conflicts
# Remove markers and keep desired code

# After fixing all conflicts
git add backend/src/main/java/Service.java
git commit -m "Merge develop into feature/my-feature, resolve conflicts"
git push
```

### 8.2 Conflicts in Pull Requests

```bash
# PR shows conflicts with develop branch

# Update your local branch
git checkout feature/my-feature
git pull origin feature/my-feature

# Merge develop into your branch
git fetch origin develop
git merge origin/develop

# Resolve conflicts as above
# ... edit files ...

git add .
git commit -m "Merge develop and resolve conflicts"
git push origin feature/my-feature

# PR will automatically update
```

### 8.3 Conflicts Between Backend and Frontend (Shared Files)

```bash
# Example: Both modified README.md or package.json at root

# Backend dev commits first
git add README.md
git commit -m "docs: Update backend setup instructions"
git push

# Frontend dev gets conflict
git pull origin develop
# CONFLICT in README.md

# Open README.md and resolve
# Keep both changes, organize properly

git add README.md
git commit -m "Merge: Combine backend and frontend README updates"
git push
```

### 8.4 Using Merge Tools

```bash
# Configure merge tool (one-time setup)
git config --global merge.tool vimdiff
# or
git config --global merge.tool meld
# or for VS Code
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait --merge $REMOTE $LOCAL $BASE $MERGED'

# When conflict occurs
git mergetool

# After resolving all conflicts
git commit
```

---

## Code Review Process

### 9.1 Creating a Pull Request

```bash
# Ensure your branch is up to date
git checkout feature/my-feature
git pull origin develop
git merge develop  # or rebase

# Push final changes
git push origin feature/my-feature

# Then create PR through GitHub/GitLab/Bitbucket web interface
# Title: "feat(backend): Add employee management API"
# Description:
# - What changes were made
# - Why they were made
# - How to test
# - Related issue numbers
```

### 9.2 Addressing Review Comments

```bash
# Make requested changes
# ... edit files based on feedback ...

# Commit changes
git add .
git commit -m "refactor: Address PR feedback - extract validation logic"

# Push (PR updates automatically)
git push origin feature/my-feature

# For major changes, you might want to add more commits
git commit -m "test: Add additional test cases per review"
git commit -m "docs: Update API documentation"
git push origin feature/my-feature
```

### 9.3 Squashing Commits Before Merge

```bash
# Interactive rebase to squash commits
git checkout feature/my-feature
git rebase -i develop

# Editor opens with commit list:
# pick abc1234 First commit
# pick def5678 Second commit  
# pick ghi9012 Third commit

# Change to:
# pick abc1234 First commit
# squash def5678 Second commit
# squash ghi9012 Third commit

# Save and close, then edit combined commit message

# Force push (your branch only!)
git push --force-with-lease origin feature/my-feature
```

### 9.4 Reviewing Others' Code

```bash
# Fetch all remote branches
git fetch origin

# Check out the PR branch
git checkout feature/others-feature
# or
git checkout -b feature/others-feature origin/feature/others-feature

# Review the changes
git log develop..feature/others-feature
git diff develop...feature/others-feature

# Test the changes locally
cd backend
./mvnw clean test
./mvnw spring-boot:run

# Or for frontend
cd frontend
npm install
npm test
npm start

# Add review comments in GitHub/GitLab interface
# Or add commits to their branch if they gave you permission
git add .
git commit -m "fix: Minor typo in documentation"
git push origin feature/others-feature
```

---

## Best Practices

### 10.1 Commit Message Conventions

```bash
# Format: <type>(<scope>): <subject>

# Types:
# feat:     New feature
# fix:      Bug fix
# docs:     Documentation changes
# style:    Code style (formatting, no logic change)
# refactor: Code refactoring
# test:     Adding tests
# chore:    Build process, dependencies
# perf:     Performance improvements

# Examples:
git commit -m "feat(backend): Add employee validation"
git commit -m "fix(frontend): Resolve navigation bug"
git commit -m "docs: Update API documentation"
git commit -m "test(backend): Add integration tests for auth"
git commit -m "chore(frontend): Update Angular dependencies"
git commit -m "refactor(backend): Extract service layer logic"
git commit -m "style(frontend): Format code with Prettier"
git commit -m "perf(backend): Optimize database queries"
```

### 10.2 Keeping Commits Atomic

```bash
# Bad: One large commit
git add .
git commit -m "Add employee feature"

# Good: Multiple focused commits
git add backend/src/main/java/com/example/entity/Employee.java
git commit -m "feat(backend): Add Employee entity"

git add backend/src/main/java/com/example/repository/EmployeeRepository.java
git commit -m "feat(backend): Add Employee repository"

git add backend/src/main/java/com/example/service/EmployeeService.java
git commit -m "feat(backend): Add Employee service"

git add backend/src/test/
git commit -m "test(backend): Add Employee service tests"
```

### 10.3 Branch Hygiene

```bash
# List all local branches
git branch

# List remote branches
git branch -r

# Delete merged local branch
git branch -d feature/completed-feature

# Delete unmerged local branch (force)
git branch -D feature/abandoned-feature

# Delete remote branch
git push origin --delete feature/old-feature

# Prune deleted remote branches
git fetch --prune

# Clean up old branches
git branch --merged develop | grep -v "develop" | grep -v "main" | xargs git branch -d
```

### 10.4 Using Git Aliases

```bash
# Set up useful aliases
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.lg "log --graph --oneline --all"
git config --global alias.last "log -1 HEAD"
git config --global alias.unstage "reset HEAD --"

# Use aliases
git co develop
git br feature/my-feature
git ci -m "feat: Add feature"
git st
git lg
```

### 10.5 Ignoring Files

```bash
# Backend specific (add to backend/.gitignore)
cat > backend/.gitignore << 'EOF'
target/
*.class
*.jar
*.war
.mvn/
mvnw
mvnw.cmd
EOF

# Frontend specific (add to frontend/.gitignore)
cat > frontend/.gitignore << 'EOF'
node_modules/
dist/
.angular/
*.log
EOF

# If you accidentally committed something
git rm --cached backend/target/*
git commit -m "chore: Remove target directory from git"
```

---

## Advanced Scenarios

### 11.1 Cherry-Picking Commits

```bash
# You want a specific commit from another branch
git checkout feature/my-feature

# Find the commit hash
git log feature/other-feature

# Cherry-pick the commit
git cherry-pick abc1234

# If conflicts, resolve them
git add .
git cherry-pick --continue

# Or abort
git cherry-pick --abort

# Push
git push origin feature/my-feature
```

### 11.2 Reverting Changes

```bash
# Undo last commit (keep changes staged)
git reset --soft HEAD~1

# Undo last commit (keep changes unstaged)
git reset HEAD~1

# Undo last commit (discard changes) - DANGEROUS!
git reset --hard HEAD~1

# Undo a pushed commit (create new commit that reverses)
git revert abc1234
git push origin develop

# Revert a merge commit
git revert -m 1 abc1234
```

### 11.3 Stashing for Context Switching

```bash
# Save current work temporarily
git stash save "WIP: employee feature"

# See stash list
git stash list

# Apply latest stash
git stash pop

# Apply specific stash
git stash apply stash@{1}

# Apply stash to different branch
git stash save "Feature work"
git checkout different-branch
git stash pop

# Delete stash
git stash drop stash@{0}

# Clear all stashes
git stash clear
```

### 11.4 Working with Submodules (If Needed)

```bash
# Add a submodule
git submodule add https://github.com/org/library.git backend/libs/library

# Clone repository with submodules
git clone --recursive https://github.com/org/repo.git

# Or after cloning
git submodule init
git submodule update

# Update submodule
cd backend/libs/library
git pull origin main
cd ../../..
git add backend/libs/library
git commit -m "chore: Update library submodule"
```

### 11.5 Git Bisect (Finding Bugs)

```bash
# Start bisecting
git bisect start

# Mark current version as bad
git bisect bad

# Mark a known good version
git bisect good v1.0.0

# Git checks out middle commit
# Test it, then mark as good or bad
git bisect good  # or git bisect bad

# Continue until bug is found
# Git will tell you the first bad commit

# End bisect
git bisect reset
```

### 11.6 Rebasing Interactive

```bash
# Rewrite last 3 commits
git rebase -i HEAD~3

# Commands in editor:
# pick = use commit
# reword = change commit message
# edit = stop to amend commit
# squash = combine with previous commit
# fixup = like squash but discard message
# drop = remove commit

# Example:
# pick abc1234 First commit
# reword def5678 Second commit
# squash ghi9012 Third commit

# Force push after rebase
git push --force-with-lease origin feature/my-feature
```

### 11.7 Working with Tags

```bash
# Create tag for release
git tag -a v1.0.0 -m "Release version 1.0.0"

# Push tag to remote
git push origin v1.0.0

# Push all tags
git push origin --tags

# List tags
git tag

# Checkout specific tag
git checkout v1.0.0

# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin :refs/tags/v1.0.0
```

### 11.8 Worktrees (Multiple Checkouts)

```bash
# Create worktree for hotfix while working on feature
git worktree add ../hotfix-workspace hotfix/critical-bug

# Work in the hotfix
cd ../hotfix-workspace
# ... make fixes ...
git commit -am "fix: Critical bug"
git push

# Return to main workspace
cd ../main-workspace

# List worktrees
git worktree list

# Remove worktree
git worktree remove ../hotfix-workspace
```

---

## Troubleshooting

### 12.1 Accidentally Committed to Wrong Branch

```bash
# You committed to develop instead of feature branch

# Create branch at current HEAD
git branch feature/my-feature

# Reset develop to before your commits
git reset --hard origin/develop

# Switch to your feature branch
git checkout feature/my-feature

# Your commits are now on the feature branch
git push -u origin feature/my-feature
```

### 12.2 Need to Undo Pushed Commits

```bash
# If no one else pulled your commits
git reset --hard HEAD~1
git push --force-with-lease origin feature/my-feature

# If others pulled your commits (safer)
git revert HEAD
git push origin feature/my-feature
```

### 12.3 Merge Nightmare - Start Over

```bash
# Abort ongoing merge
git merge --abort

# Or abort rebase
git rebase --abort

# Reset to remote state
git fetch origin
git reset --hard origin/feature/my-feature

# Start fresh
git checkout develop
git pull origin develop
git checkout -b feature/my-feature-v2
# ... copy your changes manually ...
```

### 12.4 Recover Deleted Branch

```bash
# Find the commit hash
git reflog

# Look for: "checkout: moving from feature/deleted-branch to develop"
# Note the commit hash

# Recreate branch
git checkout -b feature/recovered-branch abc1234
```

### 12.5 Large Files Accidentally Committed

```bash
# Remove large file from history
git filter-branch --tree-filter 'rm -f path/to/large-file' HEAD

# Or use BFG Repo Cleaner (faster)
java -jar bfg.jar --delete-files large-file.zip
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# Force push
git push --force-with-lease origin develop
```

### 12.6 Diverged Branches

```bash
# Error: "Your branch and 'origin/feature' have diverged"

# Option 1: Merge remote changes
git pull origin feature/my-feature

# Option 2: Rebase on remote (cleaner)
git pull --rebase origin feature/my-feature

# Option 3: Force push (only if you're sure)
git push --force-with-lease origin feature/my-feature
```

### 12.7 Can't Push - Non-Fast-Forward

```bash
# Error: "Updates were rejected because the tip of your current branch is behind"

# Pull changes first
git pull origin feature/my-feature

# Or rebase
git pull --rebase origin feature/my-feature

# Then push
git push origin feature/my-feature
```

---

## Complete Workflow Example

### Scenario: Implementing Employee Search Feature

**Backend Developer:**

```bash
# Day 1 - Start
git checkout develop
git pull origin develop
git checkout -b feature/backend/employee-search

# Implement search endpoint
cd backend/src/main/java/com/example/controller
# ... edit EmployeeController.java ...
git add EmployeeController.java
git commit -m "feat(backend): Add employee search endpoint"

# Implement service
cd ../service/impl
# ... edit EmployeeServiceImpl.java ...
git add EmployeeServiceImpl.java
git commit -m "feat(backend): Implement employee search logic"

# Push end of day
git push -u origin feature/backend/employee-search

# Day 2 - Continue
git pull origin feature/backend/employee-search  # In case of team member contributions
cd backend/src/test/java/com/example/service
# ... add tests ...
git add .
git commit -m "test(backend): Add employee search tests"

# Update with latest develop
git fetch origin develop
git rebase origin/develop
git push --force-with-lease origin feature/backend/employee-search

# Create PR
# Via GitHub/GitLab web interface

# Address review comments
# ... make changes ...
git add .
git commit -m "refactor: Extract search criteria validation"
git push origin feature/backend/employee-search

# PR approved and merged to develop
```

**Frontend Developer:**

```bash
# Day 1 - Start (parallel with backend)
git checkout develop
git pull origin develop
git checkout -b feature/frontend/employee-search-ui

# Create search component
cd frontend/src/app/components
ng generate component employee-search
git add employee-search/
git commit -m "feat(frontend): Create employee search component"

# Mock backend API for development
# ... create mock service ...
git add .
git commit -m "feat(frontend): Add mock employee search service"
git push -u origin feature/frontend/employee-search-ui

# Day 2 - Backend API ready
git pull origin develop
git fetch origin feature/backend/employee-search

# Test with real backend locally
git worktree add ../backend-test feature/backend/employee-search
cd ../backend-test/backend
./mvnw spring-boot:run &
cd ../../main-workspace/frontend

# Update service to use real API
git add .
git commit -m "feat(frontend): Integrate employee search with backend API"
git push origin feature/frontend/employee-search-ui

# Create PR
# After both PRs merged, they work together in develop
```

---

## Quick Reference

### Essential Commands

```bash
# Setup
git clone <url>
git init

# Daily work
git status
git add <file>
git commit -m "message"
git push
git pull

# Branching
git branch <name>
git checkout <branch>
git checkout -b <new-branch>
git merge <branch>

# Syncing
git fetch
git pull
git push

# Information
git log
git diff
git status

# Undo
git reset
git revert
git checkout -- <file>

# Collaboration
git fetch origin
git pull origin <branch>
git push origin <branch>
```

### Common Patterns

```bash
# Start new feature
git checkout develop && git pull && git checkout -b feature/my-feature

# Update feature with develop
git fetch origin develop && git rebase origin/develop

# Quick commit
git add . && git commit -m "feat: description" && git push

# Clean merged branches
git branch --merged | grep -v "develop\|main" | xargs git branch -d
```

---

## Summary

This guide covers comprehensive Git workflows for a Spring Boot/Angular monorepo. Key principles:

1. **Always branch from develop** for new features
2. **Keep commits atomic and well-described**
3. **Pull before push** to avoid conflicts
4. **Use descriptive branch names** with prefixes
5. **Review code through PRs** before merging
6. **Keep develop stable** and main production-ready
7. **Communicate with team** about major changes
8. **Test locally** before pushing
9. **Document your changes** in commit messages
10. **Clean up branches** after merging

Remember: Git is a tool to facilitate collaboration. When in doubt, communicate with your team!
