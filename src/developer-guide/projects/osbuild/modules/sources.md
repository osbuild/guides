# Sources

[Manifests](../manifests/index.md) contain a sources section which tells `osbuild` to get prerequisites for [pipelines](./pipelines.md). These sources are written down in such a way that the sources are unable to change. This is an important bit for having reproducible manifests.

Sources are stored in the `sources` property of the root manifest object. See the [manifest](../manifests/index.md) description for a full example.

Sources are refered to by their content hash, this can be cumbersome to do by hand. [osbuild-mpp](../executables/osbuild-mpp.md) offers depsolving and other utilities to expand these content hashes for you.

## Available Sources

`osbuild` comes with the following upstream sources.

### org.osbuild.curl

The `org.osbuild.curl` source allows files served over HTTP(s) to be part of the manifest.

Read [source code](https://github.com/osbuild/osbuild/blob/main/sources/org.osbuild.curl).

#### Example JSON

```json
{
  "sources": {
    "org.osbuild.curl": {
      "items": {
        "sha256:...": "...",
        "sha256:...": "..."
    }
  }
}
```

### org.osbuild.inline

The `org.osbuild.inline` source allows files to be inlined in the manifest.

Read [source code](https://github.com/osbuild/osbuild/blob/main/sources/org.osbuild.inline).

#### Example JSON

### org.osbuild.ostree

The `org.osbuild.ostree` source allows manifests to use ostree references as a source.

Read [source code](https://github.com/osbuild/osbuild/blob/main/sources/org.osbuild.ostree).

### org.osbuild.skopeo

The `org.osbuild.ostree` source allows manifests to use container layers as a source.

Read [source code](https://github.com/osbuild/osbuild/blob/main/sources/org.osbuild.skopeo).

#### Example JSON

