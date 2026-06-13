# Git Commands — Detailed Explanation & Interview Guide

This document explains every command from the [README.md](README.md) cheat sheet in depth. Each section includes **what the command does**, **how it works internally**, **real-world scenarios**, and **interview Q&A** so you can both use Git confidently and answer questions about it.

---

## Table of Contents

- [Core Concepts](#core-concepts)
- [Setup](#setup)
- [SSH Setup](#ssh-setup)
- [Start a Repository](#start-a-repository)
- [Daily Workflow](#daily-workflow)
- [Branching](#branching)
- [Remotes](#remotes)
- [Inspecting History](#inspecting-history)
- [Undoing Changes](#undoing-changes)
- [Tags and Releases](#tags-and-releases)
- [Collaboration](#collaboration)
- [Cleanup](#cleanup)
- [Quick Reference](#quick-reference)
- [Interview Questions — Rapid Fire](#interview-questions--rapid-fire)

---

## Core Concepts

Before diving into commands, understand these foundational ideas. Interviewers test these constantly.

### The Three Areas of Git

```
Working Directory  ──git add──▸  Staging Area (Index)  ──git commit──▸  Local Repository
        ▲                                                                      │
        └──────────────────── git checkout / restore ◂──────────────────────────┘
```

| Area | What lives here |
|---|---|
| **Working Directory** | The actual files on your disk that you edit. |
| **Staging Area (Index)** | A snapshot of what will go into the next commit. Think of it as a "loading dock" — you choose exactly which changes ship. |
| **Local Repository (.git)** | The full history of commits, branches, tags, and config stored inside the hidden `.git` folder. |

### Snapshots, Not Diffs

Git stores **snapshots** of every file at each commit, not line-by-line diffs. Internally it uses content-addressable storage (SHA-1 hashes) and packs unchanged files as pointers to previous blobs, so it is very space-efficient despite storing full snapshots.

### HEAD, Branches, and Commits

- **Commit**: An immutable snapshot identified by a 40-character SHA-1 hash.
- **Branch**: A lightweight movable pointer to a commit. Creating a branch is nearly instant (it just writes 41 bytes — the hash — to a file).
- **HEAD**: A special pointer that tells Git "you are here." It usually points to a branch name, which in turn points to a commit.

```
HEAD ──▸ main ──▸ commit abc123
```

### Interview Q&A — Core Concepts

> **Q: What is the difference between Git and GitHub?**
> **A:** Git is a distributed version control system that runs locally on your machine. GitHub is a cloud hosting service that provides a remote repository, pull requests, issues, CI/CD, and collaboration features on top of Git.

> **Q: What does "distributed" mean in DVCS?**
> **A:** Every developer has a full copy of the entire repository history locally. You can commit, branch, merge, and view history completely offline. There is no single point of failure — any clone can restore the full project.

> **Q: What is the SHA-1 hash in Git?**
> **A:** Every Git object (commit, tree, blob, tag) is identified by a 40-character SHA-1 hash computed from its content. This ensures data integrity — if even one bit changes, the hash changes entirely.

---

## Setup

### `git config --global user.name "Your Name"`

**What it does:** Sets the author name that will be embedded in every commit you make on this machine.

**How it works:** Git writes this value to `~/.gitconfig` (your global config file). When you commit, Git reads this value and records it in the commit metadata so teammates know who wrote the code.

**Scope levels:**
| Flag | Scope | Config file |
|---|---|---|
| `--system` | All users on the machine | `/etc/gitconfig` |
| `--global` | Your user account | `~/.gitconfig` |
| `--local` (default) | Current repository only | `.git/config` |

Local overrides global, which overrides system.

### `git config --global user.email "your_email@example.com"`

**What it does:** Sets the email address written into commit metadata. GitHub uses this email to link commits to your profile — make sure it matches your GitHub account email.

### `git config --global init.defaultBranch main`

**What it does:** When you run `git init`, the first branch will be called `main` instead of the legacy default `master`.

**Why it matters:** Many teams and platforms (GitHub, GitLab) have adopted `main` as the default. Setting this avoids confusion when you push a new repo.

### `git config --global pull.rebase false`

**What it does:** Tells `git pull` to use **merge** strategy (the traditional default) rather than **rebase**.

**Merge vs. Rebase on pull:**
- **Merge** (`pull.rebase false`): Creates a merge commit that joins your local and remote changes. Preserves the exact history of when things happened.
- **Rebase** (`pull.rebase true`): Replays your local commits on top of the remote commits. Produces a cleaner, linear history but rewrites commit hashes.

### `git config --global fetch.prune true`

**What it does:** Automatically removes remote-tracking branches that no longer exist on the remote whenever you `git fetch`. Without this, stale branches like `origin/feature-old` pile up forever.

### `git config --global core.autocrlf true`

**What it does:** On Windows, converts line endings to CRLF (`\r\n`) in your working directory and back to LF (`\n`) when committing. This prevents messy diffs caused by different OSes using different line endings.

| OS | Working directory | Repository |
|---|---|---|
| Windows (`true`) | CRLF | LF |
| Mac/Linux (`input`) | LF | LF |

### Interview Q&A — Setup

> **Q: How do you check all your current Git config settings?**
> **A:** `git config --list --show-origin` — this shows every setting and which file it comes from.

> **Q: Can you have different Git identities for work and personal projects?**
> **A:** Yes. Use `git config --local user.email "work@company.com"` inside a work repo. Or use conditional includes in `~/.gitconfig`:
> ```ini
> [includeIf "gitdir:~/work/"]
>     path = ~/.gitconfig-work
> ```

---

## SSH Setup

SSH lets you authenticate with GitHub/GitLab without typing your password every time.

### `ssh-keygen -t ed25519 -C "your_email@example.com"`

**What it does:** Generates a key pair:
- **Private key** (`~/.ssh/id_ed25519`) — stays on your machine, never share this.
- **Public key** (`~/.ssh/id_ed25519.pub`) — you upload this to GitHub/GitLab.

**Why Ed25519?** It's faster, more secure, and produces shorter keys than RSA. It's the recommended algorithm unless you need compatibility with very old systems.

**What `-C` does:** Adds a comment (typically your email) to the public key for identification. It doesn't affect cryptographic security.

### `eval "$(ssh-agent -s)"` (Linux)

**What it does:** Starts the SSH agent as a background process. The agent holds your decrypted private key in memory so you don't re-enter your passphrase every time you push or pull.

### `ssh-add ~/.ssh/id_ed25519` (Linux)

**What it does:** Loads your private key into the running SSH agent. If your key has a passphrase, you type it once here and the agent remembers it for the session.

### `cat ~/.ssh/id_ed25519.pub` (Linux)

**What it does:** Prints your public key to the terminal. Copy this entire output and paste it into GitHub → Settings → SSH Keys.

### `ssh -T git@github.com`

**What it does:** Tests your SSH connection to GitHub. A successful response looks like:

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

If it fails, common causes are: key not added to agent, wrong key uploaded to GitHub, or firewall blocking port 22.

### `Start-Service ssh-agent` (Windows PowerShell)

**What it does:** Starts the OpenSSH Authentication Agent Windows service. You may need to run PowerShell as Administrator if the service is set to Manual or Disabled.

### `ssh-add $env:USERPROFILE\.ssh\id_ed25519` (Windows)

**What it does:** Same as the Linux version but uses the Windows environment variable `$env:USERPROFILE` (which resolves to `C:\Users\YourName`).

### `Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub` (Windows)

**What it does:** PowerShell equivalent of `cat` — prints the public key so you can copy it.

### `git clone git@github.com:owner/repository.git`

**What it does:** Clones a repository using the SSH protocol instead of HTTPS. The `git@github.com:` prefix tells Git to use SSH authentication.

**SSH vs HTTPS:**
| Feature | SSH | HTTPS |
|---|---|---|
| Authentication | Key pair (no password) | Username + token/password |
| Setup effort | One-time key setup | Token per session/machine |
| Firewall friendliness | Port 22 (sometimes blocked) | Port 443 (almost never blocked) |

### `git remote set-url origin git@github.com:owner/repository.git`

**What it does:** Switches an existing repository's remote URL from HTTPS to SSH (or vice versa) without re-cloning.

### Interview Q&A — SSH

> **Q: What's the difference between SSH and HTTPS for Git authentication?**
> **A:** SSH uses public-key cryptography — you generate a key pair, upload the public key to GitHub, and authenticate without a password. HTTPS uses a personal access token or password. SSH is more convenient for frequent pushes; HTTPS works better behind corporate firewalls that block port 22.

> **Q: What do you do if `ssh -T git@github.com` says "Permission denied (publickey)"?**
> **A:** 1) Check that your key is loaded: `ssh-add -l`. 2) Verify the correct public key is added to GitHub. 3) Check `~/.ssh/config` for wrong IdentityFile settings. 4) Try verbose mode: `ssh -vT git@github.com` for debugging.

---

## Start a Repository

### `git init`

**What it does:** Creates a new `.git` directory in the current folder, which initializes a brand-new Git repository. This `.git` folder contains all the internal structures: `objects/`, `refs/`, `HEAD`, `config`, etc.

**When to use:** When starting a new project from scratch locally.

**What it does NOT do:** It does not create a remote repository. You need to separately create one on GitHub/GitLab and link it with `git remote add`.

### `git clone <repository_url>`

**What it does:** Downloads a complete copy of a remote repository — all commits, branches, tags, and history — and sets up `origin` as the remote automatically.

**Under the hood:**
1. Creates a new directory named after the repo
2. Runs `git init` inside it
3. Runs `git remote add origin <url>`
4. Runs `git fetch --all`
5. Checks out the default branch

**Useful variations:**
| Command | Purpose |
|---|---|
| `git clone --depth 1 <url>` | Shallow clone — only the latest commit (faster for CI) |
| `git clone --branch <name> <url>` | Clone and check out a specific branch |
| `git clone --recurse-submodules <url>` | Also clone embedded submodules |

### `git remote add origin <repository_url>`

**What it does:** Registers a remote repository under the alias `origin`. The name `origin` is a convention — you could call it anything, but nearly every Git tool expects `origin` as the primary remote.

**When to use:** After `git init` when you want to connect your local repo to a remote hosted on GitHub, GitLab, Bitbucket, etc.

### `git add .`

**What it does:** Stages **all** changes in the current directory and subdirectories — new files, modified files, and deleted files — into the staging area (index).

**Key detail:** `git add .` is relative to your current working directory. If you're inside a subdirectory, it only stages changes under that subdirectory. Use `git add -A` (or `git add --all`) to stage everything repo-wide regardless of your current directory.

### `git commit -m "Initial commit"`

**What it does:** Takes everything in the staging area, creates a new commit object with a unique SHA-1 hash, and moves the current branch pointer forward to this new commit.

**Anatomy of a commit object:**
```
tree      a1b2c3...    (snapshot of the file tree)
parent    d4e5f6...    (previous commit — empty for first commit)
author    Your Name <email> timestamp
committer Your Name <email> timestamp
message   Initial commit
```

### `git push -u origin main`

**What it does:** Pushes the local `main` branch to the `origin` remote. The `-u` (short for `--set-upstream`) flag creates a tracking relationship, so future `git push` and `git pull` from this branch won't need extra arguments.

**What "tracking" means:** After `-u`, your local `main` knows it corresponds to `origin/main`. Running `git status` will then say things like "Your branch is ahead of 'origin/main' by 2 commits."

### Interview Q&A — Start a Repository

> **Q: What's the difference between `git init` and `git clone`?**
> **A:** `git init` creates a brand-new empty repository. `git clone` copies an existing remote repository with all its history. If the project already exists remotely, use `clone`. If you're starting from scratch, use `init`.

> **Q: What is a shallow clone and when would you use it?**
> **A:** `git clone --depth 1` downloads only the latest commit, not full history. It's used in CI/CD pipelines to speed up builds when you don't need history. Downside: you can't view old commits or do meaningful diffs without unshallowing.

---

## Daily Workflow

### `git status`

**What it does:** Shows the state of your working directory and staging area. It tells you:
- Which files are **modified** but not yet staged
- Which files are **staged** and ready to commit
- Which files are **untracked** (new files Git doesn't know about)

This is the most-used Git command. Run it frequently to stay oriented.

### `git status -sb`

**What it does:** Short-form status with branch info. Example output:
```
## main...origin/main [ahead 2]
 M README.md
?? new-file.txt
A  staged-file.txt
```

| Symbol | Meaning |
|---|---|
| `M` (right column) | Modified, not staged |
| `M` (left column) | Modified, staged |
| `A` | New file, staged |
| `??` | Untracked file |
| `D` | Deleted |

### `git add <file>`

**What it does:** Stages a specific file. The file's current content is copied into the staging area.

**Key insight:** If you modify a file, stage it, then modify it again, the second modification is **not** automatically staged. You'd need to `git add` it again. The staging area is a snapshot, not a live reference.

### `git commit -m "Describe the change"`

**What it does:** Creates a commit from the staging area with a descriptive message.

**Writing good commit messages:**
```
feat: add user login endpoint              ← type: short summary (imperative mood)
                                             ← blank line
Adds POST /api/login with JWT token         ← body: explain WHY, not WHAT
generation. Closes #42.                      ← reference issues
```

Common prefixes (Conventional Commits): `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `ci`.

### `git commit --amend`

**What it does:** Replaces the most recent commit with a new one. Use cases:
1. **Fix a typo in the commit message:** `git commit --amend -m "corrected message"`
2. **Add a forgotten file:** `git add forgotten.txt && git commit --amend --no-edit`

**⚠️ Warning:** Amending rewrites history (creates a new SHA). Never amend commits that have already been pushed to a shared branch unless you force-push and coordinate with your team.

### Interview Q&A — Daily Workflow

> **Q: What is the staging area and why does Git have it?**
> **A:** The staging area (or index) lets you selectively choose which changes to include in a commit. Without it, every commit would have to include ALL your changes. With it, you can make 10 changes to 5 files but commit only the 3 related changes, keeping commits focused and reviewable.

> **Q: What's the difference between `git add .` and `git add -A`?**
> **A:** `git add .` stages new, modified, and deleted files **in the current directory and below**. `git add -A` (or `--all`) stages changes across the **entire repository** regardless of your current directory. In the repo root, they behave the same.

> **Q: Can you undo a `git commit --amend`?**
> **A:** Yes. Use `git reflog` to find the original commit's SHA before the amend, then `git reset --hard <original-sha>` to go back. The reflog keeps references for at least 30 days.

---

## Branching

### `git branch <branch_name>`

**What it does:** Creates a new branch pointer at the current commit. It does **not** switch to the new branch — you stay on whatever branch you were on.

**Under the hood:** Git simply creates a file at `.git/refs/heads/<branch_name>` containing the current commit's SHA. That's it. Branching is essentially free.

### `git switch <branch_name>`

**What it does:** Switches your working directory to the specified branch. HEAD moves to point at the new branch, and your files are updated to match that branch's latest commit.

**`switch` vs `checkout`:** `git switch` was introduced in Git 2.23 to be a clearer, safer alternative to `git checkout`. `checkout` does too many things (switch branches, restore files, detach HEAD). `switch` only switches branches.

### `git switch -c <branch_name>`

**What it does:** Creates a new branch **and** switches to it in one step. Equivalent to:
```bash
git branch <branch_name>
git switch <branch_name>
```

### `git merge <branch_name>`

**What it does:** Combines the changes from `<branch_name>` into your current branch.

**Types of merge:**

| Type | When | What happens |
|---|---|---|
| **Fast-forward** | Your branch hasn't diverged | Branch pointer simply moves forward; no merge commit |
| **Three-way merge** | Both branches have new commits | Git creates a merge commit with two parents |
| **Conflict** | Same lines changed in both branches | Git pauses and asks you to resolve manually |

**Merge conflict workflow:**
```bash
git merge feature          # Conflict detected
# Edit conflicted files, remove <<<< ==== >>>> markers
git add <resolved-files>
git commit                 # Completes the merge
```

### `git branch -d <branch_name>`

**What it does:** Safely deletes a branch that has been **fully merged** into the current branch. Git refuses to delete if the branch has unmerged work, protecting you from accidental data loss.

### `git branch -D <branch_name>`

**What it does:** Force-deletes a branch regardless of merge status. Use this when you're certain you want to discard the work on that branch.

### `git branch -vv`

**What it does:** Lists all local branches with:
- The latest commit hash and message
- The upstream tracking branch (e.g., `origin/main`)
- Whether you're ahead, behind, or diverged

Example output:
```
  feature  abc1234 [origin/feature: ahead 2] Add login form
* main     def5678 [origin/main] Merge pull request #10
  hotfix   789abcd Fix typo
```

### Interview Q&A — Branching

> **Q: What is a branch in Git, internally?**
> **A:** A branch is a lightweight pointer (a 41-byte file) that points to a commit SHA. Creating a branch doesn't copy any files or history. That's why Git branching is nearly instant, unlike older VCS tools like SVN that copy the entire directory tree.

> **Q: What is a fast-forward merge?**
> **A:** When the current branch has no new commits since the branch point, Git can simply move the pointer forward to the target branch's latest commit. No merge commit is created. It's as if the work was done directly on the current branch.

> **Q: How do you resolve a merge conflict?**
> **A:** 1) Run `git merge`. 2) Git marks conflicted files with `<<<<<<<`, `=======`, `>>>>>>>` markers. 3) Edit each file to keep the correct code. 4) `git add` the resolved files. 5) `git commit` to complete the merge. Tools like VS Code, IntelliJ, or `git mergetool` provide visual conflict resolution.

> **Q: What's the difference between `git switch` and `git checkout`?**
> **A:** `git switch` (Git 2.23+) is focused: it only switches branches. `git checkout` is overloaded — it can switch branches, restore files, detach HEAD, and more. `switch` + `restore` split `checkout`'s responsibilities for clarity and safety.

> **Q: What branching strategy do you use?**
> **A:** Common strategies:
> - **Git Flow**: `main`, `develop`, `feature/*`, `release/*`, `hotfix/*` — good for scheduled releases.
> - **GitHub Flow**: `main` + short-lived feature branches + pull requests — simple, great for continuous delivery.
> - **Trunk-Based Development**: Everyone commits to `main` with short-lived branches (< 1 day) — used by high-velocity teams like Google.

---

## Remotes

### `git remote -v`

**What it does:** Lists all configured remotes with their fetch and push URLs.

```
origin  git@github.com:user/repo.git (fetch)
origin  git@github.com:user/repo.git (push)
```

You can have multiple remotes (e.g., `origin` for your fork, `upstream` for the original repo).

### `git fetch`

**What it does:** Downloads new commits, branches, and tags from the remote into your local repository **without changing your working directory or current branch**. It updates your remote-tracking branches (e.g., `origin/main`).

**Key insight:** `fetch` is always safe. It never modifies your local branches or files. It just updates your knowledge of what's on the remote.

### `git fetch --prune`

**What it does:** Same as `git fetch`, but also removes remote-tracking branches that no longer exist on the remote. For example, if someone deleted `feature/old` on GitHub, this removes your local `origin/feature/old` reference.

### `git push origin <branch_name>`

**What it does:** Uploads your local branch's commits to the remote. If the remote branch doesn't exist yet, it's created.

**When push is rejected:** If someone else has pushed to the same branch since your last pull, Git rejects your push to prevent overwriting their work. You must `pull` first, resolve any conflicts, then push again.

### `git pull`

**What it does:** This is actually **two commands in one**:
```
git fetch + git merge
```
It downloads remote changes and immediately merges them into your current branch.

**Potential gotcha:** `git pull` can create unexpected merge commits if your local branch has diverged from the remote. Many teams prefer `git pull --rebase` for a cleaner history.

### `git branch --set-upstream-to=origin/<branch_name>`

**What it does:** Links your current local branch to a remote tracking branch. After this, `git push` and `git pull` know where to push/pull from without specifying the remote and branch name every time.

### Interview Q&A — Remotes

> **Q: What's the difference between `git fetch` and `git pull`?**
> **A:** `git fetch` downloads remote changes but doesn't modify your working tree or local branches — it's a safe "look but don't touch" operation. `git pull` does `fetch` + `merge` (or `fetch` + `rebase`), so it modifies your current branch. Always `fetch` if you want to inspect changes before integrating them.

> **Q: What is a remote-tracking branch?**
> **A:** It's a read-only local reference like `origin/main` that mirrors the state of a branch on the remote. It's updated by `git fetch`. You cannot commit directly to `origin/main` — it's a bookmark showing "the last known state of `main` on the remote."

> **Q: Can you have multiple remotes?**
> **A:** Yes. Common setup for open-source contribution:
> - `origin` → your fork
> - `upstream` → the original project
> You fetch from `upstream` to stay current, and push to `origin` for your PRs.

---

## Inspecting History

### `git log --oneline --graph --decorate --all`

**What it does:** Shows a compact, visual history of the entire repository.

| Flag | Purpose |
|---|---|
| `--oneline` | One line per commit (short SHA + message) |
| `--graph` | ASCII art showing branch and merge topology |
| `--decorate` | Shows branch names and tags next to commits |
| `--all` | Includes all branches, not just the current one |

Example output:
```
* abc1234 (HEAD -> main, origin/main) Merge pull request #5
|\
| * def5678 (feature) Add search bar
|/
* 789abcd Update README
* 111aaaa Initial commit
```

### `git show <commit_hash>`

**What it does:** Displays a single commit's metadata (author, date, message) and the full diff of what changed. You can use the full 40-character SHA or a short prefix (usually 7 characters).

### `git log -p`

**What it does:** Shows the commit log with the **full patch (diff)** for each commit. This is like running `git show` for every commit in the history. Useful for reviewing what changed over time, but can produce very long output.

### `git diff`

**What it does:** Shows **unstaged** changes — the difference between your working directory and the staging area.

### `git diff --staged`

**What it does:** Shows **staged** changes — the difference between the staging area and the last commit. These are the changes that will go into the next commit.

### `git diff <branch_name>`

**What it does:** Compares your current branch (working directory) with another branch tip.

### `git diff <commit_a>..<commit_b>`

**What it does:** Shows all changes between two specific commits.

**Two dots vs three dots:**
- `A..B` — Changes in B that are not in A (what B added)
- `A...B` — Changes on both sides since their common ancestor (what's different in both)

### `git blame <file>`

**What it does:** Shows who last modified each line in a file, with the commit hash and timestamp. Essential for understanding code ownership and tracking down when a bug was introduced.

```
abc1234 (Alice 2024-01-15) function login() {
def5678 (Bob   2024-02-20)   validateInput();
789abcd (Alice 2024-03-10)   return generateToken();
```

### Interview Q&A — Inspecting History

> **Q: How do you find which commit introduced a bug?**
> **A:** Use `git bisect`. It performs a binary search through your commit history:
> ```bash
> git bisect start
> git bisect bad              # current commit has the bug
> git bisect good abc1234     # this older commit was fine
> # Git checks out a middle commit — you test and mark good/bad
> # Repeat until Git identifies the exact commit
> git bisect reset
> ```

> **Q: What's the difference between `git diff` and `git diff --staged`?**
> **A:** `git diff` shows changes between the working directory and the staging area (unstaged changes). `git diff --staged` shows changes between the staging area and the last commit (what will be committed). Together they cover the full picture of uncommitted work.

> **Q: How do you search commit messages for a keyword?**
> **A:** `git log --grep="keyword"` searches commit messages. `git log -S"function_name"` (pickaxe) searches for commits that added or removed a specific string in the code.

---

## Undoing Changes

This is the most important section for interviews. Understanding undo operations shows you truly understand Git's architecture.

### `git restore --staged <file>`

**What it does:** Removes a file from the staging area but keeps the modifications in your working directory. Your changes are preserved — they're just no longer queued for the next commit.

**Before (staged):** `git status` shows the file as green/staged.
**After:** `git status` shows the file as red/modified (unstaged).

### `git restore <file>`

**What it does:** Discards local changes in a file and restores it to the version in the staging area (or the last commit if nothing is staged).

**⚠️ Warning:** This is destructive. The local changes are gone and cannot be recovered (they were never committed).

### `git revert <commit_hash>`

**What it does:** Creates a **new commit** that is the exact inverse of the specified commit. If commit X added 10 lines, `git revert X` creates commit Y that removes those 10 lines.

**Why this is safe:** It doesn't rewrite history. The original commit stays in the log. This makes it safe for shared/public branches.

**Example:**
```
Before: A → B → C (HEAD)
git revert B
After:  A → B → C → B' (HEAD)   ← B' undoes B's changes
```

### `git reset --soft <commit_hash>`

**What it does:** Moves the branch pointer back to the specified commit. Changes from the "removed" commits are kept **staged** (in the index). Your working directory is untouched.

**Use case:** Squash the last 3 commits into one:
```bash
git reset --soft HEAD~3
git commit -m "Combined feature"
```

### `git reset --mixed <commit_hash>`

**What it does:** (This is the **default** mode of `git reset`.) Moves the branch pointer back and **unstages** changes, but keeps them in the working directory.

**Use case:** You staged files by accident and want to redo which files are staged.

### `git reset --hard <commit_hash>`

**What it does:** Moves the branch pointer back and **discards everything** — staged changes and working directory modifications are both deleted.

**⚠️ This is the most dangerous Git command.** Changes that were never committed are unrecoverable. Changes that were committed can be recovered via `git reflog`.

### Summary: reset modes

```
                   Branch Pointer    Staging Area    Working Directory
--soft             ✅ Moves back     ❌ Unchanged    ❌ Unchanged
--mixed (default)  ✅ Moves back     ✅ Cleared      ❌ Unchanged
--hard             ✅ Moves back     ✅ Cleared       ✅ Cleared
```

### `git reflog`

**What it does:** Shows a log of every position HEAD has pointed to — every commit, checkout, merge, reset, rebase, amend, etc. This is your safety net.

**Recovering from a bad `reset --hard`:**
```bash
git reflog
# Find the SHA of the commit you want to recover
# e.g., HEAD@{3}: commit: important feature
git reset --hard HEAD@{3}
```

**Retention:** Reflog entries expire after 90 days (for reachable commits) or 30 days (for unreachable ones) by default.

### `git stash push -m "wip"`

**What it does:** Saves all uncommitted changes (staged and unstaged) to a temporary stack, and reverts your working directory to a clean state. The `-m "wip"` adds a description.

**When to use:**
- You need to switch branches but have half-done work
- A colleague asks you to review their code urgently
- You want to test something with a clean working tree

### `git stash list`

**What it does:** Lists all saved stashes:
```
stash@{0}: On main: wip
stash@{1}: On feature: debug session
```

### `git stash pop`

**What it does:** Restores the latest stash **and removes it** from the stash list. If there's a conflict, the stash is kept in the list.

### `git stash apply`

**What it does:** Restores the latest stash but **keeps it** in the stash list. Use this when you want to apply the same stash to multiple branches.

### Interview Q&A — Undoing Changes

> **Q: What's the difference between `git reset` and `git revert`?**
> **A:** `git reset` moves the branch pointer backwards, effectively erasing commits from the branch's history. It rewrites history. `git revert` creates a NEW commit that undoes a previous commit's changes. It preserves history. Use `revert` on shared branches (safe), `reset` on local/private branches.

> **Q: How do you recover a commit after `git reset --hard`?**
> **A:** Use `git reflog` to find the lost commit's SHA, then `git reset --hard <sha>` or `git checkout <sha>` to recover it. Reflog entries last 30–90 days.

> **Q: Explain the three modes of `git reset`.**
> **A:**
> - `--soft`: Moves HEAD back, keeps changes staged (ready to re-commit differently)
> - `--mixed` (default): Moves HEAD back, unstages changes (changes stay in working directory)
> - `--hard`: Moves HEAD back, discards ALL changes (staged + working directory)

> **Q: What's the difference between `git stash pop` and `git stash apply`?**
> **A:** Both restore the stashed changes. `pop` removes the stash from the list after applying. `apply` keeps it. Use `apply` when you want to apply the same changes to multiple branches.

> **Q: How would you undo the last commit but keep changes?**
> **A:** `git reset --soft HEAD~1` — this moves the branch pointer one commit back but keeps all the changes staged, ready to be re-committed differently.

---

## Tags and Releases

### `git tag <tag_name>`

**What it does:** Creates a **lightweight tag** — a simple pointer to a commit, similar to a branch that never moves. No extra metadata is stored.

### `git tag -a <tag_name> -m "Release note"`

**What it does:** Creates an **annotated tag** — a full Git object with the tagger's name, email, date, and a message. Annotated tags are recommended for releases because they're signed and carry metadata.

| Feature | Lightweight | Annotated |
|---|---|---|
| Stored as | Ref (pointer) | Full Git object |
| Has message | No | Yes |
| Has author | No | Yes |
| Can be signed (GPG) | No | Yes |
| Use case | Temporary/private bookmarks | Releases, version marking |

### `git show <tag_name>`

**What it does:** Displays the tag information and the commit it points to. For annotated tags, it shows the tag message, tagger, and date.

### `git push origin <tag_name>`

**What it does:** Pushes a specific tag to the remote. **Tags are not pushed by default** — `git push` only pushes branches.

### `git push origin --tags`

**What it does:** Pushes **all** local tags to the remote at once.

### `git tag -d <tag_name>`

**What it does:** Deletes a tag from your local repository.

### `git push origin --delete <tag_name>`

**What it does:** Deletes a tag from the remote repository. You need both commands (local + remote) for a complete deletion.

### Interview Q&A — Tags

> **Q: What's the difference between a tag and a branch?**
> **A:** Both point to commits, but a branch pointer moves forward with each new commit. A tag is a fixed, immutable pointer — it always points to the same commit. Tags are used to mark specific points like releases (v1.0, v2.0).

> **Q: Why are tags not pushed with `git push`?**
> **A:** By design, Git treats tags as local bookmarks. You might have personal or temporary tags. Requiring explicit `git push origin --tags` prevents accidental pollution of the remote.

---

## Collaboration

### `git switch --track origin/<branch_name>`

**What it does:** Creates a local branch that tracks a remote branch. This is how you start working on a teammate's branch:
```bash
git fetch                                # Get latest remote info
git switch --track origin/feature-login  # Creates local 'feature-login' tracking the remote
```

### `git pull --rebase`

**What it does:** Fetches remote changes and **rebases** your local commits on top instead of merging. This creates a clean, linear history.

**Merge vs. Rebase visualization:**

```
Merge result:                    Rebase result:
A → B → C → M (merge commit)    A → B → C → D' → E'
         ↗                      (linear — D and E replayed on top of C)
    D → E
```

### `git rebase <branch_name>`

**What it does:** Takes the commits on your current branch that aren't in `<branch_name>` and replays them one by one on top of `<branch_name>`.

**The Golden Rule of Rebase:** Never rebase commits that have been pushed to a shared/public branch. Rebasing rewrites commit hashes, which causes confusion and conflicts for others.

**When to rebase:**
- Updating a feature branch with the latest `main` before creating a PR
- Cleaning up local commits before pushing

### `git rebase -i <commit_hash>`

**What it does:** Opens an interactive editor listing recent commits. You can:

| Action | What it does |
|---|---|
| `pick` | Keep the commit as-is |
| `reword` | Edit the commit message |
| `squash` | Meld into the previous commit |
| `fixup` | Like squash but discard this commit's message |
| `drop` | Remove the commit entirely |
| `edit` | Pause rebase to amend the commit |
| `reorder` | Change the order of commits (by moving lines) |

**Common use case:** Squashing 5 messy WIP commits into 1 clean commit before a PR:
```bash
git rebase -i HEAD~5
# Change 'pick' to 'squash' (or 's') for commits 2-5
# Save and edit the combined message
```

### `git cherry-pick <commit_hash>`

**What it does:** Applies a single commit from another branch onto your current branch. It creates a new commit with the same changes but a different SHA.

**Use case:** A hotfix was committed on `develop` and you need it on `main` immediately, without merging the entire `develop` branch.

### `git worktree add ../repo-feature <branch_name>`

**What it does:** Creates a separate working directory linked to the same repository but checked out to a different branch. You can work on two branches simultaneously without stashing or committing.

**Use case:** You're in the middle of a feature but need to do a quick hotfix. Instead of stashing, create a worktree for the hotfix branch.

### `git worktree list`

**What it does:** Lists all working trees linked to this repository.

### Interview Q&A — Collaboration

> **Q: What is `git rebase` and when should you use it?**
> **A:** Rebase replays your commits on top of another branch, creating a linear history. Use it to update feature branches with the latest `main`. Never rebase commits that have been pushed to a shared branch — it rewrites history and breaks others' work.

> **Q: What's the difference between merge and rebase?**
> **A:**
> - **Merge** preserves the full history with a merge commit. It's safe and non-destructive. The graph shows branches.
> - **Rebase** creates a linear history by replaying commits. It looks cleaner but rewrites commit hashes. Not safe for shared branches.

> **Q: What is cherry-picking and when would you use it?**
> **A:** Cherry-pick applies a single specific commit from one branch to another. Use it for applying critical hotfixes to production without merging an entire development branch.

> **Q: What is interactive rebase used for?**
> **A:** Interactive rebase (`-i`) lets you rewrite recent history: squash multiple commits into one, reword messages, reorder commits, or drop commits. Teams use it to clean up messy commit histories before creating pull requests.

> **Q: Explain a typical PR workflow.**
> **A:** 1) `git switch -c feature-x` — create branch. 2) Make commits. 3) `git push -u origin feature-x`. 4) Open PR on GitHub. 5) Team reviews, requests changes. 6) Make fixes, push again. 7) Rebase on `main` if needed. 8) Squash-merge or merge into `main`. 9) Delete the feature branch.

---

## Cleanup

### `git clean -n`

**What it does:** Performs a **dry run** — lists which untracked files *would* be removed without actually deleting anything. Always run this first.

### `git clean -f`

**What it does:** Deletes untracked files. Git requires the `-f` (force) flag as a safety measure to prevent accidental deletion.

**Note:** By default, `git clean` ignores files listed in `.gitignore`. To remove those too, add `-x`.

### `git clean -fd`

**What it does:** Deletes untracked files **and** untracked directories.

### `git remote prune origin`

**What it does:** Removes local remote-tracking references (like `origin/old-feature`) for branches that have been deleted on the remote. Same as `git fetch --prune` but without downloading new data.

### Interview Q&A — Cleanup

> **Q: How do you remove untracked files safely?**
> **A:** Always start with `git clean -n` (dry run) to see what will be removed. Then use `git clean -f` for files only or `git clean -fd` for files and directories. Add `-x` to also remove gitignored files (like build artifacts).

---

## Quick Reference

These are the commands you'll type dozens of times per day:

| Command | Purpose |
|---|---|
| `git status -sb` | Quick overview of what's changed |
| `git add .` | Stage all changes |
| `git commit -m "Message"` | Commit with a message |
| `git push` | Push to the tracked remote branch |
| `git pull` | Fetch and merge remote changes |
| `git log --oneline --graph --decorate --all` | Visual history |
| `git reflog` | Recovery tool — find lost commits |

---

## Interview Questions — Rapid Fire

This section covers commonly asked Git interview questions that don't fit neatly into one section above.

> **Q: What is `.gitignore`?**
> **A:** A file listing patterns of files/directories Git should ignore (not track). Common entries: `node_modules/`, `*.log`, `.env`, `__pycache__/`, `build/`. It must be committed to the repo to be shared.

> **Q: What is `git bisect`?**
> **A:** A binary search tool for finding which commit introduced a bug. You mark a known good commit and a known bad commit. Git checks out the midpoint and you test. Repeat until the exact culprit commit is found — in O(log n) steps.

> **Q: What is a detached HEAD?**
> **A:** When HEAD points directly to a commit instead of a branch name. This happens when you `git checkout <commit-hash>`. Any commits you make in detached HEAD are "orphaned" when you switch branches (but recoverable via reflog). Fix: create a branch from the detached state with `git switch -c new-branch`.

> **Q: What is `git submodule`?**
> **A:** A way to embed one Git repository inside another. The parent repo stores a pointer to a specific commit in the child repo. Used for shared libraries or dependencies. Alternative: `git subtree` (copies the code instead of linking).

> **Q: How do you squash commits?**
> **A:** Two approaches:
> 1. Interactive rebase: `git rebase -i HEAD~N`, mark commits as `squash`
> 2. Soft reset: `git reset --soft HEAD~N && git commit`
> GitHub PRs also offer a "Squash and merge" button.

> **Q: What is `git gc`?**
> **A:** Garbage collection — compresses and cleans up the `.git` directory. It packs loose objects, removes unreachable objects, and optimizes storage. Git runs it automatically, but you can trigger it manually.

> **Q: What is a merge commit vs. a squash merge vs. a rebase merge?**
> **A:**
> - **Merge commit**: Preserves all individual commits and creates a merge commit. Full history.
> - **Squash merge**: Combines all branch commits into one commit on the target. Clean history, but individual commits are lost.
> - **Rebase merge**: Replays commits linearly on the target. Clean history with individual commits preserved, but SHAs change.

> **Q: How do you revert a merge commit?**
> **A:** `git revert -m 1 <merge-commit-hash>`. The `-m 1` flag tells Git which parent to keep (mainline). This is needed because a merge commit has two parents.

> **Q: What is the difference between `HEAD`, `HEAD~`, and `HEAD^`?**
> **A:**
> - `HEAD` — current commit
> - `HEAD~1` (or `HEAD~`) — first parent, one commit back
> - `HEAD~3` — three commits back (following first parents)
> - `HEAD^1` — first parent (same as `HEAD~`)
> - `HEAD^2` — second parent (used for merge commits to reference the merged branch)

> **Q: What is a Git hook?**
> **A:** Scripts that Git executes automatically before or after events like `commit`, `push`, `merge`. Common hooks:
> - `pre-commit`: Run linting/tests before allowing a commit
> - `commit-msg`: Validate commit message format
> - `pre-push`: Run tests before pushing
> Hooks live in `.git/hooks/`. Tools like Husky manage them for JavaScript projects.

> **Q: How do you handle large files in Git?**
> **A:** Use **Git LFS (Large File Storage)**. It replaces large files (images, videos, binaries) with lightweight pointers in the repo, storing the actual files on a separate server.
> ```bash
> git lfs install
> git lfs track "*.psd"
> ```

> **Q: What is `git stash drop` vs `git stash clear`?**
> **A:** `git stash drop stash@{N}` removes a specific stash. `git stash clear` removes ALL stashes. Neither can be undone.

---

*This cheat sheet covers 50+ Git commands and 30+ interview questions. Master these and you'll handle any Git workflow or interview with confidence.*