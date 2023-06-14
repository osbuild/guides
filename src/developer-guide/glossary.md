# Glossary

| Term                 | Explanation                                                  |
| -------------------- | ------------------------------------------------------------ |
| AMI                  | Amazon Machine Image (image type)                            |
| Blueprint            | Definition of customizations in the image                    |
| Compose              | Request from the user that produces one or more images. Images in a single compose are, in theory, the same, but for different platforms, such as Azure or AWS. In practice they are slightly different because every cloud platform requires a different package set and system configuration. osbuild-composer running the Weldr API can only create one image at a time, so one compose maps directly to one image build. It can map to multiple image builds when used with other APIs, such as the Koji API. |
| Composer API         | HTTP API meant as publicly accessible (over TCP). It was created specifically for osbuild-composer and does not support some Weldr features like blueprint management, but adds new features like building different distros and architectures. |
| GCP                  | Google Cloud Platform |
| Image Build          | One request from osbuild-composer to osbuild-worker. Its result is a single image. |
| Image Type           | Image file format usually associated with a specific use case. For example: AMI for AWS, qcow2 for OpenStack, etc. |
| Manifest             | Input for the osbuild tool. It should be a precise definition of an image. See https://www.osbuild.org/man/osbuild-manifest.5 for more information. |
| osbuild              | Low-level tool for building images. Not meant for end-user usage. |
| osbuild-composer     | HTTP service for building OS images.                         |
| OSTree               | Base technology for immutable OS images: Fedora IoT and RHEL Edge |
| Repository overrides | osbuild-composer uses its own set of repository definitions. In case a user wants to use custom repositories, "overrides" can be created in /etc/osbuild-composer |
| Weldr API            | Local HTTP API used for communication between composer-cli/cockpit-composer and osbuild-composer. It comes from the lorax-composer project. |
