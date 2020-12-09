# Blueprint reference

Blueprints are simple text files in [TOML format](https://toml.io/en/) that describe which packages, and what versions, to install into the image. They can also define a limited set of customizations to make to the final image.

A basic blueprint looks like this:

```toml
name = "base"
description = "A base system with bash"
version = "0.0.1"

[[packages]]
name = "bash"
version = "4.4.*"
```

The name field is the name of the blueprint. It can contain spaces, but they will be converted to - when it is written to disk. It should be short and descriptive.

description can be a longer description of the blueprint, it is only used for display purposes.

version is a semver compatible version number. If a new blueprint is uploaded with the same version the server will automatically bump the PATCH level of the version. If the version doesn't match it will be used as is. eg. Uploading a blueprint with version set to 0.1.0 when the existing blueprint version is 0.0.1 will result in the new blueprint being stored as version 0.1.0.

## Packages and modules

`[[packages]]` and `[[modules]]` entries describe the package names and matching version glob to be installed into the image.

The names must match the names exactly, and the versions can be an exact match or a filesystem-like glob of the version using `*` wildcards and `?` character matching.

*Currently there are no differences between packages and modules in osbuild-composer. Both are treated like an rpm package dependency.*

For example, to install `tmux-2.9a` and `openssh-server-8.*`, you would add this to your blueprint:

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

Groups have three different ways of categorizing their packages: mandatory, default, and optional. For purposes of blueprints, mandatory and default packages will be installed. There is no mechanism for selecting optional packages.

For example, if you want to install the anaconda-tools group you would add this to your blueprint:

```toml
[[groups]]
name="anaconda-tools"
```

groups is a TOML list, so each group needs to be listed separately, like packages but with no version number.

## Customizations

The `[customizations]` section can be used to configure the hostname of the final image. eg.:

```toml
[customizations]
hostname = "baseimage"
```

This is optional and may be left out to use the defaults.

### kernel

This allows you to append arguments to the bootloader's kernel commandline.

For example:
```toml
[customizations.kernel]
append = "nosmt=force"
```
