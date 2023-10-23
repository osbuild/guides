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

**Note:** Starting with `osbuild-composer` version 91, the cloud upload target results are also attached to the image output metadata as well as to the build metadata. See the [Image outout metadata](#image-output-metadata) section for more details.

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

## Type-specific metadata

`osbuild-composer` attaches extra metadata to a Koji build as well as to each of the outputs attached to a Koji build.

### Output metadata

`osbuild-composer` attaches the following outputs for each of the built images to the build:

* built image
* osbuild manifest
* osbuild logs

All outputs have the (build) type set to `image`, except for the log, which don't have any (build) type set and also have no metadata attached. The metadata attached to the image and manifest outputs is described below.

`osbuild-composer` uses the `image` type for all image builds via Koji, and so type-specific information is placed into the `extra.image` map of each output. Note that this is a legacy type in Koji and may be changed to use `extra.typeinfo.image` in the future. Clients fetching such data should first look for it within `extra.typeinfo.image` and fall back to `extra.image` when the former is not available.

#### Image output metadata

Data attached to the image output as metadata under `extra.image`:

* `arch` - architecture of the image
* `boot_mode` - boot mode of the image. Can be one of:
  * `legacy`
  * `uefi`
  * `hybrid`
  * `none`
* `osbuild_artifact` - information about the osbuild configuration used to produce the image
  * `export_filename` - filename of the image artifact as produced by osbuild
  * `export_name` - name of the manifest pipeline that was exported to produce the image
* `osbuild_version` - version of osbuild used to produce the image
* `upload_target_results` - **optional** list of cloud upload target results, if the image build request contained request to upload the image also to a specific cloud environment, in addition to Koji. Each entry in the list contains:
  * `name` - name of the upload target
  * `options` - upload-target specific options with information to locate the image in the cloud environment
  * `osbuild_artifact` - information about the osbuild configuration used to produce the image for this specific upload target. Technically, osbuild can export multiple different artifacts from the same manifest, but in reality, this is not used at this point.
    * `export_filename` - filename of the image artifact as produced by osbuild
    * `export_name` - name of the manifest pipeline that was exported to produce the image

##### Example of image output metadata

The following example shows the metadata attached to an image output under the `extra.image` key:

```json
{
  "arch": "x86_64",
  "boot_mode": "hybrid",
  "osbuild_artifact": {
    "export_filename": "image.raw.xz",
    "export_name": "xz"
  },
  "osbuild_version": "93",
  "upload_target_results": [
    {
      "name": "org.osbuild.aws",
      "options": {
        "ami": "ami-0d06fff61b0395df0",
        "region": "us-east-1"
      },
      "osbuild_artifact": {
        "export_filename": "image.raw.xz",
        "export_name": "xz"
      }
    }
  ]
}
```

#### Manifest output metadata

Data attached to the manifest output as metadata under `extra.image`:

* `arch` - architecture of the image produced by the manifest
* `info` - additional information about the manifest
  * `osbuild_composer_version` - version of `osbuild-composer` used to produce the manifest
  * `osbuild_composer_deps` - list of `osbuild-composer` dependencies, which could affect the content of the manifest. Each entry in the list contains:
    * `path` - Go module path of the dependency
    * `version` - version of the dependency
    * `replace` - **optional** Go module path of the replacement module, if the dependency was replaced
      * `path` - Go module path of the replacement module
      * `version` - version of the replacement module

##### Example of manifest output metadata

The following example shows the metadata attached to a manifest output under the `extra.image` key:

```json
{
  "arch": "x86_64",
  "info": {
    "osbuild_composer_version": "git-rev:f6e0e993919cb114e4437299020e80032d0e40a7",
    "osbuild_composer_deps": [
      {
        "path": "github.com/osbuild/images",
        "version": "v0.7.0"
      }
    ]
  }
}
```

### Build metadata

The metadata attached by `osbuild-composer` to the Koji build itself is a compilation of the metadata attached to the individual outputs. The individual output metadata are always nested under the output's filename.

Image output metadata are nested under the `extra.typeinfo.image` key, manifest output metadata are nested under the `extra.osbuild_manifest` key.

#### Example of build metadata

The following example shows the metadata attached to a Koji build under the `extra` key:

```json
{
  "typeinfo": {
    "image": {
      "name-version-release.x86_64.raw.xz": {
        "arch": "x86_64",
        "boot_mode": "hybrid",
        "osbuild_artifact": {
          "export_filename": "image.raw.xz",
          "export_name": "xz"
        },
        "osbuild_version": "93",
        "upload_target_results": [
          {
            "name": "org.osbuild.aws",
            "options": {
              "ami": "ami-0d06fff61b0395df0",
              "region": "us-east-1"
            },
            "osbuild_artifact": {
              "export_filename": "image.raw.xz",
              "export_name": "xz"
            }
          }
        ]
      }
    }
  },
  "osbuild_manifest": {
    "name-version-release.x86_64.raw.xz.manifest.json": {
      "arch": "x86_64",
      "info": {
        "osbuild_composer_version": "git-rev:f6e0e993919cb114e4437299020e80032d0e40a7",
        "osbuild_composer_deps": [
          {
            "path": "github.com/osbuild/images",
            "version": "v0.7.0"
          }
        ]
      }
    }
  }
}
```

## Locally reproducing Koji image builds

With the osbuild manifest and additional metadata attached to each Koji build, it is now possible to locally reproduce the Koji image builds. Note that the `osbuild` tool supports building images only for the same architecture as the host system.

To rebuild the image locally, one must first download the osbuild manifest from the Koji build. The osbuild manifest is attached to the Koji build as a separate output with the `.manifest.json` suffix. It is also desirable to ensure that the same `osbuild` version is used to build the image locally as was used to build the image in Koji. The `osbuild` version used to build the image is attached to the Koji build as the `osbuild_version` key in the metadata.

To build the image locally using the downloaded osbuild manifest, run the following command:

```bash
# Set OUTPUT_DIR to a directory path for the image build artifacts
# Set STORE_DIR to a directory path for the osbuild cache
# Set MANIFEST_PATH to the path to the downloaded osbuild manifest
# Set EXPORT_NAME to the name of the manifest pipeline that was exported to produce the image (`export_name` key in the image output metadata).

sudo osbuild --output-directory ${OUTPUT_DIR} --store ${STORE_DIR} --export ${EXPORT_NAME} ${MANIFEST_PATH}
```

Once the image build finishes, the image artifact will be available in the `${OUTPUT_DIR}/<export_name>/<export_filename>` path. The `export_name` and `export_filename` keys are attached to the image build metadata and described in the [Image output metadata](#image-output-metadata) section.
