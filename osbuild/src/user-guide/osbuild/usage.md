# Usage

After [installing](./installation.md) `osbuild` you're ready to build your first images. The `osbuild` [repository](https://github.com/osbuild/osbuild) comes with a bunch of example [manifest](./concepts.md#manifest) files to build various images.

## First Build

Let's take one of the example files: [fedora-container.json](https://raw.githubusercontent.com/osbuild/osbuild/main/test/data/manifests/fedora-container.json) and build it.

```
$ curl -s -o fedora-container.json https://raw.githubusercontent.com/osbuild/osbuild/main/test/data/manifests/fedora-container.json
$ osbuild fedora-container.json
build:      9590527d97c7cffe87595ff5a239b605258610e2cad0df8a7979e4f9c83d7e5b
tree:       1a705fc081acab5fc3400bd1cf9bdccb94a1ad249301c5bf9770fb4081df8d10
container:  cd10fb1a74eac14f75618a2aa7e67edc2c33447f267a261749f75322dc4ffee3
$ sudo osbuild --export container --output-directory=. fedora-container.json
...
$ file container/fedora-container.tar
container/fedora-container.tar: POSIX tar archive
```

There's a lot to unpack here so let's take a closer look. We start by downloading a manifest file. These JSON files describe how exactly the image should be assembled. If you take a look inside of them you might see they're quite explicit and verbose about what they need and where to get it.

That's because the manifest files are usually generated at a higher level by (for example), `osbuild-composer` and so what you are seeing in the manifest is the fully generated and written out repeatable build description.

When building the manifest with `osbuild` you get some output related to *build*, *tree*, and *container*. What do those names and identifiers in there actually mean?

The names and identifiers relate to the manifest, they are the [pipelines](./concepts.md#pipelines) contained in them. Each pipeline can be exported separately by passing either its name or its identifier. The identifiers are generated based on the content of the pipeline.

## Arguments

* `--store` supply a directory where intermediate outputs can be stored.
* `--checkpoint`
* `--export` export a pipeline by name or identifier to the `--output-directory`.
* `--json` format all output as JSON.
* `--output-directory` the directory used for the `--export` command and where outputs will be written to.
* `--inspect`
* `--monitor`
* `--monitor-fd`
* `--stage-timeout`
