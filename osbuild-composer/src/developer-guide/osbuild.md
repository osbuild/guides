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

## How osbuild works in practise

The following subsections describe how OSBuild tries to achieve the outlined high level goals.

### Manifest versions

OSBuild accepts two versions of manifests. Both manifests are plain JSON files. The following sections contain examples of both *(note that comments are not allowed in JSON, so the examples below are not actually valid JSON)*.

#### Version 1

The version 1 manifest is built around the idea that an artifact is produced by downloading files from the Internet (e.g. RPMs), using them to build and modify a filesystem tree (using stages), and finally using a read-only version of the final filesystem tree as an input to a assembler which produces the desired artifact.

```yaml
{
   # This version contains 2 top-level keys.
   # First sources, these get downloaded from a network and are available
   # in the stages.
   "sources": {},
   # Second is a pipeline, which can optionally contain a nested "build"
   # pipeline.
   "pipeline": {
      # The build pipeline is used to create a build container that is
      # later used for building the actual OS artifact. This is mostly
      # to increase reproducibility and host-guest separation.
      # Also note that this is optional.
      "build": {
         "pipeline": {
            "stages": [
               {
                  "name": "",
                  "options": {}
               },
               {
                  "name": "",
                  "options": {}
               }
            ],
            "runner": ""
         }
      },
      # The pipeline itself is a list of osbuild stages.
      "stages": [
         {
            "name": "",
            "options": {}
         },
         {
            "name": "",
            "options": {}
         }
      ],
      # And finally exactly one osbuild assembler.
      "assembler": {
         "name": "",
         "options": {}
      }
   },
}
```

#### Version 2

Version 2 is more complicated because OSBuild needed to cover additional use cases like OSTree commit inside of a OCI container. In general that is an artifact inside of another artifact. This is why it comes with multiple pipelines.

```yaml
{
   # This version has 3 top-level keys.
   # The first one is simply a version.
   "version": "2",
   # The second one are sources as in version 1, but keep in mind that in this
   # version, stages take inputs instead of sources because inputs can be both
   # downloaded from a network and produced by a pipeline in this manifest.
   "sources": {},
   # This time the 3rd entry is a list of pipelines.
   "pipelines": [
      {
         # A custom name for each pipeline. "build" is used only as an example.
         "name": "build",
         # The runner is again optional.
         "runner": "",
         "stages": [
            {
               # The "type" is same as "name" in v1.
               "type": "",
               # The "inputs" field is new in v2. You can specify what goes to
               # the stage. Example inputs are RPMs and OSTree commits from the
               # "sources" section, but also filesystem trees built by othe
               # pipelines.
               "inputs": {},
               "options": {}
            }
         ]
      },
      {
         # Again only example name.
         "name": "build-fs-tree",
         # But this time the pipeline can use the previous one as a build pipeline.
         # The name:<something> is a reference format in OSBuild manifest v2.
         "build": "name:build",
         "stages": []
      },
      {
         "name": "do-sth-with-the-tree",
         "build": "name:build",
         "stages": [
            {
               "type": "",
               "inputs": {
                  # This is an example of how to use the filesystem tree built by
                  # another pipeline as an input to this stage.
                  "tree": {
                     "type": "org.osbuild.tree",
                     "origin": "org.osbuild.pipeline",
                     "references": [
                        # This is a reference to the name of the pipeline above.
                        "name:build-fs-tree"
                     ]
                  }
               },
               "options": {}
            }
         ]
      },
      {
         # In v2 the assembler is a pipeline as well.
         "name": "assembler",
         "build": "name:build",
         "stages": []
      }
   ]
}
```

### Components of osbuild

OSBuild is designed as a set of loosely coupled or independent components. This subsection describes each of them separately so that the following section can describe how they work together.

#### Object Store

Object store is a directory (also a class representing it) that contains multiple filesystem trees. Each filesystem tree lives in a directory whose name represents hash of the pipeline resulting in this tree. In OSBuild, a user can specify a "checkpoint" which stores particular filesystem tree inside of the Object Store.

#### Build Root

It is a directory where OSBuild modules (stages and assemblers) are executed. The directory contains full operating system which is composed of multiple things:
 * Executables and libraries needed for building the OS artifact (these are either from the host or created in a build pipeline).
 * Directory where the resulting filesystem tree resides.
 * Few directories bind-mounted directly from the host system (like `/dev`)
 * API sockets for communication between the stage running inside a container and the osbuild process running outside of it (directly on the host).

#### Sources

Sources are artifacts that are downloaded from the Internet. For example, generic files downloaded with `curl`, or OSTree commits downloaded using `libostree`.

#### Inputs

Inputs are a generalization of the concept of sources, but this time an "input" can be both downloaded, as sources are, or generated using osbuild pipeline. That means one pipeline can be used as an input for another pipeline so you can have an artifact inside of an artifact (for example OSTree commit inside of a container).

#### APIs

OSBuild allows for bidirectional communication from the build container to the osbuild process running on the host system. It uses Unix-domain sockets and JSON-based communication (`jsoncomm`) for this purpose. Examples of available APIs:
 * osbuild - provides basic osbuild features like passing arguments to the stage inside the build container or reporting exceptions from the stage back to the host
 * remoteloop - helps with setting up loop devices on the host and forwarding them to the container
 * sources - runs a source module and returns the result

### What happens during simplified osbuild run

This section puts the above concepts into context. It does not aim to describe all the possible code paths. To understand `osbuild` properly, you need to read the source code, but it should help you get started.

During a single `osbuild` run, this is what usually happens:
 1. Preparation
    1. Validate the manifest schema to make sure it is either v1 or v2 manifest
    2. Object Store is instantiated either from an empty directory or from already existing one which might contain already cached filesystem trees.
 2. Processing the manifest
    1. Download sources
    2. Run all pipelines sequentially
 3. Processing a pipeline (one of N)
    1. Check the Object Store for cached filesystem trees and start from there if it already contains parially built artifact
 4. Processing a module (stage or assembler)
    1. Create a BuildRoot, which means initializing a `bwrap` container, mounting all necessary directories, and forwarding API sockets.
    2. From the build container, use the osbuild API to get arguments and run the module
 5. If an assembler is present in the manifest, run it and store the resulting artifact in the output directory

## Issues that do not fit into the high level goals

### Bootstrapping the build environment

The "build" pipeline was introduced to improve reproducibility. Ideally, given a build pipeline, one would always get the same filesystem tree. But, to create the first filesystem tree, you need some tools. So, where go you get them from? Of course from the host operating system (OS). The problem with getting tools from the host OS this is that the host can affect the final result.

*We've already had this issue many times, because most of the usual CLI tools were not created with reproducibility in mind.*

### The struggle with GRUB

The standard tooling for creating GRUB does not fit to our stage/assembler concept because it wants to modify the filesystem tree and create the resulting artifact at the same time. As a result we have our own reimplementation of these tools.

## Running OSBuild from sources

It is not strictly required to run OSBuild installed from an RPM package but if you attempt to run `osbuild` from the command line like this:
```
$ python3 -m osbuild
```
and, at the same time, you will include SELinux stage in the manifest, it will most likely fail because the `python3` executable and all stages and assemblers in the checkout are not labeled properly. To overcome this issue, create two additional files.

 1. New entrypoint which will soon have the right SELinux label, let's call it `osbuild-cli`:
 ```python
 #!/usr/bin/python3

import sys

from .osbuild.main_cli import osbuild_cli as main


if __name__ == "__main__":
    r = main()
    sys.exit(r)
 ```

 2. A script to relabel all the files that need it:
 ```bash
 #!/bin/bash

LABEL=$(matchpathcon -n /usr/bin/osbuild)

echo "osbuild label: ${LABEL}"

chcon ${LABEL} osbuild-cli

find . -maxdepth 2 -type f -executable -name 'org.osbuild.*' -print0 |
    while IFS= read -r -d '' module; do
	chcon ${LABEL} ${module}
    done
 ```

Now run the script and use the entrypoint to execute OSBuild from git checkout.