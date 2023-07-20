# osbuild-dev

`osbuild-dev` is a manifest inspection tool and is only packaged for Fedora.

## Installation

`osbuild-dev` is shipped in the `osbuild-tools` package on Fedora starting with `osbuild` version 91. Otherwise it is available in the `/tools` directory of a [source checkout](https://github.com/osbuild/osbuild).

## Usage

### Pretty-Printing Manifest Files

`osbuild-dev manifest print [manifest]` pretty-prints a manifest file as a tree structure in your terminal.

```
€ osbuild-dev manifest print samples/fedora-build-v2.json | head -n 10
fedora-build-v2.json
├── version: 2
└── pipelines (1)
    └── 0
        ├── name: build
        ├── source-epoch: 1659397331
        └── stages (2)
            ├── 0
            │   ├── type: org.osbuild.rpm
            │   ├── inputs
```

By default it will resolve content hashes in the manifest to the names of their sources:

```
€ osbuild-dev manifest print samples/fedora-build-v2.json
fedora-build-v2.json
├── version: 2
└── pipelines (1)
    └── 0
        ├── name: build
        ├── source-epoch: 1659397331
        └── stages (2)
            ├── 0
            │   ├── type: org.osbuild.rpm
            │   ├── inputs
            │   │   └── packages
            │   │       ├── type: org.osbuild.files
            │   │       ├── origin: org.osbuild.source
            │   │       └── references
            │   │           ├── sha256:904a986d78a659d2c8fa5b810d884c0362f345979c74e0731065e22ab8988c9d: alternatives-1.19-2.fc36.x86_64.rpm
            │   │           ├── sha256:e0826095bb40863ddd1aed39a40fe8f041e1472049f322702f1fdcd024afe819: audit-libs-3.0.8-1.fc36.x86_64.rpm
            │   │           ├── sha256:980fcdf5271317b6222bf1b207e77244e145644a9e0c7bd2924c69577d3f6010: basesystem-11-13.fc36.noarch.rpm
            │   │           ├── sha256:41e48721a1bc39ded294bef9e9051c591bc43f3b8e691c188de688c892c115df: bash-5.1.16-2.fc36.x86_64.rpm
            │   │           ├── sha256:28b98634695e2f0a9f5f616b84ecb21d2726f1b360ca16ba8f48aceda4a8e0fa: bubblewrap-0.5.0-2.fc36.x86_64.rpm
            │   │           ├── sha256:dfe5b8a66e8c4593b0f5d7e6fc3bbeb605ec3cd0853fc9bc3a51afeb71d2f4e4: bzip2-libs-1.0.8-11.fc36.x86_64.rpm
            │   │           ├── sha256:9a16ff0282725909ce05ee23e8c6b1ec05917811e7a45af0933472781b1efd96: coreutils-9.0-5.fc36.x86_64.rpm
```

This behaviour can be disabled by passing `--no-resolve-sources`, `osbuild-dev manifest print --no-resolve-sources [manifest]`. `osbuild-dev manifest print` will skip the sources parts of the manifest, this can be disabled with `--no-skip-sources`.

If you want to skip specific stages, such as the `org.osbuild.rpm` stage you can do so:

```
€ osbuild-dev manifest print --ignore-stage=org.osbuild.rpm samples/fedora-build-v2.json
fedora-build-v2.json
├── version: 2
└── pipelines (1)
    └── 0
        ├── name: build
        ├── source-epoch: 1659397331
        └── stages (1)
            └── 0
                ├── type: org.osbuild.selinux
                └── options
                    ├── file_contexts: etc/selinux/targeted/contexts/files/file_contexts
                    └── labels
                        ├── /usr/bin/cp: system_u:object_r:install_exec_t:s0
                        └── /usr/bin/tar: system_u:object_r:install_exec_t:s0
```

### Diffing Manifest Files

The above pretty-printing is useful when diffing manifest files. You can do this with `osbuild manifest diff [manifest] [manifest]` which takes the same options as `osbuild manifest print [manifest]`.

By default `osbuild-dev manifest diff` will open `vimdiff`. This can be problematic when used in scripts or diffing many manifests; if you want to switch to `diff` directly you can use the `--simple` argument:

```
€ osbuild-dev manifest diff --simple --ignore-stage=org.osbuild.rpm samples/fedora-build-v2.json samples/fedora-uki.json
--- /tmp/tmph_fwiien/fedora-build-v2.json-864a  2023-07-20 12:53:12.050345775 +0200
+++ /tmp/tmph_fwiien/fedora-uki.json-2de8       2023-07-20 12:53:12.075345323 +0200
@@ -1,14 +1,113 @@
-fedora-build-v2.json
+fedora-uki.json
 ├── version: 2
-└── pipelines (1)
-    └── 0
-        ├── name: build
-        ├── source-epoch: 1659397331
+└── pipelines (5)
+    ├── 0
+    │   ├── runner: org.osbuild.fedora36
+    │   ├── name: build
+    │   ├── source-epoch: 1659397331
+    │   └── stages (1)
# omitted for brevity
```
