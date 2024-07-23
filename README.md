

1. **Create a new repository:**
   - Go to GitHub and log in to your account.
   - Click on the "+" icon in the upper right corner and select "New repository".
   - Enter a repository name, for example, `git-cheat-sheet`.
   - Choose to make it public.
   - Check the box that says "Initialize this repository with a README".
   - Click "Create repository".

2. **Edit the README file:**
   - On the newly created repository page, click on the `README.md` file.
   - Click on the pencil icon (edit button) to edit the file.

3. **Paste the content:**
   - Copy the cheat sheet content below:

```markdown
# Git Cheat Sheet

## Setup
- **Configure user information for all repositories:**
  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "your_email@example.com"
  ```

## Starting a Project
- **Initialize a new Git repository:**
  ```bash
  git init
  ```
- **Clone an existing repository:**
  ```bash
  git clone <repository_url>
  ```
- **Create a new repository on GitHub:**
  1. Go to GitHub and log in to your account.
  2. Click on the "+" icon in the upper right corner and select "New repository".
  3. Enter the repository name and description.
  4. Choose to make it public or private.
  5. Click "Create repository".

## Adding Code to a New Repository
1. **Initialize the repository:**
   ```bash
   git init
   ```
2. **Add the remote repository:**
   ```bash
   git remote add origin <repository_url>
   ```
3. **Add files to the staging area:**
   ```bash
   git add <file>
   git add .  # Add all files
   ```
4. **Commit the files:**
   ```bash
   git commit -m "Initial commit"
   ```
5. **Push the changes to the remote repository:**
   ```bash
   git push -u origin main
   ```

## Basic Snapshotting
- **Check the status of your repository:**
  ```bash
  git status
  ```
- **Add files to the staging area:**
  ```bash
  git add <file>
  git add .          # Add all files
  ```
- **Commit changes:**
  ```bash
  git commit -m "Commit message"
  ```
- **Amend the last commit:**
  ```bash
  git commit --amend
  ```

## Branching and Merging
- **Create a new branch:**
  ```bash
  git branch <branch_name>
  ```
- **Switch to a branch:**
  ```bash
  git checkout <branch_name>
  ```
- **Create and switch to a new branch:**
  ```bash
  git checkout -b <branch_name>
  ```
- **Merge a branch into the current branch:**
  ```bash
  git merge <branch_name>
  ```
- **Delete a branch:**
  ```bash
  git branch -d <branch_name>
  ```
- **Delete a branch (force):**
  ```bash
  git branch -D <branch_name>
  ```

## Remote Repositories
- **Add a remote repository:**
  ```bash
  git remote add origin <repository_url>
  ```
- **Fetch from the remote repository:**
  ```bash
  git fetch
  ```
- **Push changes to the remote repository:**
  ```bash
  git push origin <branch_name>
  ```
- **Pull changes from the remote repository:**
  ```bash
  git pull
  ```
- **Set the remote branch for the current branch:**
  ```bash
  git branch --set-upstream-to=origin/<branch_name>
  ```

## Inspecting and Comparing
- **Show commit history:**
  ```bash
  git log
  ```
- **Show commit history with diffs:**
  ```bash
  git log -p
  ```
- **Show a specific commit:**
  ```bash
  git show <commit_hash>
  ```
- **Compare branches:**
  ```bash
  git diff <branch_name>
  ```
- **Compare staged changes:**
  ```bash
  git diff --staged
  ```

## Undoing Changes
- **Unstage a file:**
  ```bash
  git reset <file>
  ```
- **Revert a commit (create a new commit that undoes changes):**
  ```bash
  git revert <commit_hash>
  ```
- **Reset to a specific commit:**
  ```bash
  git reset --hard <commit_hash>
  ```
- **Stash changes:**
  ```bash
  git stash
  ```
- **Apply stashed changes:**
  ```bash
  git stash apply
  ```
- **List stashes:**
  ```bash
  git stash list
  ```

## Working with Tags
- **Create a tag:**
  ```bash
  git tag <tag_name>
  ```
- **Push tags to remote:**
  ```bash
  git push origin --tags
  ```
- **Delete a local tag:**
  ```bash
  git tag -d <tag_name>
  ```
- **Delete a remote tag:**
  ```bash
  git push origin --delete tag <tag_name>
  ```

## Advanced Commands
- **Rebase a branch:**
  ```bash
  git rebase <branch_name>
  ```
- **Interactive rebase:**
  ```bash
  git rebase -i <commit_hash>
  ```
- **Cherry-pick a commit:**
  ```bash
  git cherry-pick <commit_hash>
  ```

## Collaboration
- **Fork a repository:**
  - Go to the repository page on GitHub/GitLab and click on the "Fork" button.
- **Create a pull request:**
  - Go to your forked repository, switch to the branch you want to merge, and click on the "New pull request" button.

## Cleaning Up
- **Remove untracked files:**
  ```bash
  git clean -f
  ```
- **Remove untracked files and directories:**
  ```bash
  git clean -fd
  ```

## Summary Commands
- **Show the summary of changes:**
  ```bash
  git status
  git diff
  git log
  ```

This cheat sheet should cover most of the commands you'll need to manage your Git repositories effectively.