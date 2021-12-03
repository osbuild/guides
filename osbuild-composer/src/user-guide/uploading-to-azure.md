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
