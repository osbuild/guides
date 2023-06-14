# Image Builder CRC API Architecture Document

## Service Description
The `image-builder` API in CRC serves as the public API used either directly by customers or through the
CRC UI. Through this API customers can create, manage and view image builds. The service in CRC is
responsible for access management, quotas, rate-limiting, etc. In the future it may interact with other
services in CRC in order to add value to the image build experience.

The actual image build requests are passed on to `composer`, which is outside the scope of this document.

## Technology Stack
The service is written in Golang, and the list of dependencies can be found in
[go.mod](https://github.com/osbuild/image-builder/blob/main/go.mod).

The `ubi8/go-toolset:latest` container is used as a builder, and `ubi8/ubi-minimal:latest` to run the
binary. The container images are located here: https://quay.io/repository/cloudservices/image-builder.

## Components
The service consists of the image-builder app running in CRC, and its backing database. If either of
these are unavailable, the service does not work at all, new images cannot be built, and historical
builds cannot be introspected. Already built images that may be in used by customers are unaffected,
only their history and metadata can no longer be queried through the service.

## Routes
The public route is `/api/v1/image-builder`, a detailed list can be found at
[https://console.redhat.com/docs/api/image-builder](https://console.redhat.com/docs/api/image-builder).

## Dependencies
Image builder has the following internal and external dependencies.

### Internal
Image Builder relies on 3Scale to set the `x-rh-identity` header. It uses the header for authentication,
and quota application. It also uses the account number to map previously made compose requests to that
account number.

### External
 - AWS RDS for data storage. See the section on state.
 - Quay as a container registry. Without this, the service cannot be redeployed.
 - Github as an upstream repository. Without this, the service cannot be redeployed.
 - Gitlab, AWS EC2, and Openstack for upstream testing. Without this changes to the service cannot land.


## Service Diagram
See parent page.

## Application Success Criteria
1. Customers can queue image builds and view their state.
2. Customers can introspect and manage existing builds.
3. Quotas are applied according to policy to manage cost of running the service.
4. The service is able to provide functionality to make its own functionality discoverable
 - Enumerate supported features
 - Package search

## SLOs

The image builder API has the following SLOs, but we aim to add more and make these stricter as we
gain more experience from production. Our SLO targets are defined in App Interface.

### Latency

The ratio of requests that are considered significantly fast. The aim is to make it possible
to have a responsive UI. The exception is currently the `/compose` call, which is long-running and
our SLO targets reflect a higher latency threshold. The UI must be implemented with this 
in mind.

### Stability

The proportion of successful (or unsuccessful due to user error) `/compose` requests. The aim is for users
to be able to reliably queue image builds, even if some retries are required.

## State
The service depends on a PostgreSQL database, the default postgres12-rds-1 template is used. The
database stores metadata about each build, making it possible to enumerate past builds and to enforce
quota limits. If the state is lost historical data would be lost, but the user could still use their
images if they have saved the necessary information. The quote calculations would be off, but in the
worst case scenario customers would be able to build more images than they are meant to, which would
not be a big problem.

## Load Testing
Image Builder is currently being load tested on a weekly basis with failure thresholds reflecting the
SLIs. The load tests happen against stage CRC. An example can be found
[here](https://gitlab.com/osbuild/ci/image-builder/-/jobs/1541382293).

More information can be found [upstream](https://github.com/osbuild/image-builder/blob/main/test/README.md).

## Capacity
The needed capacity might grow a little bit in all directions (DB and number of pods), but any growth should
be slow. Currently  pods are running, and limits have been set on memory or cpu usage. The default insights
limits and quotas are used, which should be more than enough.
