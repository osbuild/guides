# Terminology

## Module

The parts that make up `osbuild`.

## Operating System Artifact

A thing that is an operating system image. A tarball, an ISO, a container, or an ostree commit.

## Source

A file that is needed by [stage](./user-guide/stages.md) and should be retrieved from outside the chroot. Sources are retrieved by [source modules](./user-guide/stages.md) and are addressed by their content hash.
