# Local Cloud API Development

The following instructions assume you are running osbuild-composer in a local
VM on some version of Fedora and that you have the osbuild-composer github
repository available.  The VM should have ssh access from the host system. In
these examples I use `localvm` as an alias for the VM's ssh settings in my
`~/.ssh/config` file.

## Setup Local API Access

The osbuild-composer cloud api listens to port 443, but it requires SSL
certificates in order to authenticate the requests.  You can generate the
needed certificates using a slightly modified script from the `./tools/`
directory, and the system running the script needs to have `openssl` installed
on it.

These changes will let you use curl on the VM to POST the composer api json
request files to the service listening on 127.0.0.1:443.

From the osbuild-composer git repo copy `./tools/gen-certs.sh` and
`./test/data/x509/openssl.cnf` to a temporary directory.  Edit the
`gen-certs.sh` script and replace all of the `subjectAltName=` entries with
`subjectAltName=IP:127.0.0.1` and generate new certs like so:

    ./gen-certs.sh /tmp/openssl.cnf /tmp/local-certs/ /tmp/working-certs/

Copy the new certs to the VM:

    scp /tmp/local-certs/* localvm:/etc/osbuild-composer/

ssh into the VM and stop any currently running osbuild services and then start
the cloud api socket service by running:

    systemctl stop '*osbuild*'
    systemctl start osbuild-composer-api.socket osbuild-remote-worker.socket

Make a helper script to POST json cloud api requests to the service.  Save this
in a file named `start-cloudapi` on the VM:

``` bash
#!/usr/bin/sh
curl -v -k --cert /etc/osbuild-composer/client-crt.pem \
--cacert /etc/osbuild-composer/ca-crt.pem \
--key /etc/osbuild-composer/client-key.pem \
https://localhost/api/image-builder-composer/v2/compose \
--header 'Content-Type: application/json' \
--data @$1
```

Now you need a simple request to create a guest (qcow2) image.  This uses Fedora 38, and
doesn't include gpg key checking.  Save this as `simple-guest.json`:

``` json
{
  "distribution": "fedora-38",
  "image_request":
    {
      "architecture": "x86_64",
      "image_type": "guest-image",
      "repositories": [
          {
            "name": "fedora",
            "metalink": "https://mirrors.fedoraproject.org/metalink?repo=fedora-38&arch=x86_64",
            "check_gpg": false
          },
          {
            "name": "updates",
            "metalink": "https://mirrors.fedoraproject.org/metalink?repo=updates-released-f38&arch=x86_64",
            "check_gpg": false
          },
          {
            "name": "fedora-modular",
            "metalink": "https://mirrors.fedoraproject.org/metalink?repo=fedora-modular-38&arch=x86_64",
            "check_gpg": false
          },
          {
            "name": "updates-modular",
            "metalink": "https://mirrors.fedoraproject.org/metalink?repo=updates-released-modular-f38&arch=x86_64",
            "check_gpg": false
          }
      ]
    }
}
```

Use `./start-cloudapi simple-guest.json` start the build.  You should get a JSON response similar to this:

``` json
{"href":"/api/image-builder-composer/v2/compose","kind":"ComposeId","id":"f3ac9290-23c0-47b4-bb9e-cadee85d1340"}
```

This will run the build, but since it doesn't have any upload instructions it
will fail at the upload step and delete the image from the local system.
`journalctl -f` will show the progress and the upload error.

If you want to upload results to a service include the upload details in the request.  If you
want to save the results locally continue to the next section.


## Skip upload and save locally

You can configure osbuild-composer to save the image locally and not try to
upload it.  This allows you to examine the image, or copy it somewhere to do a
test boot of it.  This is not enabled normally because there are no provisions
for cleaning up the images -- you need to do that manually before your disk
runs out of space.

The `local_save` upload option is enabled by setting an environmental variable
in the `osbuild-composer.service` file.  You can either edit the file directly,
which will need to be replaced every time you update the osbuild-composer rpm,
or you can create a drop-in file by running `systemctl edit
osbuild-composer.service` and adding these lines:

    [Service]
    Environment="OSBUILD_LOCALSAVE=1"

You can confirm the change by running `systemctl cat osbuild-composer.service`.
Now stop the local osbuild-composer services and start the cloudapi service by
running:

    systemctl stop '*osbuild*'
    systemctl start osbuild-composer-api.socket osbuild-remote-worker.socket

Make a new composer api request json file with the `local_save` upload option
set to true.  Copy the `simple-guest.json` example to `local-guest.json` and add
the `upload_options` section:

``` json
{
  "distribution": "fedora-38",
  "image_request":
    {
      "architecture": "x86_64",
      "image_type": "guest-image",
      "upload_options": {
          "local_save": true
      },
      "repositories": [ ... SAME AS PREVIOUS EXAMPLE ... ]
    }
}
```

You can now run `./start-cloudapi local-guest.json` to start the build.  You
should get a JSON response similar to this:

    {"href":"/api/image-builder-composer/v2/compose","kind":"ComposeId","id":"4674e0d3-ecb3-4cbe-9c31-ca14b7425eaa"}

and monitor the progress with `journalctl -f`.  When the compose is finished the
result will be saved in
`/var/lib/osbuild-composer/artifacts/4674e0d3-ecb3-4cbe-9c31-ca14b7425eaa`

Remember to monitor your disk usage, it can fill up quickly if you do not delete old artifact
entries.  These are un-managed, unlike the store used with the weldr api, so they can be removed manually with a simple `rm -rf /var/lib/osbuild-composer/artifacts/*`
