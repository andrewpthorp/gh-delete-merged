# gh-delete-merged

A [GitHub CLI](https://cli.github.com/) extension that deletes local branches whose pull request was merged.

```sh
gh extension install andrewpthorp/gh-delete-merged
```

## Why

Tools that detect merged branches with git alone (`git branch --merged`, `git-delete-squashed`) compare commits or diffs against the default branch. That misses branches when:

- the PR was **squash merged** or **rebase merged**, so the branch's commits never appear on `main`
- the PR was part of a **stack**, so the branch's diff against `main` includes its parent branches
- the branch was **rebased or amended**, or `main` was merged into it during review
- the PR was edited on GitHub (suggestions, conflict resolution)

`gh delete-merged` asks GitHub instead. For each local branch it looks up merged pull requests with that head branch, and only deletes the branch when its local tip is **exactly** the commit that was merged. If you have commits that were never merged, the branch is skipped.

## Usage

Run it inside a repository:

```sh
gh delete-merged --dry-run  # list what would be deleted
gh delete-merged            # delete those branches
```

```
delete  feature/login-page (#1234)
delete  fix/typo-in-readme (#1240)
skip    feature/search (has commits that were not merged in #1251)

Dry run: 2 branch(es) would be deleted. Run without --dry-run to delete them.
```

### Flags

| Flag | Description |
| --- | --- |
| `-n`, `--dry-run` | List the branches instead of deleting them |
| `-e`, `--exclude <branch>` | Never delete this branch (can be repeated) |
| `-h`, `--help` | Show help |

### What is never deleted

- the repository's default branch
- the branch you have checked out
- branches checked out in other worktrees
- branches passed to `--exclude`
- branches whose local tip differs from the merged PR's head commit

## Things to know

- **It only touches local branches.** Remote branches are left alone.
- **Long-lived branches can match.** If a branch like `develop` is the head of a merged PR (for example `develop` → `main`) and your local `develop` is exactly at that commit, it will be listed. Run with `--dry-run` first, or use `--exclude develop`.
- **"Merged" means merged into any base.** A stacked PR merged into its parent branch counts, because its commits are preserved there.
- **It checks the repository `gh` resolves** for the current directory (see `gh repo set-default`).
- Requires `gh` (authenticated) and `git`. Runs on the bash that ships with macOS (3.2).

## License

MIT
