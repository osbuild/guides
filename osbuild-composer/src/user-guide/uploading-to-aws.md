# Uploading an image to AWS

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
