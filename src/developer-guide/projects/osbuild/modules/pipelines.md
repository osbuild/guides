# Pipelines

Each pipeline is a collection of [stages](./stages.md) that together form a logical grouping. A commonly seen pipeline is for the one that builds the operating system root. Each pipeline can be exported on their own and depend on other pipelines.

Pipelines are stored in the `pipelines` property of the root manifest object. See the [manifest](../manifests/index.md) description for a full example.

## Properties

| property  | value |
|-----------|-------|
| `name` | Pipeline name. |
| `stages` | A list of [stages](./stages.md) to be ran in succession. |
| `build` | Optional: the `name:name` or `sha256:hash` of the pipeline to use as a buildroot for this pipeline. |
| `runner` | Optional: the runner to use as a buildroot for this pipeline. |
| `source-epoch` | Optional: set all files in the resulting tree to this UNIX timestamp. |



## Example JSON

This example shows the general layout of the pipelines in a manifest where we have a `build` pipeline which puts together the buildroot, the `tree` pipeline which puts together the file tree and lastly an `image` pipeline that transforms the tree pipeline result into an image for consumption.

```json
{
  "pipelines": [
    {
      "name": "build",
      "runner": "org.osbuild.fedora37",
      "stages": [...]
    },
    {
      "name": "tree",
      "build": "name:build",
      "stages": [...]
    },
    {
      "name": "image",
      "build": "name:build",
      "stages": [...]
    }
  ]
}
```

