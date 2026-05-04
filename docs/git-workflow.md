# Git Workflow Cheat Sheet

Quick reference for the daily PR flow. When in doubt, run `git status` — it tells you where you are and what's happening.

---

## Start of Every Session

```bash
cd ~/projects/robotic-arm-dashboard
source .venv/bin/activate                          # activate Python venv
eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519   # cache SSH passphrase
git checkout main
git pull
```

---

## The Daily PR Loop

### 1. Start a new branch

```bash
git checkout main
git pull
git checkout -b <type>/<short-description>
```

Branch name conventions:
- `feature/serial-protocol` — new functionality
- `fix/readme-tree-rendering` — bug fix
- `docs/daily-2026-05-03` — documentation only

### 2. Do the work

```bash
# edit files, create files, etc.
git status                  # what changed?
git diff                    # what changed inside files?
git add <file>              # stage specific file
git add .                   # stage everything (respects .gitignore)
git diff --staged           # what's about to be committed?
git commit -m "Imperative message under 50 chars"
```

Commit message style:
- Imperative mood: "Add controller skeleton" not "Added"
- Under ~50 characters for the summary line
- Describe *what* and *why*, not *how*

### 3. Push the branch

```bash
git push -u origin <branch-name>      # first push only (-u sets upstream)
git push                              # subsequent pushes
```

### 4. Open the PR on GitHub

- Push output prints a URL — open it, or go to repo and click "Compare & pull request"
- Title: same rules as commit messages
- Description: explain *why* the change exists (your future self will thank you)
- Click **Create pull request**

### 5. Review and merge

- Click **Files changed** tab — read the diff
- Scroll down on **Conversation** tab to merge button
- Click the **dropdown arrow** next to the green button
- Choose **Squash and merge** (preferred for solo work)
- Confirm
- Click **Delete branch** when offered

### 6. Clean up locally

```bash
git checkout main
git pull
git branch -d <branch-name>           # safe delete, refuses if unmerged
```

---

## Reflex Commands

Make these automatic — run them constantly:

| Command | When | What it shows |
|---------|------|---------------|
| `git status` | Before any action | Branch, staged files, unstaged changes |
| `git diff` | Before staging | What changed inside files |
| `git diff --staged` | Before committing | What's about to commit |
| `git log --oneline` | When lost | Commit history, one line each |
| `git branch` | When lost | All local branches, * marks current |

---

## Navigation Helpers

For when you can't remember where things are:

```bash
pwd                                    # where am I?
ls                                     # what's here?
ls -la                                 # what's here, including hidden files
tree -I '.venv|__pycache__'            # full project tree
find . -name "*daily*"                 # find files matching pattern
```

Tab completion: start typing a filename or path, hit `Tab` to autocomplete.
Hit `Tab` twice to see options if ambiguous.

---

## "I Messed Up" Recovery

Almost nothing is permanent in git.

```bash
# Wrong branch — moved changes I haven't committed yet
git stash                              # stash uncommitted changes
git checkout correct-branch
git stash pop                          # reapply changes here

# Committed to wrong branch
git reset --soft HEAD~1                # undo last commit, keep changes staged
# then checkout correct branch and commit there

# Want to throw away uncommitted changes
git checkout -- <file>                 # discard changes to one file
git reset --hard                       # discard ALL uncommitted changes (careful!)

# Deleted a branch I needed
git reflog                             # shows recent HEAD movements
# find the commit hash, then:
git checkout -b recovered-branch <hash>

# Pushed something I shouldn't have
# Don't panic. Make a new commit that fixes it. Force-pushing rewrites
# history and is a bad habit on shared branches.
```

When something feels broken, **stop and run `git status`**. Don't make it worse with random commands.

---

## Conventions Used in This Project

- **Default branch**: `main`
- **Branch prefixes**: `feature/`, `fix/`, `docs/`
- **Daily logs**: `docs/daily/YYYY-MM-DD_tagline.md`
- **Merge style**: Squash and merge
- **Commit style**: Imperative, under 50 char summary
