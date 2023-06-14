# Installation

To get started with `osbuild-composer` on your local machine, you can install the CLI interface or the Web UI, which is part of Cockpit project. 

## CLI interface

For CLI only, run the following command to install necessary packages:

```
$ sudo dnf install osbuild-composer composer-cli
```

To enable the service, run this command:

```
$ sudo systemctl enable --now osbuild-composer.socket
```

Verify that the installation works by running `composer-cli`:

```
$ sudo composer-cli status show
```

If you prefer to run this command without sudo privileges, add your user to the `weldr` group:

```
$ sudo usermod -a -G weldr <user>
$ newgrp weldr
```

## Web UI

If you prefer the Web UI interface, known as an Image Builder, install the following package:

```
$ sudo dnf install cockpit-composer
```

and enable `cockpit` and `osbuild-composer` services:

```
$ sudo systemctl enable --now osbuild-composer.socket
$ sudo systemctl enable --now cockpit.socket
```

