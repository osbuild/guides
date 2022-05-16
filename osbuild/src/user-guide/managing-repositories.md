# Managing repositories

There are two kinds of repositories used in osbuild-composer:
 1. **Custom 3rd party repositories** - use these to include packages that are not available in the official Fedora or RHEL repositories.
 2. **Official repository overrides** - use these if you want to download base system RPMs from elsewhere than the official repositories. For example if you have a custom mirror in your network. Keep in mind that this will **disable the default repositories**, so the mirror must contain all necessary packages!

## Custom 3rd party repositories

These are managed using `composer-cli` (see the manpage for complete reference). To add a new repository, create a `TOML` file like this:
```toml
id = "k8s"
name = "Kubernetes"
type = "yum-baseurl"
url = "https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64"
check_gpg = false
check_ssl = false
system = false
```
and add it using `composer-cli sources add <file-name.toml>`. Verify its presence using `composer-cli sources list` and its content using `composer-cli sources info <id>`.

### Using sources with specific distributions

A new optional field has been added to the repository source format. It is a
list of distribution strings that the source will be used with when depsolving
and building images.

Sources with no `distros` will be used with all composes. If you want to use a
source for a specific distro you set the `distros` list to the distro name(s)
to use it with.

eg. A source that is only used when depsolving or building fedora 32:

```toml
check_gpg = true
check_ssl = true
distros = ["fedora-32"]
id = "f32-local"
name = "local packages for fedora32"
system = false
type = "yum-baseurl"
url = "http://local/repos/fedora32/projectrepo/"
```

This source will be used for any requests that specify fedora-32, eg. listing
packages and specifying fedora-32 will include this source, but listing
packages for the host distro will not.


## Official repository overrides

`osbuild-composer` does not inherit the system repositories located in `/etc/yum.repos.d/`. Instead, it has its own set of official repositories defined in `/usr/share/osbuild-composer/repositories`. To override the official repositories, define overrides in `/etc/osbuild-composer/repositories`. This directory is meant for user defined overrides and the files located here take precedence over those in `/usr`. 

The configuration files are not in the usual "repo" format. Instead, they are simple `JSON` files.

### Defining official repository overrides

To set your own repositories, create this directory if it does not exist already:

```
$ sudo mkdir -p /etc/osbuild-composer/repositories
```

Based on the system you want to build an image for, determine the name of a new JSON file:

* Fedora 32 - `fedora-32.json`
* Fedora 33 - `fedora-33.json`
* RHEL 8.4 - `rhel-84.json`
* RHEL 9.0 - `rhel-90.json`

Then, create the JSON file with the following structure (or copy the file from `/usr/share/osbuild-composer/` and modify its content):

```json
{
    "<ARCH>": [
        {
            "name": "<REPO NAME>",
            "metalink": "",
            "baseurl": "",
            "mirrorlist": "",
            "gpgkey": "",
            "check_gpg": "",
            "metadata_expire": "",
        }
    ]
}
```
Specify only one of the following attributes: `metalink`, `mirrorlist`, or `baseurl`. All the remaining fields like `gpgkey`, `metadata_expire`, etc. are optional.

For example, for building a Fedora 33 image running on x86_64, create `/etc/osbuild-composer/repositories/fedora-33.json` with this content:
```json
{
    "x86_64": [
        {
            "name": "fedora",
            "metalink": "https://mirrors.fedoraproject.org/metalink?repo=fedora-33&arch=x86_64",
            "gpgkey": "-----BEGIN PGP PUBLIC KEY BLOCK-----\n\nmQINBF4wBvsBEADQmcGbVUbDRUoXADReRmOOEMeydHghtKC9uRs9YNpGYZIB+bie\nbGYZmflQayfh/wEpO2W/IZfGpHPL42V7SbyvqMjwNls/fnXsCtf4LRofNK8Qd9fN\nkYargc9R7BEz/mwXKMiRQVx+DzkmqGWy2gq4iD0/mCyf5FdJCE40fOWoIGJXaOI1\nTz1vWqKwLS5T0dfmi9U4Tp/XsKOZGvN8oi5h0KmqFk7LEZr1MXarhi2Va86sgxsF\nQcZEKfu5tgD0r00vXzikoSjn3qA5JW5FW07F1pGP4bF5f9J3CZbQyOjTSWMmmfTm\n2d2BURWzaDiJN9twY2yjzkoOMuPdXXvovg7KxLcQerKT+FbKbq8DySJX2rnOA77k\nUG4c9BGf/L1uBkAT8dpHLk6Uf5BfmypxUkydSWT1xfTDnw1MqxO0MsLlAHOR3J7c\noW9kLcOLuCQn1hBEwfZv7VSWBkGXSmKfp0LLIxAFgRtv+Dh+rcMMRdJgKr1V3FU+\nrZ1+ZAfYiBpQJFPjv70vx+rGEgS801D3PJxBZUEy4Ic4ZYaKNhK9x9PRQuWcIBuW\n6eTe/6lKWZeyxCumLLdiS75mF2oTcBaWeoc3QxrPRV15eDKeYJMbhnUai/7lSrhs\nEWCkKR1RivgF4slYmtNE5ZPGZ/d61zjwn2xi4xNJVs8q9WRPMpHp0vCyMwARAQAB\ntDFGZWRvcmEgKDMzKSA8ZmVkb3JhLTMzLXByaW1hcnlAZmVkb3JhcHJvamVjdC5v\ncmc+iQI4BBMBAgAiBQJeMAb7AhsPBgsJCAcDAgYVCAIJCgsEFgIDAQIeAQIXgAAK\nCRBJ/XdJlXD/MZm2D/9kriL43vd3+0DNMeA82n2v9mSR2PQqKny39xNlYPyy/1yZ\nP/KXoa4NYSCA971LSd7lv4n/h5bEKgGHxZfttfOzOnWMVSSTfjRyM/df/NNzTUEV\n7ORA5GW18g8PEtS7uRxVBf3cLvWu5q+8jmqES5HqTAdGVcuIFQeBXFN8Gy1Jinuz\nAH8rJSdkUeZ0cehWbERq80BWM9dhad5dW+/+Gv0foFBvP15viwhWqajr8V0B8es+\n2/tHI0k86FAujV5i0rrXl5UOoLilO57QQNDZH/qW9GsHwVI+2yecLstpUNLq+EZC\nGqTZCYoxYRpl0gAMbDLztSL/8Bc0tJrCRG3tavJotFYlgUK60XnXlQzRkh9rgsfT\nEXbQifWdQMMogzjCJr0hzJ+V1d0iozdUxB2ZEgTjukOvatkB77DY1FPZRkSFIQs+\nfdcjazDIBLIxwJu5QwvTNW8lOLnJ46g4sf1WJoUdNTbR0BaC7HHj1inVWi0p7IuN\n66EPGzJOSjLK+vW+J0ncPDEgLCV74RF/0nR5fVTdrmiopPrzFuguHf9S9gYI3Zun\nYl8FJUu4kRO6JPPTicUXWX+8XZmE94aK14RCJL23nOSi8T1eW8JLW43dCBRO8QUE\nAso1t2pypm/1zZexJdOV8yGME3g5l2W6PLgpz58DBECgqc/kda+VWgEAp7rO2A==\n=EPL3\n-----END PGP PUBLIC KEY BLOCK-----\n",
            "check_gpg": true
        }
    ]
}
```

## Using repositories that require subscription

`osbuild-composer` can use subscriptions from the host system if they are configured in the appropriate file in `/etc/osbuild-composer/repositories`. To enable such repository, copy the `baseurl` from `/etc/yum.repos.d/redhat.repo` and paste it into the JSON repository definition. Then allow RHSM support using `"rhsm": true` like this:

```json
{
  "x86_64": [
    {
      "baseurl": "https://localhost/repo",
      "gpgkey": "...",
      "rhsm": true
    }
  ]
}
```

`osbuild-composer` will read the `/etc/yum.repos.d/redhat.repo` file from the host system and use it as a source of subscriptions. The same subscriptions must be available on a remote worker, if used.
