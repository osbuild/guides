# Building an image

An image is specified with a blueprint and an image type. It will use the same distribution version (e.g. Fedora 33) and architecture (e.g. aarch64) as the host system.

Start by choosing the blueprint and image type you would like to build:

```
$ sudo composer-cli blueprints list
$ sudo composer-cli compose types
```

and trigger a compose (example using the blueprint from the previous section):

```
$ composer-cli compose start base-image-with-tmux qcow2
Compose ab71b61a-b3c4-434f-b214-1e16527766ff added to the queue
```

As you can see, the compose has a UUID assigned which you can use to monitor the progress:

```
$ composer-cli compose info ab71b61a-b3c4-434f-b214-1e16527766ff
ab71b61a-b3c4-434f-b214-1e16527766ff RUNNING  base-image-with-tmux 0.0.1 qcow2            2147483648
Packages:
    tmux-*
Modules:
Dependencies:
```

And as you can see, the compose is in a "RUNNING" state. Once the compose reaches the "FINISHED" state, you can download the result by invoking:

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

As you can see in the example output above, the resulting tarball contains not only the qcow2 image, but also a JSON file, which is the osbuild manifest (see the developer guide for explanation), and a directory with logs.

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

Keep in mind that you need to specify a way to access the machine in the blueprint. For example by creating a user with known password, setting an SSH key, or enabling cloud-init and using cloud-init ISO file.
