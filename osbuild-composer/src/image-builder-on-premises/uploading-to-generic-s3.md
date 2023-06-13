# Uploading an image to to a bucket in a Generic S3 server

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
- If your server is using HTTPS with a certificate signed by your own CA, you can either pass the CA bundle by setting the field `ca_bundle`, pointing it to the CA's public certificate, or skip SSL verification by setting `skip_ssl_verification` to `true`

Once everything is configured, you can trigger a compose as usual with additional image name and cloud provider profile:
```
$ sudo composer-cli compose start base-image-with-tmux qcow2 IMAGE_KEY generic-s3-config.toml
```
