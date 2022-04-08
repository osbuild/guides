# Uploading an image to an AWS S3 Bucket

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
