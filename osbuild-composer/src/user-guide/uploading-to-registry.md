# Uploading a container image to a registry

`osbuild-composer` can upload a container image, like the RHEL for
edge container, to a registry directly after it has been built.

In order to do so, the container reference and an upload configuration
file need to be specified when building a container artifact:

```
$ sudo composer-cli compose start BLUEPRINT container REFERENCE CONFIG.toml
```

where `BLUEPRINT` is the name for the container and `REFERENCE` the
reference to the container image, like `registry.example.com/image:tag`.
If `:tag` is omitted, `:latest` is the default.  The `CONFIG.toml` file
must include `provider = "container"`. Other values are optional.

```Toml
provider = "container" # required

[settings]
tls_verify = false     # optional, TLS verification, default: true
username = "USERNAME"  # optional, username to use
password = "PASSWORD"  # optional, password to use
```

Instead of specifying `username` and `password` directly, a central
`containers-auth.json(5)` file can be used, see
[Container registry credentials](./container-auth.md).
