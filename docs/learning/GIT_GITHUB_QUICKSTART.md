# Git and GitHub Quickstart

To learn what Git is and how to manage your projects via the command line on GitHub, see this [tutorial](https://www.w3schools.com/git/git_intro.asp?remote=github).

Use this flow for every milestone.

## One-Time Setup

Set your Git identity (your identity is attached to all your commits):

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### [Fork](https://www.w3schools.com/git/git_remote_fork.asp?remote=github) this repo under your own account
### Pick a directory on your local computer where you want the code for this project to live. Then [clone](https://www.w3schools.com/git/git_clone.asp?remote=github) your forked repo into that directory.

## Daily Workflow

### 1. Start from the `main` branch. This is the branch with working code that all other branches should branch from.

```bash
git checkout main     # Switch to `main` branch
git pull origin main  # Update your local `main` branch with changes from remote `main`
```

### 2. Create a branch for your milestone

Branch naming pattern:
- `m1-foundation`
- `m2-player-input`
- `m3-balancing`

This creates a local branch but DOES NOT create a corresponding remote branch on GitHub. That is handled on a later step.

```bash
git checkout -b m1-foundation  
```

### 3. Make changes and check status to see which files changed

```bash
git status
```

### 4. Stage and commit
Note that the best way to view changed files before committing and pushing is in a code editor like [Visual Studio Code](https://code.visualstudio.com/docs/sourcecontrol/overview#_source-control-interface).

```bash
git add .  # '.' adds all files in this directory or lower. Alternatively specify each file to add to the commit
git commit -m "Milestone 1: league setup and team rendering"
```

Commit message pattern:
- `Milestone X: short summary`

### 5. Push your branch

The first time you push a branch you've made locally, specify it should be tracked by GitHub (the `-u origin <branch>` part).
```bash
git push -u origin m1-foundation
```

For every other time you push this branch:
```bash
git push
```

### 6. Open a Pull Request on GitHub (PR)

- Go to the repo on GitHub.
- Click "Compare & pull request" for your branch.
- Fill out the PR template.
- Set base branch to `main`.
- Submit PR.

### 7. Merge PR and clean up

After the PR is approved and ready:
- Merge on GitHub.
- Then locally:

```bash
git checkout main
git pull origin main
```

Optionally delete local branches that you will no longer use:
```bash
git branch -d m1-foundation
```

## Common Mistakes and Fixes

Committed to wrong branch:
- Do not panic. Use an AI for assistance with moving your changes to the desired branch, or carefully consult Stack Overflow before trying history-rewrite commands.

Forgot to include a file:

```bash
git add <file>
git commit --amend --no-edit
git push --force-with-lease
```

PR includes unrelated files:
- Remove them from staging and recommit:

```bash
git restore --staged <file>
```

## Safe Rules

- Commit small and often.
- One milestone = one branch.
- Never use `git reset --hard` unless you fully understand consequences.
- Read `git status` before every commit and push.
