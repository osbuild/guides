# Image Builder on premises

{{#include image-builder-on-premises.svg}}

`osbuild-composer` is a service for building customized operating system images (currently only Fedora and RHEL). These images can be used with various virtualization software such as [QEMU](https://www.qemu.org/), [VirtualBox](https://www.virtualbox.org/), [VMWare](https://www.vmware.com/) and also with cloud computing providers like [AWS](https://aws.amazon.com/), [Azure](https://azure.microsoft.com/) or [GCP](https://cloud.google.com/).

There are two frontends that you can use to communicate with osbuild-composer:

- **Cockpit Composer**: The web-based management console Cockpit comes bundled with a UI extension to build operating system artifacts. See the documentation of Cockpit Composer for information, or consult the Cockpit Guide for help on general Cockpit questions.

- **Command-line Interface**: With composer-cli there exists a linux command-line interface (CLI) to some of the functionality provided by OSBuild. The CLI is part of the Weldr project, a precursor of OSBuild.

This guide contains instructions on installing `osbuild-composer` service and its basic usage.

If you want to fix a typo, or even contribute new content, the sources for this webpage are hosted in [osbuild/guides GitHub repository](https://github.com/osbuild/guides/).

For Red Hatters, the internal guides can be found [here](https://osbuild.pages.redhat.com/internal-guides/).
