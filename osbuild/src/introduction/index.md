# Introduction

Welcome to the `osbuild` project guides. These project guides are meant to help
users and developers of the `osbuild` projects to find their bearings.

## What is `osbuild`?

`osbuild` is a project to create operating system images of Linux operating
systems in a reliable fashion, isolating image creation from your host
operating system and producing a reliable and well-defined image ready to be
deployed where it is needed.

### A project, program, or service?

All of the above! `osbuild` itself is a project that contains separate programs to fulfill
all tasks related to building operating system images. The separate programs
and their tasks are described in depth in the [Ecosystem](/ecosystem/index.md)
chapter of the guides.

As a reference or quick summary:

* [osbuild](/user-guide/osbuild/index.md): the lower level 'assembler'.
* [osbuild-composer](/user-guide/osbuild-composer/index.md): glue between `osbuild` and the various interfaces.
* [weldr-client](/user-guide/weldr-client/index.md): command line interface.
* [cockpit-composer](/user-guide/cockpit-composer/index.md): self hosted web interface.
* [image-builder](/user-guide/image-builder/index.md): RedHat provided web interface and APIs.

## Why `osbuild`?

If you're building operating systems `osbuild` is for you. Building images
historically involved a bunch of complicated shell scripts.

With `osbuild` you can write a small TOML file to define your image and all
its customizations, you can then build your image in a variety of formats to
suit your own infrastructure needs or directly upload to the cloud of your
choosing to deploy new machines.


## Who is working on `osbuild`?

`osbuild` is an open source project and developed on [GitHub](https://github.com/osbuild),
the project is sponsored by [RedHat](https://redhat.com/) so a lot of the
contributors to the project work at RedHat.

## Where is `osbuild` being used?

The adoption of `osbuild` is growing. Currently `osbuild` is being used to
build the [Fedora IoT edition](https://getfedora.org/en/iot/) and various editions
of RHEL.

It is also
available to users of [RedHat Insights](https://console.redhat.com) through
the [image-builder](/user-guide/image-builder/index.md) web service where
users can build their own customized images.
