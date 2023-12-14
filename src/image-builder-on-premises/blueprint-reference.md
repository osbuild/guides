# Blueprint Reference

Blueprints are text files in the [TOML format](https://toml.io/en/) that describe customizations for the image you are building.

> An important thing to note is that these customizations are not applicable to all image types. `osbuild-composer` currently has no good validation or warning system in place to tell you if a customization in your blueprint is not supported for the image type you're building. The customization may be silently dropped.

A very basic blueprint with just the required attributes at the root looks like:

```toml
name = "basic-example"
description = "A basic blueprint"
version = "0.0.1"
```

Where:
- The `name` attribute is a string that contains the name of the blueprint. It can contain spaces, but they will be converted to `-` when it is import into `osbuild-composer`. It should be short and descriptive.
- The `description` attribute is a string that can be a longer description of the blueprint and is only used for display purposes.
- The `version` attribute is a string that contains a semver compatible version number. If a new blueprint is uploaded with the same version the server will automatically bump the PATCH level of the version. If the version doesn't match it will be used as is. For example, uploading a blueprint with version set to 0.1.0 when the existing blueprint version is 0.0.1 will result in the new blueprint being stored as version 0.1.0.

You can upload a blueprint with the `osbuild-composer blueprints push $filename` command, the blueprint will then be usable in `osbuild-composer compose` as the `name` you gave it.

Blueprints have two main sections, the [content](#content) and [customizations](#customizations) sections.

### Distribution selection with blueprints

The blueprint now supports a new `distro` field that will be used to select the
distribution to use when composing images, or depsolving the blueprint. If
`distro` is left blank it will use the host distribution. If you upgrade the
host operating system the blueprints with no `distro` set will build using the
new os. You can't build an OS image that differs from the host OS that Image Builder lives on.

eg. A blueprint that will always build a Fedora 38 image, no matter what
version is running on the host:

```toml
name = "tmux"
description = "tmux image with openssh"
version = "1.2.16"
distro = "fedora-38"

[[packages]]
name = "tmux"
version = "*"

[[packages]]
name = "openssh-server"
version = "*"
```

## Content

The content section determines what goes into the image from other sources such as packages, package groups, or containers. Content is defined at the root of the blueprint.

- [Packages](#packages).
- [Groups](#groups).
- [Containers](#containers).

### Packages

The `packages` and `modules` lists contain objects with a `name` and optional `version` attribute.

- The `name` attribute is a **required** string and can be an exact match, or a filesystem-like glob using `*` for wildcards and `?` for character matching.
- The `version` attribute is an *optional* string can be an exact match or a filesystem-like glob of the version using `*` for wildcards and `?` for character matching. If not provided the latest version in the repositories is used.

*Currently there are no differences between packages and modules in `osbuild-composer`. Both are treated like an rpm package dependency.*

> When using virtual provides as the package name the version glob should be `*`. And be aware that you will be unable to `freeze` the blueprint. This is because the provide will expand into multiple packages with their own names and versions.

For example, to install `tmux-2.9a` and `openssh-server-8.*` packages, add this to your blueprint:

```toml
[[packages]]
name = "tmux"
version = "2.9a"

[[packages]]
name = "openssh-server"
version = "8.*"
```

Or in alternative syntax:

```toml
packages = [
    { name = "tmux", version = "2.9a" },
    { name = "openssh-server", version = "8.*" }
]
```

### Groups

The `groups` list describes contains objects with a `name`-attribute.
- The `name` attribute is a **required** string and must match the id of a package group in the repositories exactly.

`groups` describes groups of packages to be installed into the image. Package groups are defined in the repository metadata. Each group has a descriptive name used primarily for display in user interfaces and an ID more commonly used in kickstart files. Here, the ID is the expected way of listing a group. Groups have three different ways of categorizing their packages: mandatory, default, and optional. For the purposes of blueprints, only mandatory and default packages will be installed. There is no mechanism for selecting optional packages.

For example, if you want to install the `anaconda-tools` group, add the following to your blueprint:

```toml
[[groups]]
name = "anaconda-tools"
```

Or in alternative syntax:

```toml
groups = [
    { name = "anaconda-tools" }
]
```

### Containers

The `containers` list contains objects with a `source` and optional `tls-verify` attribute.

These list entries describe the container images to be embedded into the image.

- The `source` attribute is a **required** string and is a reference to a container image at a registry.
- The `name` attribute is an *optional* string to set the name under which the container image will be saved in the image. If not specified `name` falls back to the same value as `source`.
- The `tls-verify` attribute is an *optional* boolean to disable TLS verification of the source download. By default this is set to `true`.

The container is pulled during the image build and stored in the image at the default local container storage location that is appropriate for the image type, so that all supported container tools like `podman` and `cri-o` will be able to work with it.

The embedded containers are not started, to do so you can create systemd unit files or quadlets with the files customization.

To embed the latest fedora container from http://quay.io, add this to your blueprint:

```toml
[[containers]]
source = "quay.io/fedora/fedora:latest"
```

Or in alternative syntax:

```toml
containers = [
    { source = "quay.io/fedora/fedora:latest" },
    { source = "quay.io/fedora/fedora-minimal:latest", tls-verify = false, name = "fedora-m" },
]
```

To access protected container resources a `containers-auth.json(5)` file can be used, see [Container registry credentials](../user-guide/container-auth.md).


## Customizations

In the customizations we determine what goes into the image that's not in the default packages defined under [Content](#content).

- [Hostname](#hostname)
- [Kernel Command Line Arguments](#kernel-command-line-arguments)
- [SSH Keys](#ssh-keys)
- [Additional Users](#additional-users)
- [Additional Groups](#additional-groups)
- [Timezone](#timezone)
- [Locale](#locale)
- [Firewall](#firewall)
- [Systemd Services](#systemd-services)
- [Files and Directories](#files-and-directories)
  - [Directories](#directories)
  - [Files](#files)
- [Installation device](#installation-device)
- [Ignition](#ignition)
- [Repositories](#repositories)
- [Filesystems](#filesystems)
- [OpenSCAP](#openscap)
- [FIPS](#fips)

### Hostname

`customizations.hostname` is an *optional* string that can be used to configure the hostname of the final image:

```toml
[customizations]
hostname = "baseimage"
```

This is optional and can be left out to use the default hostname.

### Kernel

#### Kernel Command-Line Arguments

An *optional* string that allows to append arguments to the bootloader kernel command line:

```toml
[customizations.kernel]
append = "nosmt=force"
```

### SSH Keys

An *optional* list of objects containing:

- The `user` attribute is a **required** string and must match the name of a user in the image exactly.
- The `key` attribute is a **required** string that contains the public key to be set for that user.

*Warning: `key` expects the entire content of the public key file, traditionally `~/.ssh/id_rsa.pub` but any algorithm supported by the operating system in the image is valid*

*Note: If you are adding a user you can add their SSH key in the [additional users](#additional-users) customization instead.*

Set an existing user's SSH key in the final image:

```toml
[[customizations.sshkey]]
user = "root"
key = "PUBLIC SSH KEY"
```
The key will be added to the user's `authorized_keys` file in their home directory.

### Additional Users

An *optional* list of objects that contain the following attributes:

- `name` a **required** string that sets the username.
- `description` an *optional* string.
- `password` an *optional* string.
- `key` an *optional* string.
- `home` an *optional* string.
- `shell` an *optional* string.
- `groups` an *optional* list of strings.
- `uid` an *optional* integer.
- `gid` an *optional* integer.

*Warning: `key` expects the entire content of the public key file, traditionally `~/.ssh/id_rsa.pub` but any algorithm supported by the operating system in the image is valid*

*Note: If the password starts with $6$, $5$, or $2b$ it will be stored as an encrypted password. Otherwise it will be treated as a plain text password.*

Add a user to the image, and/or set their ssh key. All fields for this section are optional except for the name. The following is a complete example:

```toml
[[customizations.user]]
name = "admin"
description = "Administrator account"
password = "$6$CHO2$3rN8eviE2t50lmVyBYihTgVRHcaecmeCk31L..."
key = "PUBLIC SSH KEY"
home = "/srv/widget/"
shell = "/usr/bin/bash"
groups = ["widget", "users", "wheel"]
uid = 1200
gid = 1200
```

### Additional groups

An *optional* list of objects that contain the following attributes:

- `name` a **required** string that sets the name of the group.
- `gid` a **required** integer that sets the id of the group.

```toml
[[customizations.group]]
name = "widget"
gid = 1130
```

### Timezone

An *optional* object that contains the following attributes:

- `timezone` an *optional* string. If not provided the UTC timezone is used..
- `ntpservers` an *optional* list of strings containing NTP servers to use. If not provided the distribution defaults are used.

```toml
[customizations.timezone]
timezone = "US/Eastern"
ntpservers = ["0.north-america.pool.ntp.org", "1.north-america.pool.ntp.org"]
```

The values supported by timezone can be listed by running the command:

```
$ timedatectl list-timezones
```

Some image types have already NTP servers setup such as Google Cloud images. These cannot be overridden because they are required to boot in the selected environment. However, the timezone will be updated to the one selected in the blueprint.

### Locale

An *optional* object that contains the following attributes to customize the locale settings for the system:

- `languages` an *optional* list of strings containing locales to be installed.
- `keyboard` an *optional* string to set the keyboard layout.

Multiple languages can be added. The first one becomes the primary, and the others are added as secondary. You must include one or more languages or keyboards in the section.

```toml
[customizations.locale]
languages = ["en_US.UTF-8"]
keyboard = "us"
```

The values supported by languages can be listed by running can be listed by running the command:

```
$ localectl list-locales
```

The values supported by keyboard can be listed by running the command:

```
$ localectl list-keymaps
```

### Firewall

An *optional* object containing the following attributes:

- `ports` an *optional* list of strings containing ports (or port ranges) and protocols to open.
- `services` an *optional* object with the following attributes containing services to enable or disable for `firewalld`.
  - `enabled` *optional* list of strings for services to enable.
  - `disabled` *optional* list of strings for services to disable.

By default the firewall blocks all access, except for services that enable their ports explicitly such as the sshd. The following blueprint can be used to open other ports or services.

*Note: Ports are configured using the `port:protocol` format; port ranges are configured using `portA-portB:protocol` format:*

```toml
[customizations.firewall]
ports = ["22:tcp", "80:tcp", "imap:tcp", "53:tcp", "53:udp", "30000-32767:tcp", "30000-32767:udp"]
```

Numeric ports, or their names from `/etc/services` can be used in the ports enabled/disabled lists.

The blueprint settings extend any existing settings in the image templates. Thus, if sshd is already enabled, it will extend the list of ports with those already listed by the blueprint.

If the distribution uses `firewalld` you can specify services listed by `firewall-cmd --get-services` in a `customizations.firewall.services` section:

```toml
[customizations.firewall.services]
enabled = ["ftp", "ntp", "dhcp"]
disabled = ["telnet"]
```

Remember that the `firewall.services` are different from the names in `/etc/services`.

Both are optional, if they are not used leave them out or set them to an empty list `[]`. If you only want the default firewall setup this section can be omitted from the blueprint.

*Note: The Google and OpenStack templates explicitly disable the firewall for their environment. This cannot be overridden by the blueprint.*

### Systemd Services

An *optional* object containing the following attributes:
- `enabled` an *optional* list of strings containing services to be enabled.
- `disabled` an *optional* list of strings containing services to be disabled.

```toml
[customizations.services]
enabled = ["sshd", "cockpit.socket", "httpd"]
disabled = ["postfix", "telnetd"]
```

This section can be used to control which services are enabled at boot time. Some image types already have services enabled or disabled in order for the image to work correctly, and cannot be overridden. For example, `ami` image type requires `sshd`, `chronyd`, and `cloud-init` services. Without them, the image will not boot. Blueprint services do not replace this services, but add them to the list of services already present in the templates, if any.

The service names are systemd service units. You may specify any systemd unit file accepted by systemctl enable, for example, cockpit.socket:

### Files and directories

You can use blueprint customizations to create custom files and directories in the image. These customizations are currently restricted only to the `/etc` directory.

When using the custom files and directories customization, the following rules apply:

- The path must be an absolute path and must be under `/etc` or `/root`.
- There must be no duplicate paths of the same directory.
- There must be no duplicate paths of the same file.

These customizations are not supported for image types that deploy ostree commits (such as `edge-raw-image`, `edge-installer`, `edge-simplified-installer`). The only exception is the Fedora `iot-raw-image` image type, which supports these customizations.

#### Directories

You can create custom directories by specifying items in the `customizations.directories` list. The existence of a specified directory is handled gracefully only if no explicit `mode`, `user` or `group` is specified. If any of these customizations are specified and the directory already exists in the image, the image build will fail. The intention is to prevent changing the ownership or permissions of existing directories.

The following example creates a directory `/etc/foobar` with all the default settings:

```toml
[[customizations.directories]]
path = "/etc/foobar"
mode = "0755"
user = "root"
group = "root"
ensure_parents = false
```

- `path` is the path to the directory to create. It must be an absolute path under `/etc`. This is the only required field.
- `mode` is the octal mode to set on the directory. If not specified, the default is `0755`. The leading zero is optional.
- `user` is the user to set as the owner of the directory. If not specified, the default is `root`. Can be specified as user name (string) or as user id (integer).
- `group` is the group to set as the owner of the directory. If not specified, the default is `root`. Can be specified as group name (string) or as group id (integer).
- `ensure_parents` is a boolean that specifies whether to create parent directories as needed. If not specified, the default is `false`.

#### Files

You can create custom files by specifying items in the `customizations.files` list. You can use the customization to create new files or to replace existing ones, if not restricted by the policy specified below. If the target path is an existing symlink to another file, the symlink will be replaced by the custom file.

Please note that the parent directory of a specified file must exist. If it does not exist, the image build will fail. One can ensure that the parent directory exists by specifying it in the `customizations.directories` list.

In addition, the following files are not allowed to be created or replaced by policy:

- `/etc/fstab`
- `/etc/shadow`
- `/etc/passwd`
- `/etc/group`

Using the `files` customization comes with a high chance of creating an image that doesn't boot. **Use this feature only if you know what you are doing**. Although the `files` customization can be used to configure parts of the OS which can also be configured by other blueprint customizations, this use is discouraged. If possible, users should always default to using the specialized blueprint customizations. Note that if you combine the files customizations with other customizations, the other customizations may not work as expected or may be overridden by the files customizations.

The following example creates a file `/etc/foobar` with the contents `Hello world!`:

```toml
[[customizations.files]]
path = "/etc/foobar"
mode = "0644"
user = "root"
group = "root"
data = "Hello world!"
```

- `path` is the path to the file to create. It must be an absolute under `/etc`. This is the only required field.
- `mode` is the octal mode to set on the file. If not specified, the default is `0644`. The leading zero is optional.
- `user` is the user to set as the owner of the file. If not specified, the default is `root`. Can be specified as user name (string) or as user id (integer).
- `group` is the group to set as the owner of the file. If not specified, the default is `root`. Can be specified as group name (string) or as group id (integer).
- `data` is the plain text contents of the file. If not specified, the default is an empty file.

Note that the `data` property can be specified in any of the ways supported by TOML. Some of them require escaping certain characters and others don't. Please refer to the [TOML specification](https://toml.io/en/v1.0.0#string) for more details.
### Installation device

The `customizations.installation_device` variable is required by
the `edge-simplified-installer` image. It allows the user to define
the destination device for the installation.

```toml
[customizations]
installation_device = "/dev/sda"
```
### Ignition

The `customizations.ignition` section allows users to provide [Ignition](https://coreos.github.io/ignition/) configuration files to be used in `edge-raw-image` and `edge-simplified-installer` images. Check the RHEL for Edge (`r4e`) [butane](https://coreos.github.io/butane/specs/) specification for a description of the supported configuration options.

The blueprint configuration can be done *either* by embedding an Ignition configuration file into the image (only available for `edge-simplified-installer`), or providing a provisioning URL that will be fetched at first boot.

#### `ignition.embedded` configuration

```toml
[customizations.ignition.embedded]
config = "eyJpZ25pdGlvbiI6eyJ2ZXJzaW9uIjoiMy4zLjAifSwicGFzc3dkIjp7InVzZXJzIjpbeyJncm91cHMiOlsid2hlZWwiXSwibmFtZSI6ImNvcmUiLCJwYXNzd29yZEhhc2giOiIkNiRqZnVObk85dDFCdjdOLjdrJEhxUnhxMmJsdFIzek15QUhqc1N6YmU3dUJIWEVyTzFZdnFwaTZsamNJMDZkUUJrZldYWFpDdUUubUpja2xQVHdhQTlyL3hwSmlFZFdEcXR4bGU3aDgxIn1dfX0="
```

Add a `base64` encoded Ignition configuration in the `config` field. This Ignition configuration will be included in the `edge-simplified-installer` image.

#### `ignition.firstboot` configuration

```toml
[customizations.ignition.firstboot]
url = "http://some-server/configuration.ig"
```

Add a URL pointing to the Ignition configuration that will be fetched during the first boot in the `url` field. Available for both `edge-simplified-installer` and `edge-raw-image`.

### Repositories

Third-party repositories are supported by the blueprint customizations. A repository can be defined and enabled in the blueprints which will then be saved to the `/etc/yum.repos.d` directory in an image.
An optional `filename` argument can be set, otherwise the repository will be saved using the the repository ID, i.e. `/etc/yum.repos.d/<repo-id>.repo`.

Please note custom repositories **cannot be used at build time to install third-party packages**. These customizations are used to save and enable third-party repositories on the image. For more information, or if you
wish to install a package from a third-party repository, please continue reading [here](../user-guide/repository-customizations.md).

The following example can be used to create a third-party repository:

```toml
[[customizations.repositories]]
id = "example"
name="Example repo"
baseurls=[ "https://example.com/yum/download" ]
gpgcheck=true
gpgkeys = [ "https://example.com/public-key.asc" ]
enabled=true
```

Since no filename is specified, the repo will be saved to `/etc/yum.repos.d/example.repo`.

The blueprint accepts the following options:

- `id` (required)
- `name`
- `filename`
- `baseurls` (array)
- `mirrorlist`
- `metalink`
- `gpgkeys` keys (array)
- `gpgcheck`
- `repo_gpgcheck`
- `priority`
- `ssl_verify`

*Note: the `baseurls` and `gpgkeys` fields both accept arrays as input. One of `baseurls`, `metalink` & `mirrorlist` must be provided*

#### Repository GPG Keys
The blueprint accepts both inline GPG keys and GPG key urls. If an inline GPG key is provided it will be saved to the `/etc/pki/rpm-gpg` directory and will be referenced accordingly
in the repository configuration. **GPG keys are not imported to the RPM database** and will only be imported when first installing a package from the third-party repository.

### Filesystems

The blueprints can be extended to provide filesytem support. Currently the `mountpoint` and minimum partition `minsize` can be set. On `RHEL-8`, custom mountpoints are supported only since version `8.5`. For older `RHEL` versions, only the root mountpoint, `/`, is supported, the size argument being an alias for the image size.

```toml
[[customizations.filesystem]]
mountpoint = "/var"
minsize = 2147483648
```

Filesystem customizations are currently **not** supported for the following image types:

- `image-installer`
- `edge-installer` (RHEL and CentOS) and `iot-installer` (Fedora)
- `edge-simplified-installer` (RHEL and CentOS)

In addition, the following image types do not create partitioned OS images and therefore filesystem customizations for these types are meaningless:

- `edge-commit` (RHEL and CentOS) and `iot-commit` (Fedora)
- `edge-container` (RHEL and CentOS) and `iot-container` (Fedora)
- `tar`
- `container`

#### Allowed mountpoints (osbuild-composer version < 94)

In addition to the root mountpoint, `/`, the following `mountpoints` and their sub-directories are allowed:

- `/var`
- `/home`
- `/opt`
- `/srv`
- `/usr`
- `/app`
- `/data`
- `/tmp`

#### Allowed mountpoints (osbuild-composer version >= 94)

Arbitrary custom mountpoints are allowed, except for specific paths that are reserved for the OS.

The following mountpoints are **not** allowed (including their sub-directories):

- `/bin`
- `/boot/efi`
- `/dev`
- `/etc`
- `/lib`
- `/lib64`
- `/lost+found`
- `/proc`
- `/run`
- `/sbin`
- `/sys`
- `/sysroot`
- `/var/lock`
- `/var/run`

The following mountpoints are allowed, but their sub-directories are **not** allowed:

- `/usr`

### OpenSCAP

From `RHEL 8.7` & `RHEL 9.1` support has been added for `OpenSCAP` build-time remediation. The blueprints accept two fields:
- the `datastream` path to the remediation instructions (optional)
- the `profile_id` of the desired security profile

If the datastream parameter is not provided, `osbuild-composer` will now provide a sensible default based on the selected distro.

Please see [the OpenSCAP page](oscap-remediation.md) for the list of available security profiles.

```toml
[customizations.openscap]
datastream = "/usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml"
profile_id = "xccdf_org.ssgproject.content_profile_cis"
```

#### OpenSCAP Tailoring

It may be desirable to add or exclude OpenSCAP rules from the remediation scan, this can be achieved by specifying tailoring options
for the OpenSCAP customizations. A tailoring file with a new tailoring profile ID is created and saved to the image. The new tailoring
profile ID is created by appending the `_osbuild_tailoring` suffix to the base profile. For example, given tailoring options for the `cis`
profile, tailoring profile `xccdf_org.ssgproject.content_profile_cis_osbuild_tailoring` will be created. The default namespace of the rules
is `org.ssgproject.content`, so the prefix may be omitted for rules under this namespace, i.e. `org.ssgproject.content_grub2_password` and `grub2_password`
are functionally equivalent.

Note: the generated tailoring file is saved to the image as `/usr/share/xml/osbuild-oscap-tailoring/tailoring.xml`

```toml
[customizations.openscap]
datastream = "/usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml"
profile_id = "xccdf_org.ssgproject.content_profile_cis"

[customizations.openscap.tailoring]
selected = [ "xccdf_org.ssgproject.content_bind_crypto_policy" ]
unselected = [ "grub2_password" ]
```

#### FIPS

Enables/disables the system FIPS mode (disabled by default).
Currently only `edge-raw-image`, `edge-installer`, `edge-simplified-installer`,
`edge-ami` and `edge-vsphere` images support this customization.

```toml
[customizations]
fips = true
```

## Example Blueprints

### Multiple customizations

The following blueprint example will:
- install the `tmux`, `git`, and `vim-enhanced` packages
- set the hostname
- set the root ssh key
- add the groups: widget, admin users and students
- set the minimal size of root filesystem.
- set the `CIS Red Hat Enterprise Linux 8 Benchmark for Level 2` OpenSCAP
profile.
- customize the OpenSCAP profile above by enabling the `bind_crypto_policy`
and disabling the `grub2_password` content rules

```toml
name = "example-custom-base"
description = "A base system with customizations"
version = "0.0.1"

[[packages]]
name = "tmux"
version = "*"

[[packages]]
name = "git"
version = "*"

[[packages]]
name = "vim-enhanced"
version = "*"

[customizations]
hostname = "custombase"

[[customizations.sshkey]]
user = "root"
key = "A SSH KEY FOR ROOT"

[[customizations.user]]
name = "widget"
description = "Widget process user account"
home = "/srv/widget/"
shell = "/usr/bin/false"
groups = ["dialout", "users"]

[[customizations.user]]
name = "admin"
description = "Widget admin account"
password = "$6$CHO2$3rN8eviE2t50lmVyBYihTgVRHcaecmeCk31LeOUleVK/R/aeWVHVZDi26zAH.o0ywBKH9Tc0/wm7sW/q39uyd1"
home = "/srv/widget/"
shell = "/usr/bin/bash"
groups = ["widget", "users", "students"]
uid = 1200

[[customizations.user]]
name = "plain"
password = "simple plain password"

[[customizations.user]]
name = "bart"
key = "SSH KEY FOR BART"
groups = ["students"]

[[customizations.group]]
name = "widget"

[[customizations.group]]
name = "students"

[[customizations.filesystem]]
mountpoint = "/"
minsize = 2147483648

[customizations.openscap]
datastream = "/usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml"
profile_id = "xccdf_org.ssgproject.content_profile_cis"

[customizations.openscap.tailoring]
selected = [ "xccdf_org.ssgproject.content_bind_crypto_policy" ]
unselected = [ "grub2_password" ]
```

### Enable system FIPS mode in Edge Simpliflied installer

The following blueprint example will enable system FIPS mode in
a `edge-simplified-installer` image:

```toml
name = "example-system-fips-mode"
description = "A FIPS enabled base system"
version = "0.0.1"

[ostree] 

ref= "test/edge"
url= "http://example.com/repo"

[customizations]
installation_device = "/dev/vda"
fips = true

[customizations.fdo]
manufacturing_server_url = "https://fdo.example.com"
diun_pub_key_insecure = true
```
