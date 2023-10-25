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
```

and add it using `composer-cli sources add <file-name.toml>`. Verify its presence using `composer-cli sources list` and its content using `composer-cli sources info <id>`.

Valid values for the `type` field are: `yum-baseurl`, `yum-mirrorlist`, and `yum-metalink`.

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
type = "yum-baseurl"
url = "http://local/repos/fedora32/projectrepo/"
```

This source will be used for any requests that specify fedora-32, eg. listing
packages and specifying fedora-32 will include this source, but listing
packages for the host distro will not.

### Verifying Repository Metadata with GPG

In addition to checking the GPG signature on rpm packages, DNF can check that
repository metadata has been signed with a GPG key. You can setup such a
repository yourself by signing your `repomd.xml` file after you have run
`createrepo_c` on your repository. For example:

```bash
cd repo/
createrepo_c .
cd repodata/
gpg -u YOUR-GPG-KEY-EMAIL --yes --detach-sign --armor repomd.xml
```

In order to check this signature you need to tell osbuild-composer what gpg key
to use to do the check.  Set `check_repogpg = true` in the source, and if the
key is available over https, set the `gpgkeys` entry to the URL for the key,
like this:

```toml
check_gpg = true
check_ssl = true
id = "custom-local"
name = "signed local packages"
type = "yum-baseurl"
url = "https://local/repos/projectrepo/"
check_repogpg = true
gpgkeys=["https://local/keys/repokey.pub"]
```

Normally you would want to distribute the key via a separate channel from the
rpms for better security, the above is just an example. You can also embed the
whole key into the source `gpgkeys` entry. If the entry starts with `-----BEGIN
PGP PUBLIC KEY BLOCK-----` it will import it directly instead of fetching it
from the url. For example:

```toml
check_gpg = true
check_ssl = true
check_repogpg
id = "custom-local"
name = "signed local packages"
type = "yum-baseurl"
url = "https://local/repos/projectrepo/"
gpgkeys=["https://remote/keys/other-repokey.pub",
'''-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v1.4.10 (GNU/Linux)

mQENBEt+xXMBCACkA1ZtcO4H7ZUG/0aL4RlZIozsorXzFrrTAsJEHvdy+rHCH3xR
cFz6IMbfCOdV+oKxlDP7PS0vWKfqxwkenOUut5o9b32uDdFMW4IbFXEQ94AuSQpS
jo8PlVMm/51pmmRxdJzyPnr0YD38mVK6qUEYLI/4zXSgFk493GT8Y4m3N18O/+ye
PnOOItj7qbrCMASoBx1TG8Zdg8ufehMnfb85x4xxAebXkqJQpEVTjt4lj4p6BhrW
R+pIW/nBUrz3OsV7WwPKjSLjJtTJFxYX+RFSCqOdfusuysoOxpIHOx1WxjGUOB5j
fnhmq41nWXf8ozb58zSpjDrJ7jGQ9pdUpAtRABEBAAG0HkJyaWFuIEMuIExhbmUg
PGJjbEByZWRoYXQuY29tPokBOAQTAQIAIgUCS37FcwIbAwYLCQgHAwIGFQgCCQoL
BBYCAwECHgECF4AACgkQEX6MFo7+On9dgAf9Hi2K1MKcmLkDeSUIXkXIAw0nAzl2
UDGLWEdDqAgFxP6UaCVtOIRCr7z4EDOQoxD7mkdekbH2W5GcTO4h8MQBHYD9EkY7
H/lTKchlFfsmafOoA3Y/tDLPKu+OIfH9Mqn2Mf7wMYGrnWSRNKYgvC5zkMgkhoPU
mSPPHyBabsdS/Kg5ZAf43ac/MXY9V8Mk6zqbBlj6QYqjJ0nBD6vwozrDQ5gJtDUL
mQho13zPn4lBJl9YJVjcgRB2WbzgSZOln0DfV22Seai66vnr5NyaOIw5B9QLSNhN
EaPFswEDLKCsns9dkDuGFX52/Mt/i7JySvwhMBqHElPzWmwCHeY45M8gBYhGBBAR
AgAGBQJLfsbpAAoJECH7Y/6XEsLNuasAn0Q0jB4Ea/95EREUkCFTm9L6nOpAAJ9t
QzwGXhrLFZzOdRWYiWcCQbX5/7kBDQRLfsVzAQgAvN5jr95pJthv2w9co9/7omhM
5rAnr9WJfbMLLiUfPPUvpL24RGO6SKy03aiVTUjlaHc+cGqOciwnNKMCSt+noyG2
kNnAESTDtCivpsjonaFP8jA3TqL0QK+yzBRKJnMnLEY1nWE1FtkMRccXvzi0Z/XQ
VhiWQyTvDFoKtepBFrH9UqWbNHyki22aighumUsW01pcPH2ogSj+HR01r7SfI/y2
EkE6loHQfCDycHmlqYV+X6GZEvf1qu2+EHEQChsHIAxWyshsxM/ZPmx/8e5S3Xmj
l7h/6E9wcsIpvnf504sLX5j4Km9I5HgJSRxHxgRPpqJ2/XiClAJanO5gCw0RdQAR
AQABiQEfBBgBAgAJBQJLfsVzAhsMAAoJEBF+jBaO/jp/SqEH/iArzrfVOhZQGuy1
KmG0+/FdJGqAEHP5HWpsaeYJok1VmhTPZd4IVFBz/bGJYyvsrPU0pJ6QLkdGxNnb
KulJocgkW5MKEL/CRc54ESKwYngigmbY4qLwhS+gB3BJg1TvoHD810MSj4wdxNNo
6JQmFmuoDsLRwaRYbKQDz95XXoGQtmV1o57T05WkLuC5OmHqnWv3rggVC8madpUJ
moUUvUWgU1qyXe3PrgMGFOibWIl7lPZ08nzKXBRvSK/xoTGxl+570AevfVHMu5Uk
Yu2U6D6/DYohtTYp0s1ekS5KQkCJM7lfqecDsQhfVfOfR0w4aF8k8u3HmWdOfUz+
9+2ZsBo=
=myjM
-----END PGP PUBLIC KEY BLOCK-----''']
```

Notice that gpgkeys can take as many key urls and keys as you need, not just one.
If the signature cannot be found an error similar to this will be returned:

```
GPG verification is enabled, but GPG signature is not available.
This may be an error or the repository does not support GPG verification:
Status code: 404 for http://repo-server/fedora/repodata/repomd.xml.asc (IP: 192.168.1.3)
```

And if the signature is invalid:

```
repomd.xml GPG signature verification error: Bad GPG signature
```

You can test the signature of the repository manually by running `gpg --verify repomd.xml.asc`
to help troubleshoot problems.


## Official repository overrides

`osbuild-composer` does not inherit the system repositories located in `/etc/yum.repos.d/`. Instead, it has its own set of official repositories defined in `/usr/share/osbuild-composer/repositories`. To override the official repositories, define overrides in `/etc/osbuild-composer/repositories`. This directory is meant for user defined overrides and the files located here take precedence over those in `/usr`.

The configuration files are not in the usual "repo" format. Instead, they are simple `JSON` files.

### Defining official repository overrides

To set your own repositories, create this directory if it does not exist already:

```bash
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

After you have created repository overrides in `/etc/osbuild-composer/repositories`, you must restart the `osbuild-composer` service in order for the overrides to take effect.

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
