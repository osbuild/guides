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

Then our [fedora-bot][fedora-bot] takes over and performs the remaining steps:

1. Get a kerberos ticket by running `kinit $USER@FEDORAPROJECT.ORG`
2. Call `fedpkg build` to schedule Koji builds for each active Fedora release (or: dist-git branch)
3. Update [Bodhi][bodhi] with the latest release

## CentOS Stream 9 release

Then our centos-bot takes over and performs the following steps:

1. Check if there is a more recent release in Fedora rawhide than in CentOS Stream 9
2. Update dist-git in schutzbot's respective fork with the latest release
3. Propose a merge request against the main repository

The following steps are manual and not part of the centos-bot automation. Before starting these steps, make sure you have
the required tools installed, i.e. `centpkg`,`rhelpkg` (see the developer guide prerequisites for more details). Additionally, you can view the centos-bot Dockerfile for reference.  
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

## RHEL 8.y release

This part describes how to release into the RHEL 8 minor version under development (i.e., the next minor release). Instructions for releasing into a released version of RHEL 8 that is still supported will be added in a later section.

At the time of this writing (2021-12-01), the next minor RHEL release is RHEL 8.6, so we will be using that as an example.

### Prerequisites

Pushing an update to RHEL requires a bug in Bugzilla or Jira that the release will address. In the early stages of the development cycle for a new minor RHEL release, we usually create a bug that simply says "Release osbuild/osbuild-composer to RHEL 8.y". Any bug can be used to mark a release, as long as the bug applies to the changes in the release (in other words, the release fixes the problem described in the bug) and the appropriate _flags_ are set.


### OSCI Pagure

This part is similar to the manual steps of the CentOS Stream 9 release process.

1. Fork the OSCI repository for the project (osbuild or osbuild-composer) and clone it locally.
2. Clone the OSCI repository for the project.
3. Switch to the minor version branch, e.g., `rhel-8.6.0`.
4. Download the source tarball from the GitHub release and place it in the root of the cloned repository.
5. Run `rhpkg new-sources <path-to-tarball>`.
6. Update the `.spec` file to match the upstream version. You can copy the upstream file directly, but make sure to modify the change log to match the downstream convention.
    - Our upstream `.spec` files usually don't contain anything in the change log.
    - The downstream change log contains one entry per release. Add a new entry with date, name, and version number, to match the existing ones.
7. Add and commit the changes. **The commit message must reference at least one eligible bug** (see [Prerequisites](#prerequisites) section above) in the form `rhbz#nnnnnn`. See the git log for examples.
8. Push the changes to your fork and open a pull request against the minor version branch in the origin repository.
9. OSCI will run a series of tests and check the referenced bug for eligibility. Once everything is green, the PR can be merged.
10. Once merged, make sure the changes in your local clone are up-to-date and run: `rhpkg build`.
    - Make note of the Brew build URL that is printed in the output of the command.

### Advisories

Once the build is finished from the previous section, a new Advisory will be created if one does not exist already and can be found on the [Red Hat Errata Tool][errate-tool]. The purpose of this Advisory is to associate package builds with metadata and prepare it for release to customers.
If the build was successful, you will be able to find the Advisory on the [advisories tab][advisories-tab] of the site. You can search or add a filter to match the project name.

1. To proceed with the release, the new build must be added to the Advisory. This should happen automatically, but if it doesn't, you can add it by first changing the state of the Advisory to "New Files" (using the Change Stage button at the top) and then adding the build clicking on the View/Modify button in the Builds section.
    - The build must be referenced by its full NEVRA (Name, Epoch, Version, Release, Architecture), which you can find in the Brew build that was triggered from `rhpkg build`.
2. The bug should have also been added automatically to the Advisory. If it wasn't you can add them from the Advisory's main view by clicking the Add Bugs button.
    - Note that only eligible bugs appear in the list when adding bugs. If a bug isn't listed, it's probably in the incorrect state. You can verify which flags are in an incorrect state by using the [Bug Eligibility Tool][bug-eligibility] of the Errata Tool.
3. Review the Advisory main view. It shows the Approval Progress of the release. This will often point to test results that require manual review.
4. RPMDiff tests: Click on View in the RPMDiff column (or the RPMDiff tab at the top) and then the RPMDiff ID for a failed test. There you can review results that are marked "Failed" (red) or "Needs inspection" (yellow) and Waive them with an appropriate explanation. If it doesn't make sense to waive a result (e.g., it's a critical bug), the release should be cancelled.
5. Covscan tests: Follow the same process as with the RPMDiff tests.
6. Once both RPMDiff and Covscan are green, the state of the Advisory can be changed to "QE" using the Change Stage button again or the Move to QE button.


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
[fedora-bot]: https://github.com/osbuild/fedora-bot
[recent-releases]: https://github.com/osbuild/osbuild-composer/tags
[errate-tool]: https://errata.devel.redhat.com
[advisories-tab]: https://errata.devel.redhat.com/errata
[bug-eligibility]: https://errata.devel.redhat.com/bugs/troubleshoot
