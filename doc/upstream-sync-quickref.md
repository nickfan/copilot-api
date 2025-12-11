# Upstream Sync Quick Reference

## Commands Overview

```bash
# Step 1: Update all branch (origin/all ← upstream/all)
git checkout all
git stash
git fetch upstream
git merge upstream/all
git push origin all
git stash pop

# Step 2: Update myall branch (origin/myall ← origin/all)
git checkout myall
git stash
git fetch origin all
git rebase origin/all
git push origin myall --force-with-lease
git stash pop
```

## What Each Command Does

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `git stash` | Save uncommitted changes | Before any branch operations |
| `git fetch upstream` | Download upstream updates | To check what's new from source |
| `git merge upstream/all` | Merge upstream into all | For all branch (mirror) |
| `git rebase origin/all` | Rebase myall onto latest | For myall branch (custom) |
| `git push` | Upload to origin | After successful merge/rebase |
| `git stash pop` | Restore saved changes | After all operations complete |

## Branch Strategy

### all branch
- **Type**: Mirror branch
- **Strategy**: `merge`
- **Goal**: Match upstream exactly

### myall branch
- **Type**: Custom branch
- **Strategy**: `rebase`
- **Goal**: Keep custom commits on top of upstream

## Status Check Commands

```bash
# Check remotes
git remote -v

# Check branch status
git branch -vv

# View last 5 commits
git log --oneline -5

# Check uncommitted changes
git status

# Verify sync across all branches
echo "upstream/all:" && git log --oneline upstream/all -3
echo "origin/all:" && git log --oneline origin/all -3
echo "local all:" && git log --oneline all -3
echo "origin/myall:" && git log --oneline origin/myall -3
echo "local myall:" && git log --oneline myall -3
```

## Troubleshooting

### Merge conflict
```bash
git status                    # See conflicted files
# Edit files to resolve conflicts
git add <file>                # Mark as resolved
git merge --continue         # Continue merge
```

### Rebase conflict
```bash
git status                    # See conflicted files
# Edit files to resolve conflicts
git add <file>                # Mark as resolved
git rebase --continue        # Continue rebase
```

### Push rejected
```bash
git fetch origin              # Get latest remote state
git rebase origin/all        # Rebase onto latest (for myall)
git pull --rebase origin all  # Pull with rebase (for all)
git push origin <branch>     # Try push again
```

### Stash issues
```bash
git stash list               # View stashed changes
git stash pop                # Apply and drop latest stash
git stash apply              # Apply without dropping
git stash drop               # Drop latest stash
```

## When to Run This Process

- [ ] Weekly sync with upstream (recommended)
- [ ] Before starting new features
- [ ] Before submitting PRs
- [ ] When upstream has important bug fixes
- [ ] When CI/CD indicates upstream changes needed

## Safety Checklist

Before running:
- [ ] All work is stashed or committed
- [ ] Currently on correct branch
- [ ] No active merge/rebase operations

After running:
- [ ] Verify all branches up to date
- [ ] Verify stash restored successfully
- [ ] Verify no unexpected files modified
- [ ] Run tests if available

## Emergency Recovery

If something goes wrong:

```bash
# Abort current operation
git merge --abort            # or git rebase --abort

# Restore from backup if exists
git reset --hard myall-backup-20251211

# Reset to remote state
git fetch origin
git reset --hard origin/myall
```

## Remote URLs (for reference)

```
origin:  git@github.com:nickfan/copilot-api.git
upstream: git@github.com:caozhiyuan/copilot-api.git
```

---

**Last Updated**: 2025-12-11
**Project**: copilot-api
**Full Guide**: See [upstream-sync.md](./upstream-sync.md)
