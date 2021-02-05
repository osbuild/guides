# Using Image Builder in staging environment from the graphical interface (GUI)

To access the service, you need to [create an account](./image-builder-access.md) to access the stage environment, and configure your browser to use [a proxy](http://hdn.corp.redhat.com/proxy.pac) on the Red Hat VPN.

## Accessing the stage environment on the graphical interface (GUI)

Now using your new account name and password, you can display the available API endpoints like this:

1. Access the [Stage Beta](https://cloud.stage.redhat.com/beta/insights/image-builder/landing).

2. Login with the credentials you created for the stage environment. See previous section for more details.

You are now able to create and monitor your composes.

## Creating and monitoring a compose using the graphical interface (GUI)

On the [Stage Beta](https://cloud.stage.redhat.com/beta/insights/image-builder/landing), perform the follow steps:

1. Click **Create Image**. A `Create a new image` dialog window opens.

2. On the `Image output` step, choose: 
* Select the image release.
* Select the Target environment: `Amazon Web Services`. environment.
Click **Next**.

3. On the `Upload to AWS` step, enter your AWS account ID. 
You can find your AWS account ID by accessing the option [My account](https://console.aws.amazon.com/billing/home?#/account). Click **Next**.

4. On the `Registration` step,  you have the option to embed an activation key and register systems on first boot or register later. Select the option that better suits you. Click **Next**.

5. On the `Review` step , review the information and click **Create**.

Image Builder as a Service creates a RHEL 8 AMI image for the `x86\_64` architecture and uploads it to AWS EC2. It will then share the AMI with the specified account.

NOTE: The image build, upload and cloud registration processes take a few minutes to complete.

Once the status is “Success”, access your AWS account and check the list of available AMIs images. The AMI is available in the `us-east-1` region.