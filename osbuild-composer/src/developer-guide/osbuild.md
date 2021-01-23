# osbuild

A CLI tool for building OS images. It takes manifest as an input and produces an image as an output. The manifest consists of:

- sources section
- pipeline

In our usual use-case, that is tied to Fedora and RHEL, not applicable to other non-RPM distros, the sources section contains an `org.osbuild.files` section, which is a list of RPMs described by their name, hash, and URL for downloading. We do not support metalink at the moment.

*This section is, very often, a source of build failures. This happens because we can only include a single link and RPM repos are often instable. Furthermore, we need to set a timeout for the `curl` download, because we want the build to timeout eventually in case the RPMs are unavailable, but it sometimes fails on slow Internet connection as well.*

The pipeline consists of a series of stages and ends with an assembler. A stage is our unit of filesystem tree modification and it is implemented as a standalone executable. For example, we have a stage for installing RPM packages, adding a user, enabling systemd service, or setting a timezone.

The difference between a stage and an assembler is that the former takes a read-write filesystem-tree and performs a certain modification to it, whereas the latter takes a read-only filesystem tree and produces an output artifact.

The pipeline contains one more "nested" pipeline, which does not have an assembler. It is called a "build" pipeline.

## High level goals

1. reproducibility
2. extensibility

The ideal case for building images would be that, given the same input manifest, the output image would always be the same no matter what machine was used for building it. Where "the same" is defined as a binary equivalent. The world of IT is, of course, not ideal therefore we define **reproducibility** as a functional equivalence (that is the image behaves the same when built on different machines) and we limit the set of build machines only to those running the same distribution, in the same version, and on the same architecture. That means if you want to build a Fedora 33 aarch64 image, you need a Fedora 33 aarch64 machine.

*It is possible to run a RHEL pipeline on Fedora, for example, but we do not test it and therefore we can't promise it will produce the correct result.*

The advantage of the stage/assembler model is that any user can **extend** the tool with their own stage or assembler.

## How it all works together

Explain downloading sources, creating build fs tree, container, caching, the problem with reflink, bwrap etc.

## Bootstrapping the build environment

The "build" pipeline was introduced to improve reproducibility. Ideally, given a build pipeline, one would always get the same filesystem tree. But, to create the first filesystem tree, you need some tools. So, where go you get them from? Of course from the host operating system (OS). The problem with getting tools from the host OS this is that the host can affect the final result.

*We've already had this issue many times, because most of the usual CLI tools were not created with reproducibility in mind.*

## The struggle with GRUB

The standard tooling for creating GRUB does not fit to our stage/assembler concept because it wants to modify the filesystem tree and create the resulting artifact at the same time. As a result we have our own reimplementation of these tools.
