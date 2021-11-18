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

Then our centos-bot takes over and performs the following steps:

1. Check if there is a more recent release in Fedora rawhide than in CentOS Stream 9
2. Update dist-git in schutzbot's respective fork with the latest release
3. Propose a merge request against the main repository

The following steps are manual and not part of the centos-bot automation. Before starting these steps, make sure you have
the required tools installed, i.e. `centpkg`,`rhelpkg` (see the developer guide prerequisites) for more details). Additionally, you can view the centos-bot Dockerfile for reference.  
The remaining steps are listed in more detail below:

1. Go to the merge request, open the Pipeline tab and click "Run pipeline" (schutzbot is an external contributor, so CI is blocked by default)
2. Fork the OSCI pagure repo
3. Cherry-pick the commit created by schutzbot and push it to OSCI's pagure
4. Download the tarball of the release and run `rhpkg new-sources <path-to-tarball>` (this will upload the tarball to the lookaside cache)
5. Create a pull request on OSCI's pagure
6. Wait for the tests to finish, check both your pull request and the OSCI Dashboard for gating test results
7. If all tests are green (or: okay/waivable) proceed with closing your PR on OSCI's pagure
8. Merge schutzbot's merge request on gitlab.com
9. Run `centpkg clone $project` to get the latest `c9s` branch (or a simple `git pull` if you already have a local copy)
10. Lastly, run `centpkg build` from the `c9s` branch of the project

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
