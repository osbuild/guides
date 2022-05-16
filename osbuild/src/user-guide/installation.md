# Installation

To get started with `osbuild-composer` on your local machine, you can install the `weldr-client` CLI interface or the `cockpit-composer` Web UI, which is a plugin for the [Cockpit project](https://cockpit-project.org/). 

We start out with installing the `osbuild-composer` service:

```
$ sudo dnf install osbuild-composer
```

To enable the service, run this command:

```
$ sudo systemctl enable --now osbuild-composer.socket
```

## CLI interface

To add the CLI, run the following command to install necessary packages:

```
$ sudo dnf install composer-cli
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

If you prefer a Web UI interface, known as an Image Builder, install the following package:

```
$ sudo dnf install cockpit-composer
```

and enable the `cockpit` service:

```
$ sudo systemctl enable --now cockpit.socket
```

Your Web UI will then be available on `https://localhost:8000`.
