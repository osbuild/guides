# Installation

To get started with osbuild-composer locally, you can install the CLI interface or the Web UI, which is part of Cockpit project. 

## CLI interface

For CLI only, invoke this command to install necessary packages:

```
$ sudo dnf install osbuild-composer composer-cli
```

And this command to enable the service:

```
$ sudo systemctl enable --now osbuild-composer.socket
```

Verify that the installation works by invoking `composer-cli`:

```
$ sudo composer-cli status show
```

If you prefer to invoke this command without sudo, add your user to the `weldr` group:

```
$ sudo usermod -a -G weldr <user>
$ newgrp weldr
```

## Web UI

If you prefer the Web UI, known as an Image Builder, install this package:

```
$ sudo dnf install cockpit-composer
```

and enable cockpit and osbuild-composer services:

```
$ sudo systemctl enable --now osbuild-composer.socket
$ sudo systemctl enable --now cockpit
```

