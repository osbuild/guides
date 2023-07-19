# Workflow

## Git Workflow

### Commits

Commits should be easy to read.

The commit message should explain clearly what it's trying to do and why. The following format is
common but not required:

```
<module>: Topic of the commit

Body of the commit, describing the changes in more detail.
```

The `<module>` should point to the area of the codebase (for instance `tests` or `tools`). The topic
should summarize what the commit is doing.

GitHub truncates the first line if it's longer than 65 characters, which is something to keep in
mind as well.

A `Fixes #issue-number` can be added to automatically link and close a related issue if it exists.

### Pull requests

A pull request should be one or more commits which form a coherent unit, it can be
rebased/rewritten/force-pushed until it's fit for merging.

How the PR developed, and the iterations it went through, should not be visible in the git
history. The end result counts: a certain amount of commits, each one forming a logical unit of
changes. Avoid 'fix-up' commits which tweak previous commits in the PR.

Pull requests should be opened from a developer's own fork to avoid random branches on the origin.

Each pull request should be reviewed, and the CI should pass.

Once a pull request is ready to be merged, it should be merged via the `Rebase and merge` or `Squash
and merge` option. This avoids merge commits on the main branch.

### Branches

Force-pushing to, or rebasing the main branch (or other release branches) is not allowed. Avoid
directly pushing (fast-forward) to those branches as well. Commits can always be reverted by opening
a new pull request.
