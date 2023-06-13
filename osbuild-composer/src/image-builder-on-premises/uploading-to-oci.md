# Uploading an image to OCI

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
