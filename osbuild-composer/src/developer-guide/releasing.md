# Releasing

This guide describes the process of releasing osbuild and osbuild-composer to [upstream][upstream-git], into [Fedora][fedora-distgit] and [CentOS Stream][centos-distgit].

## Clone the release helper

Go to the [maintainer-tools repository][maintainer-tools], clone the repository and run `pip install -r requirements.txt` in order to get all the dependencies to be able to execute the `release.py` script.

## Upstream release

Navigate to your local repository in your terminal and call the `release.py` script. It will interactively take you through the following steps:

1. Create a release branch
2. Gather all changes in the `NEWS.md` file and bump the version in `osbuild.spec` or `osbuild-composer.spec` (and potentially `setup.py`)
3. Push your changes to your fork and create a pull request
4. Create a signed tag for the release
5. Create a release on GitHub

## Fedora release

We use packit (see `.packit.yml` in the osbuild or osbuild-composer repository respectively or the [official packit documentation][packit-dev]) to automatically create pull requests against all Fedora releases based on our upstream releases.

Hence all you need to do is to navigate to [Fedora package sources][src-fedora] once your release commit is merged to `main` and processed by GitHub Actions and merge the pull request. Please note that in order to merge the new release into the active Fedora releases, you need to be [a Fedora packager][new-fedora-packager] and have commit rights for [the repository][fedora-distgit].

Next continue with the `release.py` script:

1. Get a kerberos ticket by running `kinit $USER@FEDORAPROJECT.ORG`
2. Call `packit build` from your local repository
3. Check [osbuild koji][koji] or [osbuild-composer][koji-composer] for the successful build of your release

## CentOS Stream 9 release

There's a wonderful guide on this topic in the section 5 of RHEL Developer Guide. `update-distgit.py` from the [osbuild-composer repository][update-distgit] can also save you some time here. The only differences from Fedora are that `--pkgtool centos` and `--release c9s` flags need to be used.

At the time of writing this document, gating tests are not run in CentOS dist-git. You also have to simultaneously open a PR in RHEL dist-git to verify that the test suite passes. Close this PR once the PR in CentOS dist-git is merged.

## Spreading the word on osbuild.org

The last of releasing a new version is to create a new post on osbuild.org. Just open a PR in [osbuild/osbuild.github.io]. You can find a lot of inspiration in existing release posts.

[upstream-git]: https://github.com/osbuild/osbuild
[fedora-distgit]: https://src.fedoraproject.org/rpms/osbuild
[centos-distgit]: https://gitlab.com/redhat/centos-stream/rpms/osbuild
[maintainer-tools]: https://github.com/osbuild/maintainer-tools
[packit-dev]: https://packit.dev/docs/
[src-fedora]: https://src.fedoraproject.org/rpms/osbuild/pull-requests
[new-fedora-packager]: https://fedoraproject.org/wiki/Join_the_package_collection_maintainers
[osbuild/osbuild.github.io]: https://github.com/osbuild/osbuild.github.io
[koji]: https://koji.fedoraproject.org/koji/packageinfo?packageID=29756
[koji-composer]: https://koji.fedoraproject.org/koji/packageinfo?packageID=31032
[update-distgit]: https://github.com/osbuild/osbuild-composer/blob/main/tools/update-distgit.py
