# Uploading an image to AWS

`osbuild-composer` provides the users with a convenient way to upload images directly to AWS right after the image is built. To use this feature, you need one additional configuration file for the cloud provider. In this example, the cloud provider is AWS:

```toml
provider = "aws"

[settings]
accessKeyID = "AWS_ACCESS_KEY_ID"
secretAccessKey = "AWS_SECRET_ACCESS_KEY"
bucket = "AWS_BUCKET"
region = "AWS_REGION"
key = "IMAGE_KEY"
```

But be careful here, if you are using [IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html), which is the recommendation, according to the AWS documentation, you need a specific policy that will allow VM import from the S3 bucket to EC2. See the [AWS documentation](https://docs.aws.amazon.com/vm-import/latest/userguide/vmie_prereqs.html) for more details.

Once everything is configured, you can trigger a compose as usual with additional image name and cloud provider profile:
```
$ sudo composer-cli compose start base-image-with-tmux ami IMAGE_KEY aws-config.toml
```
where IMAGE_KEY will be the name of your VM Image, once it is uploaded to EC2.


