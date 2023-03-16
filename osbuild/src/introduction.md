# Introduction

`osbuild` is a declarative pipeline-based build system for [operating system artifacts](./terminology.md). It defines a universal [pipeline description](./user-guide/pipelines.md) and a build system to execute them, producing artifacts like operating system images, working towards an image build pipeline that is more comprehensible, reproducible, and extendable.

`osbuild` is the low level 'assembler' for the `osbuild-composer` project which offers a more highlevel form of defining and building operating system images and artifacts. We generally recommend people to take a look to see if `osbuild-composer` fills their needs first, before diving into `osbuild`. You can find `osbuild-composer` on [GitHub](https://github.com/osbuild/osbuild-composer).
