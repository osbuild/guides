# Building a RHEL for Edge Installer

The following describes how to build a boot ISO which installs an OSTree-based system using the "RHEL for Edge Container" in combination with the "RHEL for Edge Installer" image types. The workflow has the same result as the [Building OSTree Image](./building-ostree-images.md) guide with the new image types automating some of the steps.

Note that there are some small differences in this procedure between RHEL 8.4 and RHEL 8.5:
- The names of the image types have changed. In 8.4, the image types were prefixed by `rhel-`. This prefix was removed in 8.5.
    - The old names `rhel-edge-container` and `rhel-edge-installer` still work in RHEL 8.5 as aliases to the new names, however these names are considered deprecated and may be removed completely in future versions.
- The internal port for the container has changed from 80 in RHEL 8.4 to 8080 in RHEL 8.5.

## Process overview

1. Create and load a blueprint with customizations.
2. Build an `edge-container` (RHEL 8.5) or `rhel-edge-container` (RHEL 8.4) image.
3. Load image in podman and start the container.
4. Create and load an empty blueprint.
5. Build an `edge-installer` (RHEL 8.5) or `rhel-edge-installer` (RHEL 8.4) image, pointing the `ostree-url` to `http://10.0.2.2:8080/repo/` and setting the `ostree-ref` to `rhel/edge/demo`.

The `edge-container` image type creates an OSTree commit and embeds it into an OCI container with a web server. When the container is started, the web server serves the commit as an OSTree repository.

The `edge-intaller` image type pulls the commit from the running container and creates an installable boot ISO with a kickstart file configured to use the embedded OSTree commit.

## Detailed workflow

### Build the container and serve the commit

Start by creating a blueprint for the commit. The content below is an example and can be modified to fit your needs. For this guide, we will name the file `example.toml`.

```toml
name = "example"
description = "RHEL for Edge Installer example"
version = "0.0.3"

[[packages]]
name = "vim-enhanced"
version = "*"

[[packages]]
name = "tmux"
version = "*"

[customizations]

[[customizations.user]]
name = "user"
description = "Example User"
password = "$6$uvdfeuHQYM6kUaea$fvvzyu.Z.u89TVCB2tq8UEc52XDFGnAqCo75BX3zu8OzIbS.EKMo/Saammb151sLrdzmlESnpNEPrJ7h5b0c6/"
groups = ["wheel"]
```

Now push the blueprint to osbuild-composer using `composer-cli`:
```
$ composer-cli blueprints push example.toml
```

And start the container build:
```
$ composer-cli compose start-ostree --ref "rhel/edge/example" example edge-container
Compose 8e8014f8-4d15-441a-a26d-9ed7fc89e23a added to the queue
```
The value for `--ref` can be changed but must begin with an alphanumeric character and contain only alphanumeric characters, `/`, `_`, `-`, and `.`.

*Note: In RHEL 8.4, the image type was called `rhel-edge-container`. It has been renamed to `edge-container` in 8.5 onwards.*

Monitor the build status using:
```
$ composer-cli compose status
```

When the compose is FINISHED, download the result:
```
$ composer-cli compose image 8e8014f8-4d15-441a-a26d-9ed7fc89e23a
8e8014f8-4d15-441a-a26d-9ed7fc89e23a-rhel84-container.tar: 670.45 MB
```

Load the container into registry:
```
$ cat 8e8014f8-4d15-441a-a26d-9ed7fc89e23a-rhel84-container.tar | podman load
Getting image source signatures
Copying blob 82934cd3e69d done
Copying config d11911c3dc done
Writing manifest to image destination
Storing signatures
Loaded image(s): @d11911c3dc4bee46cabd52b91c87f48b8a7d450fadc8cfbeb69e2de98b413521
```

Tag the image for convenience:
```
$ podman tag d11911c3dc4bee46cabd52b91c87f48b8a7d450fadc8cfbeb69e2de98b413521 localhost/edge-example
```

Start the container (note the different internal port numbers between the two versions)

For RHEL 8.4:
```
$ podman run --rm -d -p 8080:80 --name ostree-repo localhost/edge-example
```

For RHEL 8.5+:
```
$ podman run --rm -d -p 8080:8080 --name ostree-repo localhost/edge-example
```

*Note: The `-d` option detaches the container and leaves it running in the background. You can also remove the option to keep the container attached to the terminal.*

### Build the installer

Start by creating a simple blueprint for the installer. The blueprint must not have any customizations or packages; only a name, and optionally a version and a description. Add the content below to a file and name it `empty.toml`:
```
name = "empty"
description = "Empty blueprint"
version = "0.0.1"
```
The `edge-installer` image type does not support customizations or package selection, so the build will fail if any are specified.

Push the blueprint:
```
$ composer-cli blueprints push empty.toml
```

Start the build:
```
$ composer-cli compose start-ostree --ref "rhel/edge/example" --url http://10.0.2.2:8080/repo/ empty edge-installer
Compose 09d98a67-a401-4613-9a5b-b93f8a6e695f added to the queue
```
*Note: In RHEL 8.4, the image type was called `rhel-edge-installer`. It has been renamed to `edge-installer` in 8.5 onwards.*

The `--ref` argument must match the one from the `rhel-edge-container` compose.
The `--url` in this case is IP address of the container. This tutorial uses `qemu` to boot the virtual machine and `10.0.2.2` is an address which you can use to reach the host system from the guest: [User Networking](https://wiki.qemu.org/Documentation/Networking#User_Networking_.28SLIRP.29).

Monitor the build status using:
```
$ composer-cli compose status
```

When the compose is FINISHED, download the result:
```
$ composer-cli compose image 09d98a67-a401-4613-9a5b-b93f8a6e695f
 09d98a67-a401-4613-9a5b-b93f8a6e695f-rhel84-boot.iso: 1422.61 MB
```

The downloaded image can then booted to begin the installation. If you used the blueprint in this guide, use the username "user" and password "password42" to login.
