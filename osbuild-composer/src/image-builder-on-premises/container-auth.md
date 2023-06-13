# Container registry credentials

All communication with container registries is done by the `osbuild-worker`
service. It can be configured via the `/etc/osbuild-worker/osbuild-worker.toml`
configuration file. It is read only once at service start, so the service
needs to be restarted after making any changes.

The configuration file has a `containers` section with an `auth_file_path`
field that is a string referring to a path of a `containers-auth.json(5)` file
to be used for accessing protected resources. An example configuration could
look like this:

```Toml
[containers]
auth_file_path = "/etc/osbuild-worker/containers-auth.json"
```

For detailed information on the format of the authorization file itself,
refer to the corresponding man page: `man 5 containers-auth.json`.
