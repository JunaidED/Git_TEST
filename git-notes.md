# Git & GitHub Notes

---

## 1. Core Concepts

### The 3 Areas
```
Working Directory  →  Staging Area  →  Repository
  (your files)         (git add)       (git commit)
```
- **Working Directory** — where you edit files normally
- **Staging Area** — where you prepare changes before saving
- **Repository** — the permanent history of saved snapshots

### Basic Setup
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### Core Workflow
```bash
git status                  # see current state
git add filename.txt        # stage a file
git add .                   # stage all changes
git commit -m "message"     # save a snapshot
git log                     # view full history
git log --oneline           # view compact history
git log --oneline --graph --all  # visual branch history
```

---

## 2. Branching & Merging

### What is a Branch?
A branch is a separate line of development. Work on new features without touching main.

```
main:     A --- B
               \
feature:        C --- D
```

### Branch Commands
```bash
git branch branch-name          # create a branch
git checkout branch-name        # switch to a branch
git checkout -b branch-name     # create AND switch in one command
git branch                      # list all branches
git merge branch-name           # merge branch into current branch
```

### Fast-Forward vs Conflict Merge
- **Fast-forward** — one branch is simply ahead of the other, no competing changes. Git just moves the pointer forward.
- **Merge conflict** — both branches independently changed the same line after splitting. Git asks you to decide which to keep.

```
Fast-forward (no conflict):
main:             A --- B
                         \
branch:                   B --- C   ← branch started AFTER B

Conflict:
main:             A --- B --- C     ← edited line 1
                   \
branch:            A --- D          ← also edited line 1 independently
```

---

## 3. Merge Conflicts

### When do they happen?
When two branches edit the **same line** in the same file independently after splitting.

### What it looks like in the file
```
<<<<<<< HEAD
Hello, Git! - edited on main        ← your current branch
=======
Hello, Git! - edited on feature     ← incoming branch
>>>>>>> feature-branch
```

### How to resolve
1. Edit the file — keep what you want, delete conflict markers
2. Stage the resolved file: `git add filename.txt`
3. Complete the merge: `git commit -m "Resolve merge conflict"`

VSCode also shows buttons: **Accept Current Change**, **Accept Incoming Change**, **Accept Both**

---

## 4. Pull Requests (PRs)

### What is a PR?
A way of saying: "I made changes on a branch — please review and merge them into main."

### Full PR Workflow
```bash
git checkout -b feature-branch     # create branch
# make changes
git add .
git commit -m "description"
git push -u origin feature-branch  # push branch to GitHub
# open PR on GitHub → review → merge
git checkout main
git pull                           # sync latest changes
```

### Best Practices
- Keep PRs small and focused (one feature or bug fix)
- Write a clear title and description
- One other person should review before merging
- Delete the branch after merging
- Naming conventions: `feature/...`, `fix/...`, `chore/...`

### Naming Convention
```
main          → stable, production-ready code
feature/...   → new features
fix/...       → bug fixes
chore/...     → cleanup, dependencies
```

---

## 5. GitHub (Remote)

### Pushing a Local Repo to GitHub
```bash
git remote add origin https://github.com/USERNAME/REPO.git
git branch -M main
git push -u origin main
```

### Common Remote Commands
```bash
git push                    # upload commits to GitHub
git pull                    # download and apply latest changes
git remote -v               # see linked remote URLs
```

---

## 6. git stash

### What it does
Temporarily saves uncommitted work and cleans your working directory so you can switch branches.

### Commands
```bash
git stash                   # save unfinished work, clean directory
git stash list              # see all stashes (stash@{0}, stash@{1}...)
git stash pop               # restore latest stash and remove from list
git stash apply             # restore but KEEP in stash list
git stash pop stash@{1}     # restore a specific stash
git stash drop stash@{1}    # delete a specific stash without restoring
git stash clear             # delete ALL stashes
```

### Common Real-World Scenario
```
working on feature → urgent bug reported
→ git stash
→ switch to main → fix bug → commit
→ switch back to feature branch
→ git stash pop → continue where you left off
```

### pop vs apply
- `git stash pop` — restores and removes from stash list
- `git stash apply` — restores but keeps in stash list (useful for applying same stash to multiple branches)

---

## 7. Undoing Mistakes

### git restore — undo uncommitted changes
```bash
git restore filename.txt    # discard changes in working directory
```
WARNING: Permanent — changes are gone forever, cannot be recovered.

### git revert — undo a commit safely
```bash
git revert <commit-hash>    # creates a NEW commit that undoes the target commit
```
- History is preserved
- Safe for shared/public branches
- Teammates can see what happened

```
Before: A --- B --- C (mistake)
After:  A --- B --- C --- D (reverts C)
```

### git reset — undo commits (rewrites history)
```bash
git reset --soft HEAD~1     # undo commit, keep changes STAGED
git reset --mixed HEAD~1    # undo commit, keep changes UNSTAGED
git reset --hard HEAD~1     # undo commit, DELETE changes permanently
```
`HEAD~1` = one commit before current position

WARNING: Never use `git reset` on commits already pushed and shared with others. Use `git revert` instead.

### When to use each
| Command | Use case |
|---|---|
| `git restore` | Throw away uncommitted file changes |
| `git revert` | Safely undo a pushed/shared commit |
| `git reset --soft` | Undo commit but keep work to recommit differently |
| `git reset --hard` | Completely throw away last commit and all changes |

---

## 8. Forking & Cloning

### Forking
A fork is a personal copy of someone else's repository on your GitHub account.

```
Original Repo (someone else's)
        ↓ fork on GitHub
Your Copy on GitHub
        ↓ clone to local machine
Your Local Machine
        ↓ make changes → push to your fork
        ↓ open Pull Request to original repo
```

### Cloning
```bash
git clone https://github.com/USERNAME/REPO.git   # download repo to local machine
```

### upstream remote
After forking and cloning, add the original repo as `upstream` to stay in sync:
```bash
git remote add upstream https://github.com/ORIGINAL_OWNER/REPO.git
git remote -v               # should show both origin and upstream
```

Syncing with the original repo:
```bash
git fetch upstream          # download latest from original
git merge upstream/main     # apply to your local branch
git push                    # update your fork on GitHub
```

### Fork vs Clone
| | Fork | Clone |
|---|---|---|
| Where | GitHub to GitHub | GitHub to your machine |
| Purpose | Your own copy to contribute back | Work locally |

### origin vs upstream
- `origin` — your fork on GitHub (you have write access)
- `upstream` — the original repo you forked from (read only)

---

## 9. .gitignore

### What it does
Tells Git to completely ignore certain files or folders — they won't be tracked, staged, or committed.

### Common things to ignore
- Passwords / API keys / `.env` files
- Log files
- Dependencies (`node_modules`)
- OS files (`.DS_Store`)
- Build output

### How to use
Create a file called `.gitignore` in your repo root (must be UTF-8 encoded):
```
secret.txt          # ignore a specific file
*.log               # ignore all .log files
node_modules/       # ignore a folder
*.env               # ignore all .env files
```

### Important
- Always commit `.gitignore` so your whole team shares the same ignore rules
- `.gitignore` only works on untracked files — if a file is already committed, you need to `git rm --cached filename` first

---

## 10. git fetch vs git pull

```
git fetch  → downloads changes but does NOT apply them
git pull   → downloads AND immediately applies them (fetch + merge)

git pull = git fetch + git merge
```

### When to use each
| | Use when |
|---|---|
| `git pull` | You trust the changes and want them immediately |
| `git fetch` | You want to review changes before applying |

### Review workflow after git fetch
```bash
git fetch origin
git log HEAD..origin/main --oneline   # see incoming commits
git diff HEAD origin/main             # see incoming line changes
git merge origin/main                 # apply when ready
```

---

## 11. Rebasing

An alternative to merging. Rewrites history to create a clean straight line.

### Merge vs Rebase
```
Merge:
main:     A --- B ------- M
               \         /
feature:        C --- D

Rebase:
main:     A --- B --- C' --- D'   (clean straight line)
```

### Commands
```bash
git checkout feature-branch
git rebase main               # rebase feature onto main
git rebase --continue         # after resolving a conflict
git rebase --abort            # cancel the rebase
```

### Merge vs Rebase comparison
| Merge | Rebase |
|---|---|
| Preserves full history | Creates clean linear history |
| Adds a merge commit | No extra commits |
| Safe for shared branches | NEVER use on shared/public branches |

### Golden Rule
Never rebase commits that have already been pushed and shared with others — it rewrites history and causes problems for teammates.

**When in doubt, use merge.**

---

## 12. Tags

Bookmarks on specific commits. Used to mark important points in history — most commonly version releases.

### Commands
```bash
git tag                             # list all tags
git tag -a v1.0.0 -m "First release"  # create annotated tag (recommended)
git show v1.0.0                     # see tag details
git push origin v1.0.0              # push tag to GitHub
git push origin --tags              # push all tags
git checkout v1.0.0                 # go back to that exact snapshot
```

### Semantic Versioning (SemVer)
```
v MAJOR . MINOR . PATCH
  v1     .  1   .  0

MAJOR → breaking changes (not backwards compatible)
MINOR → new features, backwards compatible
PATCH → bug fixes
```

### Real World Use Cases
- Mark production deployments — instantly know what code is running
- Open source libraries — users install specific versions (`npm install react@18.0.0`)
- Easy rollback — `git checkout v1.0.0` to go back to a known good state

---

## 13. GitHub Actions

Automation tool built into GitHub. Runs automated tasks whenever something happens in your repo.

### Common Use Cases
```
push code     → automatically run tests
merge to main → automatically deploy to production
open a PR     → automatically check code quality
create a tag  → automatically build and release
```

### CI/CD
- **CI** (Continuous Integration) — automatically test code on every push
- **CD** (Continuous Deployment) — automatically deploy when tests pass

### How it works
Create a `.yml` file inside `.github/workflows/` folder:

```yaml
name: Hello World

on: [push]              # trigger: runs on every push

jobs:
  say-hello:
    runs-on: ubuntu-latest   # spin up a Linux virtual machine
    steps:
      - name: Print hello
        run: echo "Hello from GitHub Actions!"
```

### Real World Example
```yaml
name: Run Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3    # download your code
      - name: Run tests
        run: npm test                # run your test suite
```

---

## Quick Reference Cheat Sheet

```bash
# Setup
git config --global user.name "Name"
git config --global user.email "email"

# Core workflow
git status
git add .
git commit -m "message"
git push
git pull

# Branching
git checkout -b branch-name
git merge branch-name
git branch -d branch-name       # delete local branch

# Undoing
git restore file.txt            # undo uncommitted changes
git revert <hash>               # undo a commit safely
git reset --soft HEAD~1         # undo commit, keep changes staged
git reset --hard HEAD~1         # undo commit, delete changes

# Stash
git stash
git stash list
git stash pop

# Remote
git remote -v
git remote add origin <url>
git remote add upstream <url>
git fetch
git push origin branch-name
git push -u origin branch-name  # set upstream tracking

# Tags
git tag -a v1.0.0 -m "message"
git push origin v1.0.0

# History
git log --oneline
git log --oneline --graph --all
git diff HEAD origin/main
```
