# RPM Repository Snapshots

For reliable continuous development, the OSBuild project employs its own RPM
repository snapshots. These snapshots are persistent and immutable. They are
meant to be used by test farms, CI systems, and other development tools, in
case the official RPM repositories are not suitable.


> **WARNING**: These snapshots are not meant for production use! No guarantee of
safety, applicability, or fitness for a particular purpose is made. No security
fixes are applied to the repositories!


## Target Repositories

The authoritative list of repositories that we target can be found in the
`./repo/` subdirectory of the `rpmrepo` repository:

    https://github.com/osbuild/rpmrepo/tree/main/repo

This directory contains a configuration for each target repository, including
the Base-URL that will be sourced for snapshots. The following list contains an
overview (possibly outdated) of the repositories we create snapshots for:

| Platform      | Version | Architectures                    | Lifetime   |
| ------------- |:-------:|:--------------------------------:| ---------- |
| Fedora        | 31      | x86\_64                          | (obsolete) |
| Fedora        | 32      | x86\_64                          | (obsolete) |
| Fedora        | 33      | x86\_64                          | 12 months  |
| Fedora        | 34      | x86\_64                          | 12 months  |
| Fedora        | 35      | x86\_64                          | 12 months  |
| RHEL          | 8.2     | aarch64, ppc64le, s390x, x86\_64 | infinite?  |
| RHEL          | 8.3     | aarch64, ppc64le, s390x, x86\_64 | infinite?  |
| RHEL          | 8.4     | aarch64, ppc64le, s390x, x86\_64 | infinite?  |
| RHEL          | 8.5     | aarch64, ppc64le, s390x, x86\_64 | infinite?  |
| RHEL          | 9.0     | aarch64, ppc64le, s390x, x86\_64 | infinite?  |

Each target repository has an ID-string that identifies it (which also is the
filename of its target configuration file in the `./repo/` directory). Whenever
a snapshot is created, the snapshot will be identified by that ID-string
suffixed with the date it was created (and possibly some other suffix
identifiers).

An enumeration of all available snapshots of all target repositories can be
retrieved via:

    $ curl -s https://rpmrepo.osbuild.org/v2/enumerate | jq .

For a given target repository ID-string like `el9-x86_64-baseos-n9.0`, the list
of available snapshots can be queried via:

    $ curl -s https://rpmrepo.osbuild.org/v2/enumerate/el9-x86_64-baseos-n9.0 | jq .

## Usage

We provide an RPM repository for every snapshot, accessible via
`rpmrepo.osbuild.org`. The *Base URL* for a given snapshot is:

    https://rpmrepo.osbuild.org/v2/mirror/<storage>/<platform>/<snapshot>/

The parameters are:

| Key            | Value                   | Examples                     |
| -------------- |:-----------------------:|:----------------------------:|
| *\<storage\>*  | *public*, *rhvpn*       | *public*, *rhvpn*            |
| *\<platform\>* | *f\<num\>*, *el\<num\>* | *f33*, *el8*                 |
| *\<snapshot\>* | *\<tag\>*               | *f33-x86\_64-devel-20201010* |

The *storage* key selects the actual data store. Available storage includes
*public* for the anonymous, public storage, *rhvpn* for data on Red Hat private
infrastructure. The *platform* key groups the data by platform, required for
data lifetime management. The *snapshot* key selects the individual snapshot.

Note that not all data is available on all storage locations and platforms. If
you select the wrong combination, you will get *404* replies. As a general
rule, you should select the platform based on the snapshot name (e.g., for the
snapshot *f33-x86\_64-devel-20201010* you should use *f33* as platform). As
storage selector, you should use *public* for all publicly available data,
*rhvpn* for Red Hat internal data.

For instance, to access the F33 snapshot *f33-x86\_64-fedora-202103231401*, use:

    https://rpmrepo.osbuild.org/v2/mirror/public/f33/f33-x86_64-fedora-202103231401/

To access the EL8.2 snapshot *el8-x86\_64-baseos-r8.2-202103231359*, use:

    https://rpmrepo.osbuild.org/v2/mirror/rhvpn/el8/el8-x86_64-baseos-r8.2-202103231359/

## Access

By default, all snapshots are publicly available, unless they contain
confidential or proprietary data. If you decide to use these snapshots, please
contact the OSBuild Developers and give us a short notice, so we can track the
users and communicate upcoming changes:

* **RPMrepo Issue Tracker**: [@GitHub](https://github.com/osbuild/rpmrepo/issues)
