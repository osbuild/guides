# Blueprint reference

Blueprints are simple text files in [TOML format](https://toml.io/en/) that describe which packages to install into the image, allowing to specify the packages version. They can also define a limited set of customizations to make to the final image.

A basic blueprint looks like this:

```toml
name = "base"
description = "A base system with bash"
version = "0.0.1"

[[packages]]
name = "bash"
version = "4.4.*"
```

Where:
- `name` field is the name of the blueprint. It can contain spaces, but they will be converted to `-` when it is written to disk. It should be short and descriptive.
- `description` can be a longer description of the blueprint, it is only used for display purposes.
- `version` is a semver compatible version number. If a new blueprint is uploaded with the same version the server will automatically bump the PATCH level of the version. If the version doesn't match it will be used as is, for example, uploading a blueprint with version set to 0.1.0 when the existing blueprint version is 0.0.1 will result in the new blueprint being stored as version 0.1.0.

## Packages and modules

`[[packages]]` and `[[modules]]` entries describe the package names and matching version glob to be installed into the image.

The package names must match the names exactly, and the versions can be an exact match or a filesystem-like glob of the version using `*` wildcards and `?` character matching.

*Currently there are no differences between packages and modules in `osbuild-composer`. Both are treated like an rpm package dependency.*

For example, to install `tmux-2.9a` and `openssh-server-8.*` packages, add this to your blueprint:

```toml
[[packages]]
name = "tmux"
version = "2.9a"

[[packages]]
name = "openssh-server"
version = "8.*"
```

## Groups

The `[[groups]]` entries describe a group of packages to be installed into the image. Package groups are defined in the repository metadata. Each group has a descriptive name used primarily for display in user interfaces and an ID more commonly used in kickstart files. Here, the ID is the expected way of listing a group.

Groups have three different ways of categorizing their packages: mandatory, default, and optional. For purposes of blueprints, just mandatory and default packages will be installed. There is no mechanism for selecting optional packages.

For example, if you want to install the `anaconda-tools` group, add the following to your blueprint:

```toml
[[groups]]
name="anaconda-tools"
```

groups is a TOML list, so each group needs to be listed separately, like packages but with no version number.

## Customizations

The `[customizations]` section can be used to configure the **hostname** of the final image. for example:

```toml
[customizations]
hostname = "baseimage"
```

This is optional and can be left out to use the defaults.

### Kernel command-line arguments

This allows you to append arguments to the bootloader's kernel command line.

For example:

```toml
[customizations.kernel]
append = "nosmt=force"
```

### SSH Keys

Set an existing user's ssh key in the final image:

```toml
[[customizations.sshkey]]
user = "root"
key = "PUBLIC SSH KEY"
```
The key will be added to the user's `authorized_keys` file.

*Warning: `key` expects the entire content of the public key file, traditionally `~/.ssh/id_rsa.pub` but any algorithm supported by the OS is valid*

### Additional user

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

If the password starts with $6$, $5$, or $2b$ it will be stored as an encrypted password. Otherwise it will be treated as a plain text password.

*Warning: `key` expects the entire content of `~/.ssh/id_rsa.pub`*

### Additional group

Add a group to the image. Name is required and GID is optional:

```toml
[[customizations.group]]
name = "widget"
gid = 1130
```


### Timezone

Customizing the timezone and the NTP servers to use for the system:

```toml
[customizations.timezone]
timezone = "US/Eastern"
ntpservers = ["0.north-america.pool.ntp.org", "1.north-america.pool.ntp.org"]
```

The values supported by timezone can be listed by running the command:

```
$ timedatectl list-timezones
```

If no timezone is setup, the system will default to using UTC. The NTP servers are also optional and will default to using the distribution defaults, which are suitable for most uses.

Some image types have already NTP servers setup, for example, Google cloud image, and they cannot be overridden, because they are required to boot in the selected environment. But the timezone will be updated to the one selected in the blueprint.

### Locale

Customize the locale settings for the system:

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
 $ localectl list-keymaps`
 ```


Multiple languages can be added. The first one becomes the primary, and the others are added as secondary. You must include one or more languages or keyboards in the section.

### Firewall

By default the firewall blocks all access, except for services that enable their ports explicitly, like sshd. The following command can be used to open other ports or services. Ports are configured using the `port:protocol` format:

```toml
[customizations.firewall]
ports = ["22:tcp", "80:tcp", "imap:tcp", "53:tcp", "53:udp"]
```

Numeric ports, or their names from `/etc/services` can be used in the ports enabled/disabled lists.

The blueprint settings extend any existing settings in the image templates. Thus, if sshd is already enabled, it will extend the list of ports with those already listed by the blueprint.

If the distribution uses firewalld, you can specify services listed by `firewall-cmd --get-services` in a customizations.firewall.services section:

```toml
[customizations.firewall.services]
enabled = ["ftp", "ntp", "dhcp"]
disabled = ["telnet"]
```

Remember that the firewall.services are different from the names in /etc/services.

Both are optional, if they are not used leave them out or set them to an empty list `[]`. If you only want the default firewall setup this section can be omitted from the blueprint.

*NOTE: The Google and OpenStack templates explicitly disable the firewall for their environment. This cannot be overridden by the blueprint.*

### Systemd services

This section can be used to control which services are enabled at boot time. Some image types already have services enabled or disabled in order for the image to work correctly, and cannot be overridden. For example, `ami` image type requires `sshd`, `chronyd`, and `cloud-init` services. Without them, the image will not boot. Blueprint services do not replace this services, but add them to the list of services already present in the templates, if any. 

The service names are systemd service units. You may specify any systemd unit file accepted by systemctl enable, for example, cockpit.socket:

```toml
[customizations.services]
enabled = ["sshd", "cockpit.socket", "httpd"]
disabled = ["postfix", "telnetd"]
```
## Distribution selection with blueprints

The blueprint now supports a new `distro` field that will be used to select the
distribution to use when composing images, or depsolving the blueprint. If
`distro` is left blank it will use the host distribution. If you upgrade the
host operating system the blueprints with no `distro` set will build using the
new os.

eg. A blueprint that will always build a fedora-32 image, no matter what
version is running on the host:

```toml
name = "tmux"
description = "tmux image with openssh"
version = "1.2.16"
distro = "fedora-32"

[[packages]]
name = "tmux"
version = "*"

[[packages]]
name = "openssh-server"
version = "*"
```

### Filesystem Support

The blueprints can be extended to provide filesytem support. Currently the `mountpoint` and minimum partition `size` can be set. Custom mountpoints are currently only supported for `RHEL 8.5` & `RHEL 9.0`. For other distributions, only the `root` partition is supported, the size argument being an alias for the image size. 

```toml
[[customizations.filesystem]]
mountpoint = "/var"
size = 2147483648
```
In addition to the root mountpoint, `/`, the following `mountpoints` and their sub-directories are supported:

- `/var`
- `/home`
- `/opt`
- `/srv`
- `/usr`
- `/app`
- `/data`

## Example Blueprint

The following blueprint example will:
- install the `tmux`, `git`, and `vim-enhanced` packages
- set the root ssh key
- add the groups: widget, admin users and students

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
size = 2147483648
```
