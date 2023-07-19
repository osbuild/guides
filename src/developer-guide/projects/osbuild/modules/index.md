# Modules

The pluggable system of `osbuild`. Everything defined in a pipeline is generally refered to as a `module`. Modules can generally be divided in the following groups:

- [sources](./sources.md)
- [stages](./stages.md)
- [inputs](./stages.md)
- [mounts](./mounts.md)
- [devices](./devices.md)

Each module is an importable Python script, `osbuild` will try to import the referenced module and get its metadata such as a schema that defines the valid options for a module. It is important that modules present a stable API as `osbuild` has no versioning of modules. After a module has been included in `osbuild` one should **never**: add required fields, change types of fields, or any other action that breaks compatibility.

Being able to build previous manifests always is important to `osbuild`.
