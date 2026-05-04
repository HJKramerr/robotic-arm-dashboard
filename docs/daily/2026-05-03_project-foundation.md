# 2026-05-03 — Project Foundation

> Setting up the project directory, Python environment, git repository, and GitHub remote. No code yet — this session is about laying groundwork that everything else will sit on top of.

---

## Architecture Decisions

Before any setup, the high-level shape of the project was decided. Three layers, each with a specific job:

- **Arduino layer** — minimal firmware. Listens on serial, drives servos, reports back. No higher-level logic.
- **Controller layer (Python, on the Pi)** — the "brain." Holds robot state, manages serial communication, handles taught points, runs programs, will eventually do kinematics. Importable as a Python package, usable independently of any UI.
- **Dashboard layer (Python, on the Pi)** — the web interface. A view into the controller, never the source of truth.

The point of this separation: the controller can be driven from a script, REPL, or any future interface (CLI, voice, game controller) without rewriting robot logic. Tangling robot logic into UI button handlers makes future changes painful.

**On hobby servo position tracking:** standard hobby servos are open-loop from the host's perspective. They don't report position. "Current position" in the controller really means "commanded position" — the last angle we told the servo to go to. Mitigations: define a home pose and move there at startup, move slowly to reduce stress on parts and power, enforce soft limits in software.

---

## Setup Steps

### 1. Create the project directory

```bash
mkdir -p ~/projects && cd ~/projects
mkdir robotic-arm-dashboard && cd robotic-arm-dashboard
```

**Why:** convention is to keep all repos under a single parent directory (`~/projects` or `~/code`). Easier to find, easier to back up.

### 2. Check Python version

```bash
python3 --version
python3 -m venv --help
```

**Why:** confirms Python is installed and the `venv` module is available. On this Pi: Python 3.13.5, venv working. Some Pi OS images don't ship `venv` by default — would need `sudo apt install python3-venv` if missing.

### 3. Create and activate a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

**Why:** a virtual environment is an isolated Python install just for this project. Anything `pip install`'d goes into `.venv/`, not the system Python. Keeps project dependencies clean and reproducible.

After activation, the prompt shows `(.venv)` to indicate the environment is active. To leave: `deactivate`. Must be re-activated in every new SSH session before working on the project.

Verify isolation:
```bash
which python    # should point inside .venv/bin/
```

### 4. Create the directory layout

```bash
mkdir -p arm_controller dashboard arduino docs/daily tests scripts
touch arm_controller/__init__.py dashboard/__init__.py
touch README.md requirements.txt .gitignore
```

**Why each piece:**

- `arm_controller/` and `dashboard/` are Python *packages* — the empty `__init__.py` files are what make them importable. Enables `from arm_controller import Controller` later.
- `arduino/` holds `.ino` firmware files. Keeping them in the same repo means firmware and software version together.
- `docs/daily/` is for daily progress logs (this file).
- `tests/` for tests of the controller logic.
- `scripts/` for one-off utilities (calibration, diagnostics, etc.).
- `README.md` is the front door of the repo on GitHub.
- `requirements.txt` will list Python dependencies as they're added.
- `.gitignore` tells git what *not* to track.

### 5. Add `.gitkeep` files for empty directories

```bash
touch arduino/.gitkeep docs/daily/.gitkeep tests/.gitkeep scripts/.gitkeep
```

**Why:** git tracks files, not directories. An empty directory is invisible to git. The `.gitkeep` file (name is convention only — it has no special meaning to git) gives each directory at least one file so the structure gets committed.

### 6. Set up `.gitignore`

```bash
curl -o .gitignore https://raw.githubusercontent.com/github/gitignore/main/Python.gitignore
```

**Why:** `.gitignore` is a list of patterns git skips entirely. Critical to set up *before* the first commit because anything committed is much harder to truly remove later. GitHub maintains a comprehensive Python `.gitignore` covering venvs, bytecode, packaging artifacts, IDE files, etc.

Then add project-specific patterns by editing the file:

```bash
nano .gitignore
```

Append at the bottom:

```
# Project-specific
data/
logs/
*.log

# Arduino build artifacts
arduino/build/
arduino/*/build/

# Local config / secrets
config.local.*
```

Save with `Ctrl+O`, `Enter`, then exit with `Ctrl+X`.

**Note:** `.gitignore` only affects files not *already* tracked. If something gets committed and *then* added to `.gitignore`, git keeps tracking it. Order matters: gitignore first, commit second.

### 7. Write the README

A first-pass `README.md` was drafted covering project name, status, goals (as a numbered milestone list), hardware, architecture summary, and project structure. The README is the landing page on GitHub — worth being intentional about even early on. It can (and should) evolve.

Useful: `cat README.md` to view it in the terminal.

### 8. Initialize the git repository

```bash
git init
git branch -m main
```

**Why:**

- `git init` creates a hidden `.git/` directory containing all of git's tracking data. The directory is now a repo.
- `git branch -m main` renames the default branch from `master` to `main`. Modern convention. Can be set globally for future repos with `git config --global init.defaultBranch main`.

### 9. Verify what git sees

```bash
git status
```

**Why:** before adding anything, this shows what git considers untracked. Critical sanity check: confirms `.venv/` is being ignored (it shouldn't appear) and the right files are visible.

Make `git status` a habit before every `git add` and every `git commit`. It's the preview of what's about to happen.

Useful debugging command: `git check-ignore -v <filename>` — tells you specifically why a file is being ignored and which rule did it.

### 10. Configure git identity (one-time, global)

```bash
git config --global user.name "your-name"
git config --global user.email "your@email.com"
```

**Why:** every commit is stamped with author info. The email should match the GitHub account for commits to be linked to the profile.

Already configured on this Pi from previous work — verified with:
```bash
git config --global user.name
git config --global user.email
```

### 11. First commit

```bash
git add .
git status                 # verify what's staged
git commit -m "Initial project structure and documentation"
```

**Why:**

- `git add .` stages all changes (respecting `.gitignore`).
- The follow-up `git status` shows what's actually staged — last chance to catch something unintended.
- `git commit -m "..."` records the staged changes as a commit with the given message.

Result: 9 files committed (gitignore, README, requirements.txt, the two `__init__.py` files, four `.gitkeep` files), 291 lines.

### 12. Generate an SSH key for GitHub

```bash
ls -la ~/.ssh/                              # check if a key already exists
ssh-keygen -t ed25519 -C "your@email.com"
```

**Why:**

- SSH keys let git push/pull from GitHub without typing credentials every time.
- Ed25519 is a modern, secure algorithm. GitHub's recommended choice.
- The `-C` flag adds a comment (typically the email) to identify the key later.

Prompts:
1. **File location** — accept default (`~/.ssh/id_ed25519`).
2. **Passphrase** — optional. Adds an extra layer of security; will be prompted on every push unless cached with `ssh-agent`. Blank is acceptable on a personal-use Pi.

Result: two files in `~/.ssh/`:
- `id_ed25519` — **private** key. Never share or move off the Pi.
- `id_ed25519.pub` — public key. Safe to share; this goes to GitHub.

### 13. Add the public key to GitHub

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire output (one line starting with `ssh-ed25519`).

In the browser:
1. GitHub → profile picture → **Settings**
2. Sidebar → **SSH and GPG keys** → **New SSH key**
3. Title: descriptive name (e.g., `streampi`)
4. Paste the public key
5. **Add SSH key**

### 14. Test the GitHub SSH connection

```bash
ssh -T git@github.com
```

First time: prompts to trust GitHub's host key — answer `yes`.

Successful output: `Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.` The "no shell access" line is expected — GitHub doesn't allow logging into their servers, only authenticating for git operations.

### 15. (Optional) Cache the SSH passphrase for the session

If a passphrase was set on the key, every `git push` will prompt for it. Cache it for the session:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Enter the passphrase once when prompted. It stays cached until logout. To make this automatic across sessions is a follow-up task.

### 16. Create the empty repo on GitHub

In the browser:
1. GitHub → **New repository**
2. Name: matches the local directory name (`robotic-arm-dashboard`)
3. Public or private — personal preference
4. **Do NOT** initialize with README, `.gitignore`, or license — these already exist locally and would create a merge conflict on first push.

### 17. Connect the local repo to GitHub and push

```bash
git remote add origin git@github.com:<username>/<repo>.git
git remote -v                  # verify the remote is set
git push -u origin main
```

**Why:**

- `git remote add origin <url>` registers a remote repository under the name `origin` (conventional name for the main remote).
- `git remote -v` lists configured remotes and their URLs.
- `git push -u origin main` pushes the `main` branch to the `origin` remote. The `-u` flag sets `origin/main` as the upstream — only needed the first time. Future pushes on this branch can just be `git push`.

After this, the project is live on GitHub at `https://github.com/<username>/<repo>`.

---

## What's Done

- Project directory and Python virtual environment in place
- Directory structure reflecting the three-layer architecture
- `.gitignore` configured with Python defaults and project-specific additions
- Starter README written
- Local git repo initialized with first commit
- SSH key generated and added to GitHub
- Remote configured and pushed — project is on GitHub

## Open Items / Next Up

- Decide on daily-PR workflow (branches vs commits to main, PR conventions)
- Stub out the controller class skeleton in Python — methods, state, no serial yet
- Draft the serial protocol spec in `docs/protocol.md`
- Confirm servo physical range with a test sketch once Arduino is available

## Notes / Reminders

- Activate the venv at the start of every session: `source .venv/bin/activate`
- If the SSH passphrase prompt is annoying, run `eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519` after connecting
- Make `git status` a reflex before any `git add` or `git commit`
- The README's directory tree needs a code fence (triple backticks) added in the next commit — current rendering on GitHub will be broken without it
