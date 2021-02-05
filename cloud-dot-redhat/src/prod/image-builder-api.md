# Using Image Builder in production environment from command line

All you need to access the service is the “curl” tool which is available on any operating system and [an appropriately privileged user account](./image-builder-access.md).

## OAuth2 Token

Following the [instructions](https://access.redhat.com/articles/3626371), obtain an OAuth2 access token, fom now on referred to as `$token`.

## Testing access

Using your access token, you can display the available API endpoints like this:
```
curl -H "Authorization: Bearer $token" \
        https://cloud.redhat.com/api/image-builder/v1/openapi.json
```

*Tip: If you want nice formatting of the JSON object, use the “jq” tool):*
```
curl -H "Authorization: Bearer $token" \
        https://cloud.redhat.com/api/image-builder/v1/openapi.json | jq .
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
curl -H "Authorization: Bearer $token" \
        -d "@image-builder-request.json" -X POST \
        -H "Content-Type: application/json" \
        https://cloud.redhat.com/api/image-builder/v1/compose
{"id":"cb13b6e6-d24e-4417-a4f4-547f410609e1"}
```

The command specifies:
 * JSON body of the message (-d)
 * HTTP POST method (-X)
 * HTTP header “Content-Type” (-H)
 * The output contains id of the newly created compose which you can use to monitor it:
```
curl -H "Authorization: Bearer $token" \
        -H "Content-Type: application/json" \
        https://cloud.stage.redhat.com/api/image-builder/v1/composes/cb13b6e6-d24e-4417-a4f4-547f410609e1
{"status":"running"}
```

Once the status is “success”, you can go to your AWS account and check the list of available AMIs. The AMI will appear in `us-east-1`.

*(TODO: we need a way to identify the AMI and region)*
