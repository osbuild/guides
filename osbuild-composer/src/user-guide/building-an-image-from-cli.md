# Building an image

An image is specified by a blueprint and an image type. It will use the same distribution version (e.g. Fedora 33) and architecture (e.g. aarch64) as the host system.

To build a customized image, start by choosing the blueprint and image type you would like to build. To do so, run the following commands:

```
$ sudo composer-cli blueprints list
$ sudo composer-cli compose types
```

and trigger a compose (example using the blueprint from the previous section):

```
$ composer-cli compose start base-image-with-tmux qcow2
Compose ab71b61a-b3c4-434f-b214-1e16527766ff added to the queue
```

Note that the compose is assigned with a Universally Unique Identifier (UUID), that you can use to monitor the image build progress:

```
$ composer-cli compose info ab71b61a-b3c4-434f-b214-1e16527766ff
ab71b61a-b3c4-434f-b214-1e16527766ff RUNNING  base-image-with-tmux 0.0.1 qcow2            2147483648
Packages:
    tmux-*
Modules:
Dependencies:
```

At this time, the compose is in a "RUNNING" state. Once the compose reaches the "FINISHED" state, you can download the resulting image by running the following command:

```
$ sudo composer-cli compose results ab71b61a-b3c4-434f-b214-1e16527766ff
ab71b61a-b3c4-434f-b214-1e16527766ff.tar: 455.18 MB
$ fd
ab71b61a-b3c4-434f-b214-1e16527766ff.tar
$ tar xf ab71b61a-b3c4-434f-b214-1e16527766ff.tar
$ fd 
ab71b61a-b3c4-434f-b214-1e16527766ff-disk.qcow2
ab71b61a-b3c4-434f-b214-1e16527766ff.json
ab71b61a-b3c4-434f-b214-1e16527766ff.tar
logs
logs/osbuild.log
```

From the example output above, the resulting tarball contains not only the `qcow2` image, but also a `JSON` file, which is the osbuild manifest (see the [Developer guide](https://www.osbuild.org/guides/developer-guide/developer-guide.html#developer-guide) for more details), and a directory with logs.

For more options, see the `help` text for `composer-cli`:

```
$ sudo composer-cli compose help
```

#### Tip: Booting the image with qemu

If you want to quickly run the resulting image, you can use `qemu`:

```
$ qemu-system-x86_64 \
                -enable-kvm \
                -m 3000 \
                -snapshot \
                -cpu host \
                -net nic,model=virtio \
                -net user,hostfwd=tcp::2223-:22 \
                ab71b61a-b3c4-434f-b214-1e16527766ff-disk.qcow2 
```

Be aware that you must specify a way to access the machine in the blueprint. For example, you can create a user with known password, set an SSH key, or enable `cloud-init` to use a `cloud-init` ISO file.
