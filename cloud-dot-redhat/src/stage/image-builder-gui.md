# Using Image Builder in staging environment from the graphical interface (GUI)

Note: The staging environment requires access to the Red Hat VPN, so is only accessible to Red Hat employees. This is the first step in our deployment and the production environment will be available to select outside users soon!

Image Builder is an instance of an upstream image-builder project. You can find the code and report issues here: [https://github.com/osbuild/image-builder] (https://github.com/osbuild/image-builder) . It provides a REST API, which is accessible using the squid.corp.redhat.com HTTP proxy server.

To access the service, you need to create an account to access the stage environment.

## Create account in stage environment

1. Access [Ethel](http://account-manager-stage.app.eng.rdu2.redhat.com/#create).

2. Insert a username which is **not** your kerberos name, for example `image-builder-<username>`.

3. Insert a password which is **not** your kerberos password.

4. Assign a SKUs, for example, `MCT3475, MCT3718, RH0105260, RH00003`. [See `Subscription Inventory` for more details](https://access.redhat.com/management/subscriptions).

5. Select the option "Stage" from the dropdown menu, to grant access to the "Stage" environment.

6. Click "Create".

The account is now sucessfully created.

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