## Service architecture

{{#include architecture.svg}}

> Click each component in this diagram to get to the **hash** of the source code **currently running in production**.

The metadata defining the service for App-Interface is kept upstream and open as templates for both the [osbuild-composer](https://github.com/osbuild/osbuild-composer/blob/main/templates/composer.yml) and [image-builder components](https://github.com/osbuild/image-builder/blob/main/templates/image-builder.yml).
The tooling to operate the service is to large parts open source and publicly accessible, e.g. qontract in the form of [qontract-server](https://github.com/app-sre/qontract-server), [qontract-reconcile](https://github.com/app-sre/qontract-reconcile).
The architecture documents in this section comply with the AppSRE contract.

## How to contribute

Our [developer guide](https://www.osbuild.org/guides/developer-guide/developer-guide.html) is a great starting point to learn about our workflow, code style and more!

If you want to contribute to our frontend or backend, here are guides on how to get the respective stack set up for development:
 * [image-builder-frontend](https://github.com/RedHatInsights/image-builder-frontend#frontend-development)
 * [image-builder-backend](https://github.com/RedHatInsights/image-builder-frontend/blob/main/devel/README.md)

### How to reach out to us

* Matrix: `#image-builder` on fedoraproject.org
* Mailing List: [image-builder@redhat.com](mailto:image-builder@redhat.com)
* Issues and pull requests: [github.com/osbuild](https://github.com/osbuild)