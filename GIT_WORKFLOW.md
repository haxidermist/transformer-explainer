# Git Workflow Guide for Transformer Explainer Fork

This guide covers common git operations for maintaining your fork of the Transformer Explainer project.

## Repository Setup

Your fork is configured with two remotes:

```bash
# View your remotes
git remote -v

# Should show:
# origin   → https://github.com/haxidermist/transformer-explainer.git (your fork)
# upstream → https://github.com/poloclub/transformer-explainer.git (original repo)
```

## Making Changes

### 1. Check Current Status

```bash
# See what files have changed
git status

# See the actual changes
git diff
```

### 2. Stage and Commit Changes

```bash
# Stage specific files
git add path/to/file1 path/to/file2

# Or stage all changes
git add .

# Commit with a descriptive message
git commit -m "Brief description of changes"

# Or use a multi-line commit message
git commit -m "$(cat <<'EOF'
Brief summary of changes

- Detailed point 1
- Detailed point 2
- Detailed point 3

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### 3. Push to Your Fork

```bash
# Push to your fork (origin)
git push origin main

# If you've made local commits and want to force push (use carefully!)
git push origin main --force
```

## Syncing with Upstream

### Update Your Fork with Latest Changes from Original Repo

```bash
# Fetch latest changes from upstream
git fetch upstream

# Make sure you're on main branch
git checkout main

# Merge upstream changes into your main
git merge upstream/main

# If there are conflicts, resolve them, then:
git add .
git commit -m "Merge upstream changes"

# Push updated main to your fork
git push origin main
```

### Alternative: Rebase Instead of Merge

```bash
# Fetch upstream
git fetch upstream

# Rebase your changes on top of upstream
git rebase upstream/main

# If conflicts occur, resolve them, then:
git add .
git rebase --continue

# Force push (rebase rewrites history)
git push origin main --force
```

## Creating a Pull Request

If you want to contribute your changes back to the original repository:

```bash
# Make sure your changes are committed and pushed
git push origin main

# Create a pull request using GitHub CLI
gh pr create --base main --head haxidermist:main --title "Your PR title" --body "Description of your changes"

# Or create via web interface:
# Visit: https://github.com/poloclub/transformer-explainer
# Click "Compare & pull request"
```

## Deploying to GitHub Pages

### Build and Deploy

```bash
# Build the production version
npm run build

# Deploy to GitHub Pages (publishes to gh-pages branch)
npm run deploy

# Your site will be available at:
# https://haxidermist.github.io/transformer-explainer/
```

### Enable GitHub Pages (First Time Only)

1. Go to: https://github.com/haxidermist/transformer-explainer/settings/pages
2. Under "Source", select branch: `gh-pages`
3. Click "Save"
4. Site will be published to: https://haxidermist.github.io/transformer-explainer/

## Branch Management

### Create a Feature Branch

```bash
# Create and switch to new branch
git checkout -b feature/my-new-feature

# Make changes, then commit
git add .
git commit -m "Add new feature"

# Push feature branch to your fork
git push origin feature/my-new-feature

# Create PR from feature branch
gh pr create --head haxidermist:feature/my-new-feature
```

### Switch Between Branches

```bash
# List all branches
git branch -a

# Switch to a branch
git checkout main
git checkout feature/my-feature

# Delete a local branch
git branch -d feature/old-feature

# Delete a remote branch
git push origin --delete feature/old-feature
```

## Undoing Changes

### Discard Uncommitted Changes

```bash
# Discard changes to a specific file
git restore path/to/file

# Discard all uncommitted changes
git restore .

# Discard staged changes
git restore --staged path/to/file
```

### Undo Last Commit (Keep Changes)

```bash
# Undo last commit but keep changes in working directory
git reset --soft HEAD~1

# Undo last commit and discard changes
git reset --hard HEAD~1
```

### Revert a Commit

```bash
# Create a new commit that undoes a previous commit
git revert <commit-hash>

# Find commit hash
git log --oneline
```

## Viewing History

```bash
# View commit history
git log

# Compact view
git log --oneline

# View history with graph
git log --graph --oneline --all

# View changes in a specific commit
git show <commit-hash>

# View file history
git log -p path/to/file
```

## Useful Git Aliases

Add these to `~/.gitconfig`:

```ini
[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    unstage = restore --staged
    last = log -1 HEAD
    visual = log --graph --oneline --all --decorate
```

Usage:
```bash
git st        # instead of git status
git co main   # instead of git checkout main
```

## Common Workflows

### Daily Development

```bash
# 1. Start of day: sync with upstream
git checkout main
git fetch upstream
git merge upstream/main
git push origin main

# 2. Create feature branch
git checkout -b feature/my-feature

# 3. Make changes and commit
git add .
git commit -m "Implement feature"

# 4. Push to your fork
git push origin feature/my-feature

# 5. Create PR (if contributing back)
gh pr create
```

### Quick Fix

```bash
# 1. Make changes directly on main
git checkout main
git add .
git commit -m "Quick fix for bug"

# 2. Push to fork
git push origin main
```

### Update Dependencies

```bash
# Update npm packages
npm update

# Commit package-lock.json
git add package-lock.json
git commit -m "Update dependencies"
git push origin main
```

## Troubleshooting

### Merge Conflicts

```bash
# When merge conflicts occur:
# 1. Open conflicting files
# 2. Look for conflict markers:
#    <<<<<<< HEAD
#    your changes
#    =======
#    upstream changes
#    >>>>>>> upstream/main

# 3. Resolve conflicts manually
# 4. Stage resolved files
git add path/to/resolved/file

# 5. Complete the merge
git commit -m "Resolve merge conflicts"
```

### Accidentally Committed to Wrong Branch

```bash
# Move last commit to a new branch
git branch feature/new-branch
git reset --hard HEAD~1
git checkout feature/new-branch
```

### Lost Changes

```bash
# Find lost commits
git reflog

# Recover lost commit
git checkout <commit-hash>
git checkout -b recovery-branch
```

## GitHub CLI Shortcuts

```bash
# View your forks
gh repo list

# View pull requests
gh pr list

# Check out a PR locally
gh pr checkout <pr-number>

# View repo in browser
gh repo view --web

# Create issue
gh issue create
```

## Important Notes

- **Never force push to `upstream/main`** - you don't have write access
- **Always sync with upstream** before starting new work
- **Use feature branches** for significant changes
- **Test locally** before pushing (run `npm run dev`)
- **Build before deploying** to catch errors (`npm run build`)

## Quick Reference

| Task | Command |
|------|---------|
| Check status | `git status` |
| Stage all changes | `git add .` |
| Commit | `git commit -m "message"` |
| Push to fork | `git push origin main` |
| Sync from upstream | `git fetch upstream && git merge upstream/main` |
| Deploy to Pages | `npm run deploy` |
| Create PR | `gh pr create` |
| View history | `git log --oneline` |

## Resources

- [Git Documentation](https://git-scm.com/doc)
- [GitHub CLI Manual](https://cli.github.com/manual/)
- [Forking Workflow Guide](https://docs.github.com/en/get-started/quickstart/fork-a-repo)
- Original Repo: https://github.com/poloclub/transformer-explainer
- Your Fork: https://github.com/haxidermist/transformer-explainer
