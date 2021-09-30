# Basic concepts

`osbuild-composer` works with a concept of **blueprints**. A blueprint is a description of the final **image** and its **customizations**. A **customization** can be:
* an additional RPM package
* enabled service
* custom kernel command line parameter, and many others. See [Blueprint](https://www.osbuild.org/guides/blueprint-reference/blueprint-reference.html#blueprint-reference) reference for more details. 

An **image** is defined by its blueprint and **image type**, which is for example `qcow2` (QEMU Copy On Write disk image) or `AMI` (Amazon Machine Image).

Finally, `osbuild-composer` also supports **upload targets**, which are cloud providers where an image can be stored after it is built. Currently supported cloud providers are [AWS](https://aws.amazon.com/) and [Azure](https://azure.microsoft.com/).

## Example blueprint

```toml
name = "base-image-with-tmux"
description = "A base system with tmux"
version = "0.0.1"

[[packages]]
name = "tmux"
version = "*"
```

The blueprint is in [TOML format](https://toml.io/en/).

## Image types

`osbuild-composer` supports various types of output images. To see all supported types, run this command:

```
$ composer-cli compose types
```
