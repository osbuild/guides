# Image Builder Composer API Architecture Document

## Service Description
The `image-builder-composer` API routed via [api.openshift.com](https://api.openshift.com/) serves as a
job queue for pending image builds as well as a metadata-store for already built images. When an image
build is queued via the API it is turned into a set of jobs that are put on the job queue, and together
do the necessary tasks to determine how the image should be built, build the image, upload the image to
its destination, and possibly register or import it to its final format.

The `image-builder-worker` API routed via [api.openshift.com](https://api.openshift.com/) serves as the
other side of the job queue, where jobs can be dequeued to be executed and their results posted.

The actual jobs are executed by `workers`, which is outside the scope of this document.

## Technology Stack
The service is written in Golang, and the list of dependencies can be found in
[go.mod](https://github.com/osbuild/osbuild-composer/blob/main/go.mod).

The `ubi8/go-toolset:latest` container is used as a builder, and `ubi8/ubi-minimal:latest` to run the
binary. The container images are located here: https://quay.io/repository/app-sre/composer.

## Components
The service consists of the composer and the composer-worker apps running in an AppSRE managed cluster,
and their backing database.

If either composer or the database are unavailable, the service does not work at all, new images
cannot be built, and historical builds cannot be introspected. Already built images that may be in
used by customers are unaffected, only their history and metadata can no longer be queried through
the service.

If composer-workers is unavailable, new jobs can be queued and old ones can be queried, but workers
will not be able to pick up new jobs until the API is back, and they will not be able to report back
results correctly for jobs they finish while the API is down.

## Routes
The public routes are `/api/image-builder-composer/v2/` and `/api/image-builder/worker/v1/`, detailed
lists can be found at https://api.openshift.com/api/image-builder-composer/v2/openapi and
https://api.openshift.com/api/image-builder-worker/v1/openapi.

## Dependencies
Composer has the following internal and external dependencies.

### Internal
Composer relies on Red Hat SSO for authentication.

### External
 - AWS RDS for data storage. See the section on state.
 - Quay as a container registry. Without this, the service cannot be redeployed.
 - Github as an upstream repository. Without this, the service cannot be redeployed.
 - Gitlab, AWS EC2, and Openstack for upstream testing. Without this changes to the service cannot land.


## Service Diagram
See parent page.

## Application Success Criteria
1. Image builds can be queued successfully
2. Jobs can be dequeued successfully and correctly
3. Jobs are tracked correctly
3. The state of historical or in-flight builds can be queried and introspected successfully

## State
The service depends on a PostgreSQL database, the default postgres12-rds-1 template is used. The
database stores metadata about each build, making it possible to enumerate past builds as well as
function as a job queue.

If the state is lost historical data would be lost, and pending image builds might never get scheduled,
but the user could still use their existing images if they have saved the necessary information. Data
loss would not affect the ability to schedule new builds.

## Load Testing

The Image Builder API in console.redhat.com is currently being load tested on a weekly basis with failure
thresholds reflecting the SLIs. The load tests happen against stage CRC, which is backed by composer in
api.stage.openshift.com. An example can be found [here](https://gitlab.com/osbuild/ci/image-builder/-/jobs/1723976612).

More information can be found [upstream](https://github.com/osbuild/image-builder/blob/main/test/README.md).

## Capacity

The defaults described in App Interface, 1 cpu 512Mi memory per container running in the default three pods are sufficient, and our expectations
are that this will remain sufficient for the next twelve months.
