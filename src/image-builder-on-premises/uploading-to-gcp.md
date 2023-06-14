# Uploading an image to GCP

`osbuild-composer` provides the users with a convenient way to upload images directly to GCP right after the image is built. Before you can use this feature, you have to provide credentials for your user or service account, which you would like to use for uploading images to GCP.

The account associated with the credentials must have at least the following IAM roles assigned:

- `roles/storage.admin` - to create and delete storage objects
- `roles/compute.storageAdmin` - to import a VM image to Compute Engine

Now, you are ready to upload your first image to GCP.

Using a text editor of your choice, create a configuration file `gcp-config.toml` with the following content:

```toml
provider = "gcp"

[settings]
bucket = "GCP_BUCKET"
region = "GCP_STORAGE_REGION"
object = "OBJECT_KEY"
credentials = "GCP_CREDENTIALS"
```

There are several considerations when filling values in this file:

- `GCP_BUCKET` must point to an existing bucket.
- `GCP_STORAGE_REGION` can be a [regular Google storage region, but also a dual or multi region](https://cloud.google.com/storage/docs/locations#location-r).
- `OBJECT_KEY` is the name of an intermediate storage object. It must not exist before the upload, and it will be deleted when the upload process is done. If the object name does not end with `.tar.gz`, the extension is automatically added to the object name.
- `GCP_CREDENTIALS` is a Base64 encoded content of the credentials JSON file downloaded from GCP. The credentials are used to determine the GCP project to upload the image to.
   > Specifying this value in the `gcp-config.toml` may be optional if you use a different mechanism of authenticating with GCP. For more information about the various ways of authenticating with GCP, read the [Authenticating with GCP](#authenticating-with-gcp) below.

After everything is configured, you can trigger a compose as usual with an additional image name and cloud provider profile:

```bash
sudo composer-cli compose start base-image-with-tmux gce IMAGE_KEY gcp-config.toml
```

where *IMAGE_KEY* will be the name of your new GCE image, once it is uploaded to GCP.

## Authenticating with GCP

osbuild-composer supports multiple ways of authenticating with GCP.

In case the osbuild-composer is configured to authenticate with GCP in multiple ways, it uses them in the following order of preference:

1. Credentials specified with the `composer-cli` command in the configuration file.
2. Credentials configured in the osbuild-composer worker configuration.
3. Application Default Credentials from the Google GCP SDK library, which tries to automatically find a way to authenticate using the following options:
   1. If `GOOGLE_APPLICATION_CREDENTIALS` environment variable is set, it tries to load and use credentials from the file pointed to by the variable.
   2. It tries to authenticate using the service account attached to the resource which is running the code (e.g. Google Compute Engine VM).

> **Note that the GCP credentials are used to determine the GCP project to upload the image to.** Therefore, unless you want to upload all of your images to the same GCP project, you should always specify credentials with the `composer-cli` command.

### Specifying credentials with the `composer-cli` command

You need to specify the credentials with the `composer-cli` command in the provided upload target configuration `gcp-config.toml`:

```toml
provider = "gcp"

[settings]
...
credentials = "GCP_CREDENTIALS"
```

The `GCP_CREDENTIALS` value is a Base64 encoded content of the Google account credentials JSON file. The reason for this is that the file is quite large and contains multiple key values, therefore mapping them to the TOML configuration format would require more manual work from the user, than encoding the whole file in Base64 and specifying it as a single value.

To get the encoded content of the Google account credentials file with the path stored in `GOOGLE_APPLICATION_CREDENTIALS` environment variable, run:

```bash
base64 -w 0 "${GOOGLE_APPLICATION_CREDENTIALS}"
```

### Specifying credentials in the osbuild-composer worker configuration

You can configure the credentials to be used for GCP globally for all image builds in the worker configuration `/etc/osbuild-worker/osbuild-worker.toml`:

```toml
[gcp]
credentials = "PATH_TO_GCP_ACCOUNT_CREDENTIALS"
```
