# Image Builder Workers Architecture Document

## Service Description
The `workers` are a fleet of (for now) amazon EC2 instances, responsible for requesting pending jobs
from the composer-worker API in AOC, perform jobs as instructed and report back the results. The
kinds of jobs are:
 - determining build instructions for future image builds
 - building images
 - uploading images to their destination
 - registering images in their target platform

Workers are stateless, apart from their caches, and hence trivially restartable. To build images
workers need to be running in a VM with kernel access, rather than in a container, and in order
to upload the results the workers need the right credentials for each of the possible targets. In
order to request new jobs, the workers need to be issued with RH credentials.

## Technology Stack
The service is written in Golang, and the list of vendored dependencies can be found in
[go.mod](https://github.com/osbuild/osbuild-composer/blob/main/go.mod). The underlying tool is written
in python3.

Both the service and underlying tool are built as RPMs and installed into AMIs. Their dependencies are
specified in their respective .spec files:
 - https://github.com/osbuild/osbuild/blob/main/osbuild.spec
 - https://github.com/osbuild/osbuild-composer/blob/main/osbuild-composer.spec

## Components
The service consists of a fleet of workers. If no workers are available, no jobs will be built until
workers are again available. Nothing is lost as jobs will stay in the queue, but everything will
simply stall.

## Routes
The workers expose no routes.

## Dependencies
The workers have the following internal and external dependencies.

### Internal
 - Red Hat SSO for authentication. Without this, the worker cannot request new jobs.

### External
 - EC2. Without this the workers cannot run.
 - EC2, GCP and Azure to upload the respective images. Without this image upload will fail.
 - S3 to upload images for download by the user. Without this image upload will fail.
 - Packer as a build tool. Without this, the service cannot be redeployed.
 - TerraForm as a deployment orchestrator. Without this, the service cannot be redeployed.
 - Github as an upstream repository. Without this, the service cannot be redeployed.
 - Gitlab, AWS EC2, and Openstack for upstream testing. Without this changes to the service cannot land.

## Service Diagram
See parent page.

## Application Success Criteria
The worker fleet is successful if:
1. It scales on demand to avoid pending jobs having overly long queue times.
2. The jobs are executed in a timely fashion.
3. The job error rate (including image builds and uploads) is low.

## State
Workers only have ephemeral state.

To optimize build-times, workers will keep a cache of previously (partially) built or downloaded
artifacts if this is lost it will be recreated on demand with no other loss than extra running time.

## Load Testing
Image Builder is currently being load tested on a weekly basis with failure thresholds reflecting the
SLIs. The load tests happen against stage CRC. An example can be found
[here](https://gitlab.com/osbuild/ci/image-builder/-/jobs/1541382293).

The load testing happens against stage, and tests the entire stack, including the workers.

## Capacity
Increasing the rate at which workers can handle jobs is easily done by scaling up the ASG.

The workers are also limited by a 2 week image retention period in our cloud accounts. For GCP
images this means there's can have a maximum of 1000 images stored at any given time. For AWS it's
limited by the snapshots per region limit (100k). And it's limited by the amount of images that can
concurrently be imported (20). The latter might pose a problem in future.
