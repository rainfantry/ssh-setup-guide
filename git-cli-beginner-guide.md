# Git CLI Beginner's Guide

Complete guide for using Git from the command line - from zero to confident.

## Table of Contents
1. [What is Git?](#what-is-git)
2. [Initial Setup](#initial-setup)
3. [Basic Concepts](#basic-concepts)
4. [Essential Commands](#essential-commands)
5. [Daily Workflow](#daily-workflow)
6. [Working with GitHub](#working-with-github)
7. [Common Scenarios](#common-scenarios)
8. [Troubleshooting](#troubleshooting)
9. [Quick Reference](#quick-reference)

---

## What is Git?

**Git** is a version control system that tracks changes to your files. Think of it as:
- A save system with unlimited undo
- A time machine for your code
- A collaboration tool for teams

**Key Benefits:**
- Track every change you make
- Go back to any previous version
- Work on features without breaking main code
- Collaborate with others safely

---

## Initial Setup

### Install Git

**Windows:**
```bash
winget install Git.Git
```

**Mac:**
```bash
brew install git
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt install git
```

**Linux (Fedora/CentOS):**
```bash
sudo dnf install git
```

### Configure Git

**Set your identity (required):**
```bash
# Your name (will appear in commits)
git config --global user.name "Your Name"

# Your email (use your GitHub email)
git config --global user.email "your.email@example.com"

# Verify settings
git config --global user.name
git config --global user.email
```

**Optional but recommended:**
```bash
# Set default branch name to 'main'
git config --global init.defaultBranch main

# Use better colors
git config --global color.ui auto

# Set default editor (choose one)
git config --global core.editor "code --wait"  # VS Code
git config --global core.editor "nano"         # Nano (simple)
git config --global core.editor "vim"          # Vim (advanced)
```

**View all settings:**
```bash
git config --global --list
```

---

## Basic Concepts

### Repository (Repo)
A folder tracked by Git. Contains all your files and their complete history.

### Commit
A snapshot of your files at a specific time. Like a save point in a game.

### Branch
A separate line of development. Like a parallel universe for your code.

### Remote
A version of your repository hosted online (like on GitHub).

### The Three States

Files in Git can be in three states:

1. **Working Directory** - Files you're currently editing
2. **Staging Area** - Files ready to be committed
3. **Repository** - Committed files (saved permanently)

```
Working Directory → git add → Staging Area → git commit → Repository
```

---

## Essential Commands

### Starting a Repository

**Create a new repository:**
```bash
# Navigate to your project folder
cd /path/to/your/project

# Initialize Git
git init

# Check status
git status
```

**Clone an existing repository:**
```bash
# Clone from GitHub
git clone https://github.com/username/repo-name.git

# Clone with SSH
git clone git@github.com:username/repo-name.git

# Clone to specific folder
git clone https://github.com/username/repo-name.git my-folder
```

### Checking Status

```bash
# See what's changed
git status

# Short format
git status -s

# See what's actually different
git diff

# See staged changes
git diff --staged
```

### Adding Files

```bash
# Add a specific file
git add filename.txt

# Add multiple files
git add file1.txt file2.txt

# Add all files in current directory
git add .

# Add all files in project
git add -A

# Add all .js files
git add *.js

# Interactive add (advanced)
git add -p
```

### Committing Changes

```bash
# Commit with message
git commit -m "Add new feature"

# Multi-line commit message
git commit -m "Add user login" -m "Implemented authentication with JWT tokens"

# Add and commit in one step (only tracked files)
git commit -am "Quick update"

# Open editor for longer message
git commit
```

**Good commit message format:**
```
Short summary (50 chars or less)

More detailed explanation if needed. Explain what and why,
not how (code shows how).

- Bullet points are okay
- Usually use imperative mood: "Add feature" not "Added feature"
```

### Viewing History

```bash
# Show commit history
git log

# Compact one-line format
git log --oneline

# Show last 5 commits
git log -5

# Show with file changes
git log --stat

# Show actual changes
git log -p

# Pretty format
git log --oneline --graph --decorate --all

# Search commits
git log --grep="bug fix"

# See who changed what
git blame filename.txt
```

### Undoing Changes

**Before staging:**
```bash
# Discard changes in a file
git checkout -- filename.txt

# Or using restore (newer Git)
git restore filename.txt

# Discard all changes
git checkout -- .
git restore .
```

**After staging:**
```bash
# Unstage a file
git reset HEAD filename.txt

# Or using restore (newer Git)
git restore --staged filename.txt
```

**After committing:**
```bash
# Undo last commit, keep changes
git reset --soft HEAD~1

# Undo last commit, discard changes (DANGEROUS!)
git reset --hard HEAD~1

# Create new commit that undoes previous commit
git revert HEAD
```

---

## Daily Workflow

### Typical Work Session

```bash
# 1. Start work - get latest changes
git pull

# 2. Make your changes
# ... edit files ...

# 3. Check what changed
git status
git diff

# 4. Stage your changes
git add .

# 5. Commit with message
git commit -m "Add new feature description"

# 6. Push to remote
git push
```

### Working with Branches

**Why use branches?**
- Work on features without affecting main code
- Keep main branch stable
- Isolate different tasks

**Basic branch commands:**
```bash
# List branches
git branch

# Create new branch
git branch feature-name

# Switch to branch
git checkout feature-name

# Create and switch in one command
git checkout -b feature-name

# Or using switch (newer Git)
git switch feature-name
git switch -c feature-name

# Delete branch
git branch -d feature-name

# Force delete (if unmerged)
git branch -D feature-name
```

**Branch workflow:**
```bash
# 1. Create feature branch
git checkout -b add-login-page

# 2. Make changes and commit
git add .
git commit -m "Add login page HTML"

# 3. Switch back to main
git checkout main

# 4. Merge feature into main
git merge add-login-page

# 5. Delete feature branch
git branch -d add-login-page
```

### Merging

```bash
# Merge branch into current branch
git merge branch-name

# If conflicts occur:
# 1. Open conflicted files
# 2. Find conflict markers: <<<<<<<, =======, >>>>>>>
# 3. Edit to resolve
# 4. Add resolved files
git add resolved-file.txt
# 5. Complete merge
git commit
```

---

## Working with GitHub

### Setup SSH for GitHub

**Generate SSH key:**
```bash
# Generate key
ssh-keygen -t ed25519 -C "your.email@example.com"

# View public key
cat ~/.ssh/id_ed25519.pub
```

**Add to GitHub:**
1. Copy your public key
2. Go to https://github.com/settings/ssh/new
3. Paste key and save

**Test connection:**
```bash
ssh -T git@github.com
```

### Connecting to GitHub

**Add remote to existing repo:**
```bash
# Add GitHub as remote
git remote add origin git@github.com:username/repo-name.git

# Verify remote
git remote -v

# Push to GitHub
git branch -M main
git push -u origin main
```

**Change remote URL:**
```bash
# View current remote
git remote -v

# Change to SSH
git remote set-url origin git@github.com:username/repo-name.git

# Change to HTTPS
git remote set-url origin https://github.com/username/repo-name.git
```

### Syncing with GitHub

```bash
# Get latest changes
git pull

# Push your changes
git push

# Push new branch
git push -u origin branch-name

# See remote branches
git branch -r

# See all branches
git branch -a
```

### Forking Workflow

```bash
# 1. Fork on GitHub (click Fork button)

# 2. Clone your fork
git clone git@github.com:your-username/repo-name.git

# 3. Add upstream remote (original repo)
git remote add upstream git@github.com:original-owner/repo-name.git

# 4. Get latest from upstream
git fetch upstream
git merge upstream/main

# 5. Make changes and push to your fork
git push origin main

# 6. Create Pull Request on GitHub
```

---

## Common Scenarios

### Scenario 1: First Time Using Git

```bash
# 1. Configure Git
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# 2. Create a project folder
mkdir my-project
cd my-project

# 3. Initialize Git
git init

# 4. Create a file
echo "# My Project" > README.md

# 5. Add and commit
git add README.md
git commit -m "Initial commit"

# 6. Create GitHub repo, then:
git remote add origin git@github.com:username/my-project.git
git branch -M main
git push -u origin main
```

### Scenario 2: Contributing to a Project

```bash
# 1. Clone the repository
git clone git@github.com:username/project.git
cd project

# 2. Create feature branch
git checkout -b fix-typo

# 3. Make changes
# ... edit files ...

# 4. Add and commit
git add .
git commit -m "Fix typo in README"

# 5. Push branch
git push -u origin fix-typo

# 6. Create Pull Request on GitHub
```

### Scenario 3: Fixing a Mistake

**Wrong commit message:**
```bash
# Amend last commit message
git commit --amend -m "Correct message"
```

**Forgot to add a file:**
```bash
git add forgotten-file.txt
git commit --amend --no-edit
```

**Committed to wrong branch:**
```bash
# 1. Create correct branch from current state
git branch correct-branch

# 2. Reset current branch
git reset --hard HEAD~1

# 3. Switch to correct branch
git checkout correct-branch
```

### Scenario 4: Keeping Fork Updated

```bash
# 1. Add upstream remote (once)
git remote add upstream git@github.com:original/repo.git

# 2. Fetch upstream changes
git fetch upstream

# 3. Switch to main
git checkout main

# 4. Merge upstream
git merge upstream/main

# 5. Push to your fork
git push origin main
```

### Scenario 5: Stashing Changes

```bash
# Save work temporarily
git stash

# Do other work...
git checkout other-branch

# Come back and restore
git checkout original-branch
git stash pop

# List stashes
git stash list

# Apply specific stash
git stash apply stash@{0}

# Drop stash
git stash drop
```

---

## Troubleshooting

### Problem: "Permission denied (publickey)"

**Solution:**
```bash
# Check if SSH key exists
ls ~/.ssh/id_ed25519.pub

# If not, generate one
ssh-keygen -t ed25519 -C "your@email.com"

# Add to GitHub
cat ~/.ssh/id_ed25519.pub
# Copy output and add at: https://github.com/settings/ssh/new

# Test connection
ssh -T git@github.com
```

### Problem: "Your branch is ahead of 'origin/main'"

**Solution:**
```bash
# Push your commits
git push
```

### Problem: "Your branch is behind 'origin/main'"

**Solution:**
```bash
# Pull latest changes
git pull
```

### Problem: Merge Conflicts

**Solution:**
```bash
# 1. Pull latest changes
git pull

# 2. Git shows conflicts like:
# <<<<<<< HEAD
# Your changes
# =======
# Their changes
# >>>>>>> branch-name

# 3. Edit files to resolve
# Keep what you want, remove markers

# 4. Add resolved files
git add resolved-file.txt

# 5. Complete merge
git commit
```

### Problem: Accidentally committed to main

**Solution:**
```bash
# 1. Create feature branch with current state
git branch feature-branch

# 2. Reset main to previous commit
git reset --hard HEAD~1

# 3. Switch to feature branch
git checkout feature-branch
```

### Problem: Need to undo a pushed commit

**Solution:**
```bash
# Create reverse commit (safe)
git revert HEAD
git push

# Or force push (DANGEROUS if collaborating!)
git reset --hard HEAD~1
git push --force
```

### Problem: ".gitignore not working"

**Solution:**
```bash
# Remove cached files
git rm -r --cached .

# Re-add everything
git add .

# Commit
git commit -m "Fix .gitignore"
```

---

## Quick Reference

### Setup & Configuration

```bash
git config --global user.name "Name"
git config --global user.email "email@example.com"
git config --global --list
```

### Starting a Repo

```bash
git init                                    # Create new repo
git clone <url>                            # Clone existing repo
```

### Basic Workflow

```bash
git status                                 # Check status
git add <file>                            # Stage file
git add .                                 # Stage all files
git commit -m "message"                   # Commit changes
git push                                  # Push to remote
git pull                                  # Pull from remote
```

### Branches

```bash
git branch                                # List branches
git branch <name>                         # Create branch
git checkout <name>                       # Switch branch
git checkout -b <name>                    # Create and switch
git merge <branch>                        # Merge branch
git branch -d <name>                      # Delete branch
```

### History & Changes

```bash
git log                                   # View history
git log --oneline                         # Compact history
git diff                                  # See changes
git diff --staged                         # See staged changes
git show <commit>                         # Show commit details
```

### Undoing

```bash
git checkout -- <file>                    # Discard changes
git restore <file>                        # Discard changes (newer)
git reset HEAD <file>                     # Unstage file
git reset --soft HEAD~1                   # Undo commit, keep changes
git reset --hard HEAD~1                   # Undo commit, discard changes
git revert HEAD                           # Create reverse commit
```

### Remote

```bash
git remote -v                             # List remotes
git remote add origin <url>               # Add remote
git remote set-url origin <url>           # Change remote URL
git push -u origin main                   # Push and set upstream
git fetch                                 # Download changes
git pull                                  # Fetch and merge
```

### Stashing

```bash
git stash                                 # Save work temporarily
git stash pop                             # Restore saved work
git stash list                            # List stashes
git stash drop                            # Delete stash
```

### Helpful Aliases

```bash
# Add these to save typing
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.lg "log --oneline --graph --decorate"

# Now use:
git st      # instead of git status
git co main # instead of git checkout main
git lg      # pretty log
```

---

## Best Practices

### Commit Messages

**Good:**
```
Add user authentication feature
Fix login button alignment
Update dependencies to latest versions
```

**Bad:**
```
update
fix
stuff
asdfasdf
```

**Format:**
```
Short summary (50 chars max)

Longer explanation if needed (wrap at 72 chars):
- What changed
- Why it changed
- Any side effects or considerations
```

### .gitignore

Create a `.gitignore` file to exclude files from Git:

```gitignore
# Dependencies
node_modules/
vendor/

# Build outputs
dist/
build/
*.exe

# Environment files
.env
.env.local
config/secrets.yml

# IDE files
.vscode/
.idea/
*.swp

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Temporary files
tmp/
temp/
*.tmp
```

### Commit Frequency

- **Commit often** - Small, focused commits
- **One logical change** per commit
- **Working state** - Each commit should work
- **Before breaks** - Commit before lunch, end of day

### Branch Naming

```bash
feature/user-login
fix/navbar-alignment
hotfix/critical-bug
refactor/database-queries
docs/readme-update
```

---

## Learning Resources

### Practice

- Try Git: https://try.github.io/
- Learn Git Branching: https://learngitbranching.js.org/
- Git exercises: https://gitexercises.fracz.com/

### Documentation

- Official Git Book: https://git-scm.com/book/en/v2
- GitHub Guides: https://guides.github.com/
- Git Cheat Sheet: https://education.github.com/git-cheat-sheet-education.pdf

### Videos

- Git & GitHub Crash Course: Search YouTube
- The Net Ninja - Git & GitHub Tutorial
- Traversy Media - Git Tutorial

---

**Created:** 2026-02-14
**Author:** George Wu
**Purpose:** Complete beginner's guide to Git command line
