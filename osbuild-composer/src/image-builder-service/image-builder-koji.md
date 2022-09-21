# Image Builder Koji integration

This document describes how various instances of the [Koji](https://koji.build/) build system can and do integrate with the Image Builder service.

## Architecture

`osbuild-composer` can integrate with a `Koji` instance as an external Content Generator using the [koji-osbuild](https://github.com/osbuild/koji-osbuild) plugin. The overview of the integration is described in the [koji-osbuild project README](https://github.com/osbuild/koji-osbuild/blob/main/README.md).

In short, a `Koji` instance integrates directly with `osbuild-composer` API, usually as a separate tenant with a dedicated set of workers.

## Technology Stack

The [koji-osbuild](https://github.com/osbuild/koji-osbuild) plugin is implemented in Python, and the list of dependencies can be found in the [SPEC file](https://github.com/osbuild/koji-osbuild/blob/main/koji-osbuild.spec).

## Building images via Koji integration

The `koji-osbuild` plugin allows one to submit image builds via the Koji Hub API using the `osbuildImage` task. The accepted arguments schema of the `osbuildImage` task is described [in the plugin implementation](https://github.com/osbuild/koji-osbuild/blob/cc3e621754d99b3f2a4fcb85bb207bf76a0970ea/plugins/hub/osbuild.py#L12-L225). The `koji-osbuild` plugin processes the request and submits a new compose request using the `osbuild-composer` Cloud API. The plugin always sets the [`koji` property](https://github.com/osbuild/osbuild-composer/blob/434362e81ed996e6353360c3311090ae1cbe73a6/internal/cloudapi/v2/openapi.v2.yml#L736-L737) in the compose request, signaling to `osbuild-composer` that the request is coming from a `Koji` plugin.

Images built as part of a compose requests submitted via the `Koji` plugin are always implicitly uploaded to the respective `Koji` instance. Since **version 10** of the `koji-osbuild` plugin, images can be uploaded directly to the appropriate cloud environment, in addition to being uploaded to `Koji`. More details are below in the [Cloud upload](#cloud-upload) section.

The `koji-osbuild` plugin also supports specifying all image customizations supported by the `osbuild-composer` Cloud API.

There are currently two easy ways how to trigger the `osbuildImage` tasks in Koji:

* `koji-cli` - the command line client for interacting with `Koji`. The prerequisite is to install the `koji-osbuild-cli` plugin. For more information, run `koji osbuild-image --help`.
* [Pungi](https://pagure.io/pungi) - a distribution compose tool. `Pungi` interacts directly with the `Koji` Hub API and is able to submit `osbuildImage` tasks as part of a distribution compose. The details of how to configure `Pungi` to trigger image builds are described in the [project documentation](https://docs.pagure.org/pungi/configuration.html#osbuild-composer-for-building-images).

### Cloud upload

#### Prerequisites

* `koji-osbuild` version >= 10
* `osbuild-composer` version >= 58

#### Details

Images built via the `Koji` integration can be automatically uploaded to the appropriate cloud environment, in addition to the `Koji` instance. In order for this to happen, one must provide *upload_options* when using the `osbuildImage` task and the integrated `osbuild-composer` instance must be configured appropriately to be able to upload to the respective cloud environments.

Currently supported *upload_options* are:

* AWS EC2
* AWS S3
* Azure (as an image)
* GCP
* Container registry

Please note, that each image type can be uploaded only to its respective cloud target, represented by *upload_options* (e.g. the `ami` image can be uploaded only to `AWS EC2`, `gce` image can be uploaded only to `GCP`, etc.).

The allowed *upload_options* schema is defined in the [`koji-osbuild` Hub plugin](https://github.com/osbuild/koji-osbuild/blob/cc3e621754d99b3f2a4fcb85bb207bf76a0970ea/plugins/hub/osbuild.py#L110-L118) and currently matches the [`osbuild-composer` Cloud API `UploadOptions`](https://github.com/osbuild/osbuild-composer/blob/434362e81ed996e6353360c3311090ae1cbe73a6/internal/cloudapi/v2/openapi.v2.yml#L809-L815).

If the compose request contains multiple image requests (meaning that multiple images will be built), the provided *upload_options* will be used as is for all images (with all its consequences).

All the necessary data to locate the image in the cloud is attached by the `koji-osbuild` plugin to the image build task in `Koji` as `compose-status.json` file. Below is an example of such file:

```json
{
    "image_statuses": [
        {
            "status": "success",
            "upload_status": {
                "options": {
                    "ami": "ami-02e34403c421dfc17",
                    "region": "us-east-1"
                },
                "status": "success",
                "type": "aws"
            }
        }
    ],
    "koji_build_id": 1,
    "koji_task_id": null,
    "status": "success"
}
```

#### Cloud upload via `koji-cli`

In order to upload images to the cloud when using `koji-cli`, one must first create a JSON file with the appropriate *upload_options*.

Example `gcp_upload_options.json`:

```json
{
    "region": "eu",
    "bucket": "my-bucket",
    "share_with_accounts": ["alice@example.org"]
}
```

Then add the `--upload-options=gcp_upload_options.json` argument to the command line when calling `koji` CLI.

#### Cloud upload via `Pungi`

In order to upload images to the cloud, when using `Pungi` to trigger image builds, one must specify the `upload_options` option in the configuration dictionary as described in the [project documentation](https://docs.pagure.org/pungi/configuration.html#osbuild-composer-for-building-images).

Please note that the support for cloud upload has been merged to the `Pungi` project after the **4.3.6** release. Therefore, if you want to take advantage of this feature, make sure to use version higher than **4.3.6**.
