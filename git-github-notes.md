# Git & GitHub Learning Notes

---

## Core Concept: The 3 Areas

```
Working Directory  →  Staging Area  →  Repository
  (your files)         (git add)       (git commit)
```

1. **Working Directory** — where you edit files normally
2. **Staging Area** — where you prepare changes before saving
3. **Repository** — the permanent history of saved snapshots

---

## First Time Setup

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

---

## Basic Commands

```bash
git init                         # start a new repository
git status                       # see current state of repo
git add filename.txt             # stage a specific file
git add .                        # stage all changes
git commit -m "message"          # save a snapshot with a message
git log                          # see full commit history
git log --oneline                # compact commit history
git log --oneline --graph --all  # visual branch history
```

---

## Branching

Branches let you work independently without affecting main code.

```bash
git branch branch-name        # create a branch
git checkout branch-name      # switch to a branch
git checkout -b branch-name   # create AND switch in one command
git merge branch-name         # merge a branch into current branch
```

```
main:     A --- B ------- M   (M = merge commit)
               \         /
feature:        C --- D
```

---

## Merge Conflicts

Happens when two branches independently edit the same line.

Git marks the conflict in the file like this:

```
<<<<<<< HEAD
your version (current branch)
=======
their version (incoming branch)
>>>>>>> branch-name
```

**How to resolve:**
1. Edit the file — keep what you want, remove conflict markers
2. `git add filename.txt`
3. `git commit -m "Resolve merge conflict"`

**Key rule:**
- **Fast-forward** = one branch is just ahead of the other (straight line)
- **Conflict** = both branches changed independently after splitting

---

## Git Stash

Temporarily saves uncommitted work so you can switch branches.

```bash
git stash                   # save unfinished work, clean directory
git stash list              # see all stashes
git stash pop               # restore latest stash and remove from list
git stash apply             # restore but KEEP in stash list
git stash pop stash@{1}     # restore specific stash
git stash drop stash@{1}    # delete specific stash
git stash clear             # delete ALL stashes
```

**Common scenario:**
```
working on feature → urgent bug reported
→ git stash → fix bug → git stash pop → continue
```

---

## Undoing Mistakes

### 1. git restore — undo uncommitted changes (PERMANENT)
```bash
git restore filename.txt
```

### 2. git revert — undo a commit safely (adds new commit, keeps history)
```bash
git revert <commit-hash>
```
> Use this on shared/public branches

### 3. git reset — undo commits (rewrites history, use with caution)
```bash
git reset --soft HEAD~1    # undo commit, keep changes STAGED
git reset --mixed HEAD~1   # undo commit, keep changes UNSTAGED
git reset --hard HEAD~1    # undo commit, DELETE changes permanently
```

> **WARNING:** Never use `git reset` on commits already pushed and shared!

| Command | Use case |
|---|---|
| `git restore` | Undo uncommitted file changes |
| `git revert` | Undo a commit safely |
| `git reset` | Undo commits (dangerous on shared branches) |

---

## GitHub — Pushing to Remote

```bash
git remote add origin <url>   # link local repo to GitHub
git push -u origin main       # first push (sets default)
git push                      # subsequent pushes
```

**Daily workflow:**
```bash
git add .
git commit -m "description"
git push
```

---

## Pull Requests (PRs)

A PR proposes changes from a branch to be reviewed and merged.

```bash
git checkout -b feature-branch     # create branch
# make changes
git add .
git commit -m "message"
git push -u origin feature-branch  # push branch to GitHub
# open PR on GitHub → review → merge
git checkout main
git pull                            # sync latest changes
```

**Best practices:**
- Keep PRs small and focused (one feature/fix per PR)
- Write clear descriptions (what and why)
- Branch naming: `feature/...`, `fix/...`, `chore/...`
- At least one reviewer before merging

---

## git fetch vs git pull

```
git fetch   → downloads changes but does NOT apply them
git pull    → downloads AND immediately applies (fetch + merge)

git pull = git fetch + git merge
```

**Review after fetch:**
```bash
git fetch origin
git log HEAD..origin/main --oneline   # see incoming commits
git diff HEAD origin/main             # see incoming changes
git merge origin/main                 # apply when ready
```

---

## .gitignore

Tells Git to ignore certain files (passwords, dependencies, etc.)

```
secret.txt       # ignore specific file
*.log            # ignore all .log files
node_modules/    # ignore entire folder
.env             # ignore environment variables file
```

**Important:**
- Save `.gitignore` as UTF-8 encoding
- Commit `.gitignore` so your team shares the same rules
- Ignored files still exist on disk, Git just won't track them

---

## Rebasing

Alternative to merging — creates a clean linear history.

```bash
git checkout feature-branch
git rebase main               # replay commits on top of main
git rebase --continue         # after resolving conflicts
git rebase --abort            # cancel rebase
```

**Merge vs Rebase:**

```
Merge:   A --- B ------- M   (keeps full history, adds merge commit)
               \         /
feature:        C --- D

Rebase:  A --- B --- C' --- D'   (clean straight line)
```

> **WARNING:** Never rebase commits already pushed and shared with others!
> **Rule:** Use merge by default. Only rebase if your team has a rebase convention.

---

## Removing .env File Accidentally Uploaded to GitHub

> **Step 1: Revoke the exposed token immediately**
> Databricks → Settings → Developer → Access Tokens → Revoke

```bash
# Check commit history for the .env file
git log --oneline
git log --all --full-history -- .env
git show {id}:.env              # check specific commit for .env content

# Remove .env from entire git history
git stash clear                 # clear stash entries that held .env
git filter-branch --force --index-filter "git rm --cached --ignore-unmatch .env" --prune-empty --tag-name-filter cat -- --all
git push origin master --force  # overwrite remote with cleaned history

# Verify it's gone
git show 4c960396:.env          # should return: fatal: path does not exist
git ls-files .env               # should return nothing = not tracked
```

> **Step 2: Generate a new token**
> Databricks → Settings → Developer → Access Tokens → Generate new token
> Update local `.env` with new token

---

## Tags

Bookmarks on specific commits, used for version releases.

```bash
git tag -a v1.0.0 -m "First release"   # create annotated tag
git tag                                  # list all tags
git show v1.0.0                          # see tag details
git push origin v1.0.0                  # push tag to GitHub
git checkout v1.0.0                     # go to that snapshot
```

**Semantic Versioning (SemVer):**
```
v MAJOR . MINOR . PATCH
  v1     .  1   .  0

MAJOR → breaking changes
MINOR → new features, backwards compatible
PATCH → bug fixes
```

---

## Forking & Cloning

**Fork** = personal copy of someone else's repo on your GitHub account

```
Original Repo → Fork it → Your Copy on GitHub
→ Clone locally → Make changes → Push to your fork
→ Open PR to original repo
```

```bash
git clone <url>                # download repo to local machine
git remote -v                  # see linked remote repos
git remote add upstream <url>  # link to original repo
```

**Keeping fork in sync:**
```bash
git fetch upstream             # download latest from original
git merge upstream/dev         # merge into your local branch
git push                       # update your fork on GitHub
```

| | Fork | Clone |
|---|---|---|
| Where | GitHub to GitHub | GitHub to your machine |
| Purpose | Your own copy to contribute back | Work locally |

---

## GitHub Actions

Automates tasks when something happens in your repo.

**CI/CD:**
- **CI** (Continuous Integration) → automatically test on every push
- **CD** (Continuous Deployment) → automatically deploy when tests pass

**Workflow file location:** `.github/workflows/hello.yml`

```yaml
name: Hello World

on: [push]

jobs:
  say-hello:
    runs-on: ubuntu-latest
    steps:
      - name: Print hello
        run: echo "Hello from GitHub Actions!"
```

**Common real-world uses:**

| Trigger | Action |
|---|---|
| `on: [push]` | Run tests automatically |
| Merge to main | Deploy to production |
| Open a PR | Check code quality |
| Create a tag | Build and release |

---

## Complete Daily Workflow

```bash
git checkout -b feature-branch    # create branch
# make changes
git add .                          # stage
git commit -m "message"            # commit
git push -u origin feature-branch  # push to GitHub
# open PR on GitHub → review → merge
git checkout main                  # switch back
git pull                           # sync latest
```

---

## Quick Reference — All Commands

```bash
# Setup
git config --global user.name "Name"
git config --global user.email "email"

# Core
git init
git status
git add .
git commit -m "message"
git log --oneline

# Branching
git checkout -b branch-name
git merge branch-name
git branch -d branch-name         # delete branch

# Remote
git remote add origin <url>
git push -u origin main
git push
git pull
git fetch
git clone <url>

# Undoing
git restore file.txt
git revert <hash>
git reset --soft HEAD~1
git reset --hard HEAD~1

# Stash
git stash
git stash pop
git stash list

# Tags
git tag -a v1.0.0 -m "message"
git push origin v1.0.0

# Upstream (forks)
git remote add upstream <url>
git fetch upstream
git merge upstream/main
```
