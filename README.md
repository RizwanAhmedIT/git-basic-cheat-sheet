# Git Cheat Sheet

A practical Git reference for everyday local work, branching, remote syncing, and safe undo workflows.

## Contents
- Setup
- SSH Setup
- Start a Repository
- Daily Workflow
- Branching
- Remotes
- Inspecting History
- Undoing Changes
- Tags and Releases
- Collaboration
- Cleanup
- Quick Reference

## Setup
Configure your identity once per machine:

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
git config --global init.defaultBranch main
```

Useful quality-of-life settings:

```bash
git config --global pull.rebase false
git config --global fetch.prune true
git config --global core.autocrlf true
```

## SSH Setup
Create an SSH key and add it to your Git hosting account.

### Linux
Create an SSH key:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Start the SSH agent and add your key:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Copy your public key and add it to GitHub, GitLab, or Bitbucket:

```bash
cat ~/.ssh/id_ed25519.pub
```

Test the connection:

```bash
ssh -T git@github.com
```

### Windows
Open PowerShell and create an SSH key:

```powershell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Start the SSH agent and add your key:

```powershell
Start-Service ssh-agent
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

Copy your public key and add it to GitHub, GitLab, or Bitbucket:

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
```

Test the connection:

```powershell
ssh -T git@github.com
```

Use the SSH remote format when cloning or updating a remote on either system:

```bash
git clone git@github.com:owner/repository.git
git remote set-url origin git@github.com:owner/repository.git
```

## Start a Repository
Create a new repo or clone an existing one:

```bash
git init
git clone <repository_url>
```

Add the first remote and push:

```bash
git remote add origin <repository_url>
git add .
git commit -m "Initial commit"
git push -u origin main
```

## Daily Workflow
Check what changed:

```bash
git status
git status -sb
```

Stage and commit:

```bash
git add <file>
git add .
git commit -m "Describe the change"
```

Amend the latest commit when you only need a small correction:

```bash
git commit --amend
```

## Branching
Create and switch branches with the modern commands:

```bash
git branch <branch_name>
git switch <branch_name>
git switch -c <branch_name>
```

Merge a branch into the current branch:

```bash
git merge <branch_name>
```

Delete a branch after it is merged:

```bash
git branch -d <branch_name>
git branch -D <branch_name>
```

See local branches and their upstreams:

```bash
git branch -vv
```

## Remotes
List remotes and inspect remote URLs:

```bash
git remote -v
```

Fetch remote updates without merging:

```bash
git fetch
git fetch --prune
```

Push and pull:

```bash
git push origin <branch_name>
git pull
git push -u origin <branch_name>
```

Set or change the upstream branch:

```bash
git branch --set-upstream-to=origin/<branch_name>
```

## Inspecting History
View recent history in a compact graph:

```bash
git log --oneline --graph --decorate --all
```

Inspect a commit or a file-level change:

```bash
git show <commit_hash>
git log -p
```

Compare changes:

```bash
git diff
git diff --staged
git diff <branch_name>
git diff <commit_a>..<commit_b>
```

Find who changed a line:

```bash
git blame <file>
```

## Undoing Changes
Unstage files safely:

```bash
git restore --staged <file>
```

Discard working tree changes in a file:

```bash
git restore <file>
```

Revert a commit by creating a new commit that undoes it:

```bash
git revert <commit_hash>
```

Reset is powerful and rewrites history, so use it carefully:

```bash
git reset --soft <commit_hash>
git reset --mixed <commit_hash>
git reset --hard <commit_hash>
```

Recover lost work with reflog:

```bash
git reflog
```

Stash work in progress:

```bash
git stash push -m "wip"
git stash list
git stash pop
git stash apply
```

## Tags and Releases
Create and inspect tags:

```bash
git tag <tag_name>
git tag -a <tag_name> -m "Release note"
git show <tag_name>
```

Push or delete tags:

```bash
git push origin <tag_name>
git push origin --tags
git tag -d <tag_name>
git push origin --delete <tag_name>
```

## Collaboration
Track a remote branch locally:

```bash
git switch --track origin/<branch_name>
```

Use rebase when you want a linear local history:

```bash
git pull --rebase
git rebase <branch_name>
git rebase -i <commit_hash>
```

Move a single commit onto your current branch:

```bash
git cherry-pick <commit_hash>
```

If you work on multiple branches at once, use worktrees:

```bash
git worktree add ../repo-feature <branch_name>
git worktree list
```

## Cleanup
Remove untracked files carefully:

```bash
git clean -n
git clean -f
git clean -fd
```

Remove stale remote tracking branches:

```bash
git remote prune origin
```

## Quick Reference
Common one-liners:

```bash
git status -sb
git add .
git commit -m "Message"
git push
git pull
git log --oneline --graph --decorate --all
git reflog
```

This cheat sheet now covers the commands most people use day to day, plus safer recovery options when something goes wrong.