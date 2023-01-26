## Current architecture

This diagram shows the overall architecture, and each sub-section goes into details about the three main
components, which are run as independent services.

{{#include architecture.svg}}

The metadata defining the service for App-Interface is kept upstream and open as templates for both the [osbuild-composer](https://github.com/osbuild/osbuild-composer/blob/main/templates/composer.yml) and [image-builder components](https://github.com/osbuild/image-builder/blob/main/templates/image-builder.yml).
The tooling to operate the service is to large parts open source and publicly accessible, e.g. qontract in the form of [qontract-server](https://github.com/app-sre/qontract-server), [qontract-reconcile](https://github.com/app-sre/qontract-reconcile).
The architecture documents in this section comply with the AppSRE contract.
