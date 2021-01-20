# Using Image Builder in staging environment from command line

Image Builder is an instance of an upstream image-builder project. You can find the code and report issues here: https://github.com/osbuild/image-builder . It provides a REST API, which is accessible using the squid.corp.redhat.com HTTP proxy server.

All you need to access the service is the “curl” tool which is available on any operating system. You will also need to know your Red Hat username and password (which one?)

*(TODO: How is it possible that it did not exist for Tom and why does he have such a funny username?).*

To display the available API endpoints and all the JSON objects you can use invoke this command:
```
curl --user "username@redhat.com:password" \
        --proxy http://squid.corp.redhat.com:3128 \
        https://cloud.stage.redhat.com/api/image-builder/v1/openapi.json
```

The command contains:
 * Your credentials after the `--user` CLI flag
 * URL of the HTTP proxy server after the `--proxy` flag
 * URL of the staging instance of image builder

*Tip: If you want nice formatting of the JSON object, use the “jq” tool):*
```
curl --user "username@redhat.com:password" \
        --proxy http://squid.corp.redhat.com:3128 \
        https://cloud.stage.redhat.com/api/image-builder/v1/openapi.json | jq .
```

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
curl --user "username@redhat.com:password" \
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
curl --user "username@redhat.com:password" \
        --proxy http://squid.corp.redhat.com:3128 \
        -H "Content-Type: application/json" \
        https://cloud.stage.redhat.com/api/image-builder/v1/composes/cb13b6e6-d24e-4417-a4f4-547f410609e1
{"status":"running"}
```

Once the status is “success”, you can go to your AWS account and check the list of available AMIs.

*(TODO: we need a way to identify the AMI)*
