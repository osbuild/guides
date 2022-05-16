# Uploading cloud images

`osbuild-composer` can upload images to a cloud provider right after they are built. The configuration is slightly different for each cloud provider. See individual subsections of this documentation.

## Uploading an image to AWS

`osbuild-composer` provides the users with a convenient way to upload images directly to AWS right after the image is built. Before you can use this feature, you have to define `vmimport` IAM role in your AWS account. See [VM Import/Export Requirements](https://docs.aws.amazon.com/vm-import/latest/userguide/vmie_prereqs.html#vmimport-role) in AWS documentation.

Now, you are ready to upload your first image to AWS. Using a text editor of your choice, create a configuration file with the following content: 

```toml
provider = "aws"

[settings]
accessKeyID = "AWS_ACCESS_KEY_ID"
secretAccessKey = "AWS_SECRET_ACCESS_KEY"
bucket = "AWS_BUCKET"
region = "AWS_REGION"
key = "OBJECT_KEY"
```

There are several considerations when filling values in this file:
- `AWS_BUCKET` must be in the `AWS_REGION`
- The `vmimport` role must have read access to the `AWS_BUCKET`
- `OBJECT_KEY` is the name of an intermediate S3 object. It must not exist before the upload, and it will be deleted when the process is done.

> If your authentication method requires you to also specify a session token, you can put it in the `settings` section of the configuration file in a field named `sessionToken`.

Once everything is configured, you can trigger a compose as usual with additional image name and cloud provider profile:
```
$ sudo composer-cli compose start base-image-with-tmux ami IMAGE_KEY aws-config.toml
```
where IMAGE_KEY will be the name of your new AMI, once it is uploaded to EC2.

## Uploading an image to an AWS S3 Bucket

`osbuild-composer` provides the users with a convenient way to upload images, of all sorts, directly to an AWS S3 bucket right after the image is built.

Using a text editor of your choice, create a configuration file with the following content:

```toml
provider = "aws.s3"

[settings]
accessKeyID = "AWS_ACCESS_KEY_ID"
secretAccessKey = "AWS_SECRET_ACCESS_KEY"
bucket = "AWS_BUCKET"
region = "AWS_REGION"
key = "OBJECT_KEY"
```

There are several considerations when filling values in this file:
- `AWS_BUCKET` must be in the `AWS_REGION`

> If your authentication method requires you to also specify a session token, you can put it in the `settings` section of the configuration file in a field named `sessionToken`.

Once everything is configured, you can trigger a compose as usual with additional image name and cloud provider profile:
```
$ sudo composer-cli compose start base-image-with-tmux qcow2 IMAGE_KEY aws-s3-config.toml
```

# Uploading an image to Microsoft Azure

`osbuild-composer` builds images and delivers them to [Microsoft Azure]
automatically. These images are ready to use with [virtual machines] in the
Azure cloud.

[Microsoft Azure]: https://azure.microsoft.com/en-us/
[virtual machines]: https://azure.microsoft.com/en-us/services/virtual-machines/

## Initial setup

Before you can upload images to Azure with `osbuild-composer`, your account
needs some initial setup. Be sure to complete these steps

* Create a resource group
* Create a storage account inside the resource group
* Create a storage container within the storage account
* Gather your access keys

For a detailed walkthrough on each step within the Azure portal, review the
[Build RHEL images for Azure with Image Builder] post on the Red Hat Blog.

Make a note of the following items during the setup so you can provide them to
`osbuild-composer` during the build process:

* the name of your storage account
* the name of the storage container inside your storage account
* the access key for your storage account

[Build RHEL images for Azure with Image Builder]: https://www.redhat.com/en/blog/build-rhel-images-azure-image-builder

## Deploy

Push a blueprint containing your image configuration and create a new file
called `azure.toml` that contains the information about your Azure storage
account:

```toml
provider = "azure"

[settings]
storageAccount = "your storage account name"
storageAccessKey = "storage access key you copied in the Azure portal"
container = "your storage container name"
```

Build and deploy the image to Azure:

```shell
composer-cli compose start my_blueprint vhd my_image_key azure.toml
```

In this example `my_blueprint` is the name of the blueprint containing your
image configuration. Replace `my_image_key` with the preferred image name you
want to see in Azure. This is the name that appears inside your storage
container.

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

## Uploading an image to to a bucket in a Generic S3 server

`osbuild-composer` provides the users with a convenient way to upload images, of all sorts, directly to a bucket in a Generic S3 server right after the image is built.

Using a text editor of your choice, create a configuration file with the following content:

```toml
provider = "generic.s3"

[settings]
endpoint = "S3_SERVER_ENDPOINT"
accessKeyID = "S3_ACCESS_KEY_ID"
secretAccessKey = "S3_SECRET_ACCESS_KEY"
bucket = "S3_BUCKET"
region = "S3_REGION"
key = "OBJECT_KEY"
```

There are several considerations when filling values in this file:
- `AWS_REGION` must still be set (e.g. to us-east-1) even if it has no meaning in your S3 server

Once everything is configured, you can trigger a compose as usual with additional image name and cloud provider profile:
```
$ sudo composer-cli compose start base-image-with-tmux qcow2 IMAGE_KEY generic-s3-config.toml
```

## Uploading an image to OCI

`osbuild-composer` provides the users with a convenient way to upload images directly to OCI right after the image is built.
See [Managing Custom Images](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/managingcustomimages.htm) in OCI documentation (includes permissions details).

Now, you are ready to upload your first image to OCI. Using a text editor of your choice, create a configuration file with the following content: 

```toml
provider = "oci"

[settings]
user = "OCI_CLI_USER"
tenancy = "OCI_CLI_TENANCY"
fingerprint = "OCI_CLI_FINGERPRINT"
region = "OCI_CLI_REGION"
bucket = "OCI_BUCKET"
namespace = "OCI_NAMESPACE"
compartment = "OCI_COMPARTMENT"
private_key = '''
...
'''
```

There are several considerations when filling values in this file:
- `OCI_BUCKET` must be in the `OCI_REGION` and must exist before the upload

Once everything is configured, you can trigger a compose as usual with additional image name and cloud provider profile:
```
$ sudo composer-cli compose start BLUEPRINT_NAME oci IMAGE_KEY oci-config.toml
```
where `IMAGE_KEY` will be the name of your new OCI image once uploaded.
