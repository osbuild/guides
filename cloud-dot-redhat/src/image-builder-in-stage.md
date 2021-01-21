# Using Image Builder in staging environment from command line

Image Builder is an instance of an upstream image-builder project. You can find the code and report issues here: [https://github.com/osbuild/image-builder](https://github.com/osbuild/image-builder) . It provides a REST API, which is accessible using the squid.corp.redhat.com HTTP proxy server.

All you need to access the service is the “curl” tool which is available on any operating system and a user account in staging environment.

## Create account in stage environment

Start here:[http://account-manager-stage.app.eng.rdu2.redhat.com/#create](http://account-manager-stage.app.eng.rdu2.redhat.com/#create)

and:
 * Fill in a username which is **not** your kerberos name, for example `image-builder-<nickname>`.
 * Create new password which is **not** your kerberos password.
 * Assign yourself these SKUs (something like a subscription): `MCT3475, MCT3718, RH0105260, RH00003`
 * Check that you are creating the account in "Stage" environment
 * Click "Create"

Now using your new account name and password, you can display the available API endpoints like this:
```
curl --user "username:password" \
        --proxy http://squid.corp.redhat.com:3128 \
        https://cloud.stage.redhat.com/api/image-builder/v1/openapi.json
```

The command contains:
 * Your credentials after the `--user` CLI flag
 * URL of the HTTP proxy server after the `--proxy` flag
 * URL of the staging instance of image builder

*Tip: If you want nice formatting of the JSON object, use the “jq” tool):*
```
curl --user "username:password" \
        --proxy http://squid.corp.redhat.com:3128 \
        https://cloud.stage.redhat.com/api/image-builder/v1/openapi.json | jq .
```

## Creating and monitoring a compose

There are two main API endpoints related to starting and monitoring a compose:
 1. HTTP POST /v1/compose
 2. HTTP GET /v1/composes/<uuid>

If you want to start a new compose, you need to create a JSON file containing a description of the image you want. For example:
```json
{
  "distribution": "rhel-8",
  "image_requests": [
    {
      "architecture": "x86_64",
      "image_type": "ami",
      "upload_requests": [
        {
          "type": "aws",
          "options": {
            "share_with_accounts": [
              "your-aws-account-id"
            ]
          }
        }
      ]
    }
  ]
}
```

Save the file as “image-builder-request.json”.

Given the JSON file above, Image Builder will create a RHEL 8 AMI image for x86\_64 architecture and upload it to AWC EC2. It will then share the AMI with the specified account.

Now trigger a new compose:
```
curl --user "username:password" \
        --proxy http://squid.corp.redhat.com:3128 \
        -d "@image-builder-request.json" -X POST \
        -H "Content-Type: application/json" \
        https://cloud.stage.redhat.com/api/image-builder/v1/compose
{"id":"cb13b6e6-d24e-4417-a4f4-547f410609e1"}
```

The command specifies:
 * JSON body of the message (-d)
 * HTTP POST method (-X)
 * HTTP header “Content-Type” (-H)
 * The output contains id of the newly created compose which you can use to monitor it:
```
curl --user "username:password" \
        --proxy http://squid.corp.redhat.com:3128 \
        -H "Content-Type: application/json" \
        https://cloud.stage.redhat.com/api/image-builder/v1/composes/cb13b6e6-d24e-4417-a4f4-547f410609e1
{"status":"running"}
```

Once the status is “success”, you can go to your AWS account and check the list of available AMIs.

*(TODO: we need a way to identify the AMI)*
