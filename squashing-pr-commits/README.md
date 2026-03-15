# Squashing PR Commits with Soft Reset

When a PR has many noisy commits, you can squash them all into a single clean commit
before merging. The trick is to find the exact commit where your feature branch diverged
from `main` — programmatically — so you don't have to count commits manually.

## How it works

### 1. Soft-reset back to the divergence point

```bash
git reset --soft $(git merge-base HEAD main)
```

`git merge-base HEAD main` returns the best common ancestor between your branch and
`main` — the commit where they diverged. Passing it directly to `git reset --soft`
moves `HEAD` back to that point while leaving all your changes staged.

### 2. Create the single squashed commit

```bash
git commit -m "feat: your descriptive single commit message"
```

### 3. Force-push to update your PR branch

```bash
git push --force-with-lease
```

`--force-with-lease` is safer than `--force`: it refuses to overwrite the remote branch
if someone else pushed to it since you last fetched.

## Why `git merge-base` instead of counting commits?

Counting commits (`git reset --soft HEAD~N`) is fragile — you have to know `N` and it
breaks if the branch history changes. `git merge-base HEAD main` always resolves to the
correct divergence point regardless of how many commits are on the branch.

## Example

```
main:    A - B - C
                  \
feature:           D - E - F - G   (4 messy commits)
```

```bash
# On your feature branch
git reset --soft $(git merge-base HEAD main)  # HEAD moves to C, D+E+F+G staged
git commit -m "feat: implement new widget"
git push --force-with-lease
```

```
main:    A - B - C
                  \
feature:           H   (one clean commit containing all changes from D-G)
```

## Notes

- This rewrites history, so only do it on branches that **you own** and haven't been
  reviewed commit-by-commit by others.
- If your branch is behind `main`, rebase first (`git rebase main`) so the divergence
  point is up to date before squashing.
