# Uploading an ostree commit to a Pulp server with ostree support

`osbuild-composer` provides the users with a convenient way to upload ostree commits to a Pulp instance with [ostree content support](https://pulpproject.org/content-plugins/#ostree).

Using a text editor of your choice, create a configuration file with the following content:

```toml
provider = "pulp.ostree"

[settings]
server_address = "PULP_ADDRESS"
repository = "REPO_NAME"
basepath = "REPO_PATH"
username = "USERNAME"
password = "PASSWORD"
```

- `PULP_ADDRESS` is the URL of the Pulp instance.
- `REPO_NAME` is the name of the repository to import the commit into. If a repository with that name does not exist, it will be created, otherwise the commit will be imported into an existing ostree repository.
- `REPO_PATH` is the path that the repository will be distributed through. If the repository already exists, this option has no effect. If the repository does not exist and the repository will be created, this option is required.
- `USERNAME` and `PASSWORD` are the credentials for a user that can push to pulp on that server.

Once everything is configured, you can trigger a compose as usual with additional image name and cloud provider profile:
```
$ sudo composer-cli compose start base-image-with-tmux iot-commit IMAGE_KEY pulp-ostree-config.toml
```

The `IMAGE_KEY` value has no effect for this provider.

The `pulp.ostree` provider supports `iot-commit` and `edge-commit` image types only.
