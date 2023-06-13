# Build Reliable Operating System Images

Use Image Builder to create images of your Linux operating system in a reliable fashion, isolating the image creation from your host operating system, and producing a reliable, well-defined image ready to be deployed.

Image Builder provides both the tools to build custom operating system images, as well as applications to deploy hosted image building services. At its core, the [`osbuild`](./developer-guide/osbuild.html) project takes the responsibility of assembling custom images of operating systems according to the precise needs of the user. The [`osbuild-composer`](./developer-guide/osbuild-composer.html) project builds on top of osbuild and implements an image creation service that can be deployed as a hosted service.

## Image Builder on console.redhat.com

{{#include image-builder-service/architecture.svg}}

Image Builder is available as a managed service as part of [Red Hat's Hybrid Cloud Console](https://console.redhat.com). It is used during the development of RHEL and Fedora but is also used by Red Hat’s customers or anyone with a [Red Hat developer license](https://developers.redhat.com/register). It builds customized images of RHEL and CentOS Stream for many footprints (traditional, ostree-based), targets (Public clouds, Baremetal, etc), and architectures (x86, ARM, IBM).

[**Documentation**](./image-builder-service/architecture.html)

## Image Builder on premises

{{#include image-builder-on-premises/image-builder-on-premises.svg}}

Image Builder can also be deployed and self-hosted. In its most basic setup, it will use one osbuild-composer service and one worker running osbuild. The architecture you can build images for will depend on the architecture of the worker, i.e. on an x86 worker you can only build x86 images. You can leverage [blueprints](./image-builder-on-premises/blueprint-reference.html) to customize your images.

[**Documentation**](./image-builder-on-premises/image-builder-on-premises.html)

## How to contribute

All of our code is open source and [on GitHub](https://github.com/osbuild).

Our [developer guide](https://www.osbuild.org/guides/developer-guide/developer-guide.html) is a great starting point to learn about our workflow, code style and more!

### How to reach out to us

* Matrix: `#image-builder` on fedoraproject.org
* Mailing List: [image-builder@redhat.com](mailto:image-builder@redhat.com)
* Issues and pull requests: [github.com/osbuild](https://github.com/osbuild)