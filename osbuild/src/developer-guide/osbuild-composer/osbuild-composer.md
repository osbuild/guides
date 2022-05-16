# osbuild-composer

It is a web service for building OS images. The core of `osbuild-composer`, which is common to all APIs, is osbuild manifests generation a job queuing. If an operating system is to be supported by `osbuild-composer`, it needs the manifest generation code in `internal/distro` directory. So far, we only focus on RPM based distributions, such as Fedora and RHEL. The queuing mechanism is under heavy development at the moment.



## Interfacing with dnf package manager

We use our custom wrapper for `dnf`, which we call simply `dnf-json`, because its interface goes like this:

* Stdin - takes  a JSON object
* Stdout - returns a JSON object
* Return code is used **only** for `dnf-json` internal errors, not for errors in the operation specified on the input. Those errors are reported in the returned JSON object.

## Local API - Weldr

This API comes from the `Lorax-composer project`. `osbuild-composer` was created as a drop-in replacement for Lorax which influenced many design decisions. It uses Unix-Domain socket, so it is meant for local usage only. There are two clients:

* composer-cli
* cockpit-composer (branded as Image Builder in the Cockpit console)

Activate this API by invoking `systemctl start osbuild-composer.socket`. Systemd will create a socket at `/run/weldr/api.socket`.

## Remote APIs - Cloud and Koji

Both are under heavy development.

