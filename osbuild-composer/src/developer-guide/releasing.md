# Releasing

This guide describes the process of releasing osbuild and osbuild-composer to [upstream][upstream-git], into [Fedora][fedora-distgit] and [CentOS Stream][centos-distgit].

## Clone the release helpers

Go to the [maintainer-tools repository][maintainer-tools], clone the repository and run `pip install -r requirements.txt` in order to get all the dependencies to be able to execute the `release.py` and `update-distgit.py` scripts.

## Upstream release

Navigate to your local repository in your terminal and call the `release.py` script. It will interactively take you through the following steps:

1. Gather all pull request titles merged to `main` since the latest release tag
2. Create a draft of the next release tag

    While writing the commit message, keep in mind that it needs to conform to both Markdown and git commit message formats, have a look at the commit message for one of the [recent releases][recent-releases] to get a clear idea how it should look like.
3. Push your signed git tag to `main`

From here on a [GitHub composite action][github-action] will take over and

1. Create a GitHub release based on the tag (version and message)
2. Bump the version in `osbuild.spec` or `osbuild-composer.spec` (and potentially `setup.py`)
3. Commit and push this change to `main` so the version is already reflecting the next release

## Fedora release

We use packit (see `.packit.yml` in the osbuild or osbuild-composer repository respectively or the [official packit documentation][packit-dev]) to automatically push new releases directly to [Fedora's dist-git][fedora-distgit].

Then our fedora-bot takes over and performs the remaining steps:

1. Get a kerberos ticket by running `kinit $USER@FEDORAPROJECT.ORG`
2. Call `fedpkg build` to schedule Koji builds for each active Fedora release (or: dist-git branch)
3. Update [Bodhi][bodhi] with the latest release

## CentOS Stream 9 release

Make sure you have all the required tools installed (`centpkg`,`rhelpkg`).

1. Run `centpkg clone $project` to get the latest `c9s` branch (or a simple `git pull` if you already have a local copy)
2. Run `update-distgit.py` and set at least the upstream release version `--version $version` and the packaging tool `--pkgtool centpkg`
3. If you don't have a fork already on gitlab.com yet, run `centpkg fork`
4. Push the release commit to your fork on gitlab.com and create a merge request
5. Cherry-pick the same commit and push it to your fork on OSCI's pagure
6. Create a pull request on OSCI's pagure
7. Wait for the tests to finish, check both your pull request and the OSCI Dashboard for gating test results
8. If all gating tests are green (or: okay/waivable) proceed with closing your PR on src.osci
9. Merge your PR on gitlab.com

The automation will take care of the rest, i.e. syncing to RHEL9's dist-git and updating errata.

## Spreading the word on osbuild.org

The last of releasing a new version is to create a new post on osbuild.org. Just open a PR in [osbuild/osbuild.github.io]. You can find a lot of inspiration in existing release posts.

[upstream-git]: https://github.com/osbuild/osbuild
[fedora-distgit]: https://src.fedoraproject.org/rpms/osbuild
[centos-distgit]: https://gitlab.com/redhat/centos-stream/rpms/osbuild
[maintainer-tools]: https://github.com/osbuild/maintainer-tools
[github-action]: https://github.com/osbuild/release-action
[packit-dev]: https://packit.dev/docs/
[osbuild/osbuild.github.io]: https://github.com/osbuild/osbuild.github.io
[koji]: https://koji.fedoraproject.org
[bodhi]: https://bodhi.fedoraproject.org/
[recent-releases]: https://github.com/osbuild/osbuild-composer/tags
