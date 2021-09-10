# Workflow

## Adding news

When adding a new feature to a project which should be included in the release notes, or used for
announcements, please add it under `docs/news` in the form of a short markdown document.

Bug fixes, test changes, etc... should not be announced there.

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

A pull request should be one or more commits which are fit for rebase-and-merging on main, it can be
rebased/rewritten/force-pushed until it's fit for merging. The development process and history of
the PR should not be visible in the commits, the end result counts.

Pull requests are usually opened against the main branch. They should be opened from a developer's
own fork to avoid a lot of random branches on the origin.

Each pull request should be reviewed, and the CI should pass.

A pull request can be marked as draft, if it shouldn't be reviewed yet. But once it's ready, do not
hesitate to add reviewers. If you're unsure who to add as a reviewer, ask in the irc channel
(#osbuild on Libera Chat).

Once a pull request is ready to be merged, it should be merged via the `Rebase and merge` or `Squash
and merge` option. This avoids merge commits on the main branch.

### Branches

Force-pushing to, or rebasing the main branch (or other release branches) is not allowed. Avoid
directly pushing (fast-forward) to those branches as well. Commits can always be reverted by opening
a new pull request.
