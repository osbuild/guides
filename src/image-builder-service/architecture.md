## Service architecture

This service is open source, so all of its code is inspectable and can be contributed to.

{{#include architecture.svg}}

> Click each component in this diagram to get to the **hash** of the source code **currently running in production**.

The metadata defining the service for App-Interface is kept upstream and open as templates for both the [osbuild-composer](https://github.com/osbuild/osbuild-composer/blob/main/templates/composer.yml) and [image-builder components](https://github.com/osbuild/image-builder/blob/main/templates/image-builder.yml).
The tooling to operate the service is to large parts open source and publicly accessible, e.g. qontract in the form of [qontract-server](https://github.com/app-sre/qontract-server), [qontract-reconcile](https://github.com/app-sre/qontract-reconcile).
The architecture documents in this section comply with the AppSRE contract.

* [Service API documentation](https://developers.redhat.com/api-catalog/api/image-builder)

## How to contribute

Our [developer guide](https://www.osbuild.org/guides/developer-guide/developer-guide.html) is a great starting point to learn about our workflow, code style and more!

If you want to contribute to our frontend or backend, here are guides on how to get the respective stack set up for development:
 * [image-builder-frontend](https://github.com/RedHatInsights/image-builder-frontend#frontend-development)
 * [image-builder-backend](https://github.com/RedHatInsights/image-builder-frontend/blob/main/devel/README.md)

### How to reach out to us

* Matrix: `#image-builder` on fedoraproject.org
* Mailing List: [image-builder@redhat.com](mailto:image-builder@redhat.com)
* Issues: [Service](https://issues.redhat.com/issues/?jql=project%20%3D%20HMS%20and%20component%20in%20(%22Image%20Builder%22)), [On premises](https://issues.redhat.com/issues/?jql=project%20%3D%20COMPOSER), [github.com/osbuild](https://github.com/osbuild)
* Pull requests: [github.com/osbuild](https://github.com/osbuild)

## How open is this service?

#### 🟢 Open assets
* 🟢 [The source code is open.](https://github.com/osbuild)
* 🟢 Unit tests are open.
* 🟢 Performance tests are open.
* 🟢 Functional tests are open.
* 🟢 The dependencies are open source.
* 🟢 Deployment metadata is open. [[1]](https://github.com/osbuild/osbuild-composer/blob/main/templates/composer.yml) [[2]](https://github.com/osbuild/image-builder/blob/main/templates/image-builder.yml)
#### 🟢 Contribution workflow
* 🟢 External contributors can follow the same workflow as team members.
* 🟢 [The workflow is publicly documented.](https://www.osbuild.org/guides/developer-guide/workflow.html)
* 🟢 Regular contributors can trigger CI.
* 🟢 External contributions are eagerly reviewed.
#### 🟠 Issue tracking and planning
* 🟢 The issue tracker is public. [[1]](https://github.com/osbuild) [[2]](https://issues.redhat.com/issues/?jql=project%20%3D%20COMPOSER%20or%20(project%20%3D%20HMS%20AND%20component%20in%20(%22Image%20Builder%22)))
* 🟠 The roadmap is public. [[1]](https://github.com/orgs/osbuild/projects)
#### 🟢 Documentation
* 🟢 [User documentation is public.](https://www.osbuild.org/guides/introduction.html)
* 🟢 [Developer documentation is public.](https://www.osbuild.org/guides/developer-guide/developer-guide.html)
#### 🟠 Communication
* 🟢 [There is a public mailinglist.](mailto:image-builder@redhat.com)
* 🔴 There are public meetings.
#### 🟠 Open Site Reliability Engineering
* 🟢 [There is an open status page.](https://status.redhat.com)
* 🔴 Logging, monitoring, and alerting is open.
* 🔴 Incident management is open.