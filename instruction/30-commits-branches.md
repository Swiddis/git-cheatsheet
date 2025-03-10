# Commits and Branches: Your Map Through Time

The majority of your time with Git is spent working on *branches*. A branch is a label that points
to a specific commit. The primary difference between branches and *tags* is that branches are
expected to change their target commit over time. The difference between *tags* and *commits* is
that tags have human-provided semantic meaning.

## The Map: Understanding The Commit Graph

As an instructional example, here's a simplified graph of the recent commits (chronlogically
ordered) for a project that the author works on. You can generate a similar graph for any repository
by memorizing the incantation: `git log --oneline --graph --all`[^1]. For the most up-to-date
results, it's good to preface this with `git fetch --all`.

```
* 6ea90a6e7 (HEAD -> main, upstream/main) Support custom logs correlation (#2375)
| * 3441efcc7 (pr/db-sel) Database selector in Integration install
|/
* a7e49da04 release notes 3.0.0-aplha1 (#2379)
* cd70e059a Improve the test results for Integrations internals (#2376)
|\
| * bbad1de47 (origin/feature/better-integ-failures, feature/better-integ-failures) Assert serialized integrations works as part of usage
| * 6f88fd871 Add context to integrationReader errors
| * 71bc489a4 Add custom expect handlers for integration results
|/
* f80845e83 Clear ADMINS.md. (#2363)
* 8adb81e5f fix traces redirection while QA enabled (#2369)
```

One important thing to spot in this output is `HEAD`. This is where you currently are. In the
default color scheme (which isn't visible in this document), it's highlighted blue.

In a collaborative project with many contributors, it's typical that the `main` branch will continue
progressing as others work on it, while your features lag behind. You can see two examples of this
in the graph:
- `feature/better-integ-failures` currently points to `bbad1de47`. This was a feature that started
  getting from `380845e83`. It happens that there were no updates on `main` from then until the
  feature was merged in `cd70e059a`.
- `pr/db-sel` is a branch where I checked out a coworker's Pull Request for local review. This one
  had a change happen under it while it was in progress (`6ea90a6e7`).

Branches usually have corresponding *remotes*. `main` has a corresponding `upstream/main`.
`feature/better-integ-features` has a corresponding `origin/feature/better-integ-features`. When you
see one of these prefixes, that means that it's the version of the branch that lives on a certain
remote Git server, like those managed by GitHub. They can get out of sync in two ways:
- If you have a local change and haven't `push`ed it, your local branch will be ahead of the remote
  branch.
- If the remote branch was updated and you `fetch`ed it, but haven't `pull`ed or `merge`d it, the
  remote branch will be ahead of your local branch.

You can see exactly where the remotes live with `git remote -v`. You can have any number of remotes.

```
> git remote -v
origin  https://github.com/Swiddis/dashboards-observability.git (fetch)
origin  https://github.com/Swiddis/dashboards-observability.git (push)
upstream        https://github.com/opensearch-project/dashboards-observability.git (fetch)
upstream        https://github.com/opensearch-project/dashboards-observability.git (push)
my-favorite-coworker  https://github.com/octocat/dashboards-observability.git (fetch)
my-favorite-coworker  https://github.com/octocat/dashboards-observability.git (push)
```

## Traveling Through Time

As mentioned in the previous section: `HEAD` is where you currently are. The primary way to change
that is to switch to a branch: `git switch feature/better-integ-failures`[^2]. (It's better with tab
completion.) If we do this and run our incantation, the graph now looks like this:

```
* 6ea90a6e7 (main, upstream/main) Support custom logs correlation (#2375)
| * 3441efcc7 (pr/db-sel) Database selector in Integration install
|/
* a7e49da04 release notes 3.0.0-aplha1 (#2379)
* cd70e059a Improve the test results for Integrations internals (#2376)
|\
| * bbad1de47 (HEAD -> feature/better-integ-failures, origin/feature/better-integ-failures) Assert serialized integrations works as part of usage
| * 6f88fd871 Add context to integrationReader errors
| * 71bc489a4 Add custom expect handlers for integration results
|/
* f80845e83 Clear ADMINS.md. (#2363)
* 8adb81e5f fix traces redirection while QA enabled (#2369)
```

Switching is the primary way to get around when navigating a large repository. It lets you develop
multiple features in parallel (e.g. by creating branches for each feature and switching while
waiting on code reviews). It lets you test your coworkers' changes locally without losing your
current work.

You can also navigate to a specific commit by switching to it: `git switch --detach 71bc489a4`.
`--detach` means to disconnect from any branch labels, you genuinely want a specific point. That
means you won't be able to automatically receive updates until you switch back to a branch. This
also means that if you make changes, you won't be able to easily find them again unless you later
create a branch for them. (The primary way to recover detached commits is `git reflog`.)

## Making History

We now know how to view and navigate the graph, The other category of git commands that you use on a
daily basis will manipulate this graph. A brief rundown of the big ones:
- When you `git commit`, a new node will be added that points back to your current `HEAD`. Your
  `HEAD` will update to point to the new commit. If your `HEAD` is a branch, the local branch will
  also update to point to the new commit.
- When you `git merge`, you create a new commit that points to both the *base* you started from, and
  the *head* that you're merging in. The same `HEAD` switching semantics apply. `git rebase` will
  try to modify the history of your branch to build on top of the head branch, instead of creating a
  new merge commit.
- When you `git push`, you update the remote branch for your local branch to point to where your
  local branch currently points. This update only travels forward in time unless you add `--force`.
  If you're in a collaborative environment, use `--force-with-lease` which guards against the
  pitfalls of `--force`.
- When you `git pull`, the remote branch for your branch updates to the most up-to-date version, and
  your branch automatically updates to that same version. `get fetch` is the twin that won't update
  your local branch, it will only update the remote branch. By default, both commands only operate
  with the branch you're currently on. To update *all* branches, use `git (fetch|pull) --all`.

## Branching Strategy

There are a lot of opinions on this. The author is going to share what works for them. This strategy
is generally only necessary in an environment with lots of in-flight features and context-switching.

Firstly, all branches except copies of remote branches should have a prefix. This is the `feature/`
and `pr/` from my earlier examples. Some prefixes I use:
- `feature/` marks features I'm working on: these are usually more complex, larger changes.
- `*fix/` marks smaller, usually one-off changes. These are frequent, split by type (e.g. `bugfix`, `hotfix`),
  and meant to be cleaned up regularly: `git branch --list | grep 'hotfix/*' | xargs git branch -d`
- `pr/` marks when I check out coworkers' changes for local testing. They can be quickly cleaned.

For remote branches, when working on forks, I explicitly set them to track upstream: `git branch --set-upstream-to=upstream/[branch]`.
This means `git pull` works as expected. If you want, you can update your forks's copy with `git push origin`,
but this usually isn't necessary[^3].

I rarely develop changes outside of a prefixed branch because of nonzero review time. If `main`
becomes dirty locally, it gets annoying to start new work from a clean slate in parallel[^4][^5].

[^1]: You can also [create a nice alias](https://peternied.github.io/git/2023/05/29/git-log.html),
    but the author still recommends memorizing the command since it persists over SSH.
[^2]: `git checkout` is an older option. `switch` is newer, and nice in that it protects you from
    several footguns with checkout. If you're at a level where this document is useful to you, I
    recommend using `switch`.
[^3]: At the time of writing, the author's `origin/main` is 79 commits out of date and hasn't been
    updated in 6 months.
[^4]: For how much of a fuss I make about it, it's really not that hard to do, just mildly
    inconvenient. You can recover from most situations by running the `git log --oneline --graph --all`
    incantation and finding the commits you need.
[^5]: In a perfect world, you could operate without `main` entirely and do everything from
    `upstream/main`, but sometimes upstream is broken and you'd rather not deal with it.
