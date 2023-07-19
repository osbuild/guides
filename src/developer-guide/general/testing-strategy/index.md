# Testing strategy

Let me start with a quote:

> As the team obsessed with immutable test dependencies, how could we use ..

*One osbuild developer in one PR fixing one more piece of infrastructure which could still change.*

TODO: what do we test in each repo

## osbuild-composer

This section provides a basic summary of the various types of testing done for `osbuild-composer`. Detailed information about testing can be found in [the upstream repository][tests_readme].

[tests_readme]: https://github.com/osbuild/osbuild-composer/blob/main/test/README.md

### Unit tests

There is pretty heavy mocking in the osbuild-composer codebase.

HTTP API is unit-tested without any network communication (there is no socket), only the HTTP request/responses are tested.

### Integration tests

These test cases live under `test/cases` and each of them is a standalone script. Some of them invoke additional binaries which live under `cmd` if not specified otherwise.

1. `api.sh [aws|azure|gcp]` - test the Cloud API (running at localhost:443)
   
   * Provisions osbuild-composer and locally running remote worker.
   
   * Creates a request for compose and uploads the image to specified cloud provider. Currently AWS, Azure and GCP are supported.

   * The uploaded image is used for a VM instance in the respective cloud environment, booted and connected to via SSH. This is currently tested only for AWS and GCP.
   
   * **Requires credentials for the respective cloud provider** to work properly.

2. `aws.sh`
   
   Use osbuild-composer "the way we expect our customers to use it". That means provision `osbuild-composer` and use Weldr API to build an AMI image and upload it to EC2. Then use the `aws` CLI tool to spawn a VM from the image and make sure it boots and can be accessed.
   
   * **Requires AWS credentials**

3. `base_tests.sh`
   
   This script runs binaries implemented as part of osbuild-composer codebase in golang. It provisions osbuild-composer and then runs the tests in a loop.
   
   1. `osbuild-composer-cli-tests` - Weldr API tests using composer-cli
      
      * Executing `composer-cli` utility
      * Invoke multiple image builds
   
   2. `osbuild-weldr-tests` - Weldr API tests using golang library from `internal/client`
      
      * These live directly in the `internal` directory, which is a bit odd given that all other tests live under `cmd/`, but there might be a reason for this.
      * They invoke a build of a qcow2 image
   
   3. `osbuild-dnf-json-tests` - These make sure the interface to dnf still works
      
      * This binary will execute `dnf-json` multiple times and it will also run multiple `dnf` depsolving tasks in parallel. It is possible that it will require a high amount of RAM.
      
      * *My guess would be at least 2GB memory for a VM running this test.*
   
   4. `osbuild-auth-tests` - Make sure the TLS certificate authentication works as expected for the koji api and worker api sockets.
      
      * A certificate authority is created for these tests and the files are stored in `/etc/osbuild-composer-test/ca`
      * The certificates live in the standard configuration directory: `/etc/osbuild-composer`
      * Multiple certificates are created:
        * For osbuild-composer itself (let's say a "server" certificate)
        * For osbuild-worker
        * For a client application, in this case the test binary
        * For kojihub

4. `image_tests.sh`
   
   Possibly the most resource-hungry test case. It builds an image for all supported image types for all supported distributions on all supported architectures *(note that every distro has a different set of aches and arches have different set of supported types, e.g. there is no s390x image for AWS because there is no such machine)*.
   The "test cases" are defined in `test/cases/manifests` and they contain a boot type (where to spawn the VM), compose request (what to ask Weldr API for), and finally the expected manifest. Osbuild-composer should generate the same manifest, build the image successfully, optionally upload it to a cloud provider, boot the image, and finally verify it is running.
   
   * **Require AWS, Openstack, and Azure credentials**

5. `koji.sh`
   
   Runs a koji instance in a container. It sets up certificates and Kerberos KDC because osbuild-composer uses Kerberos to authenticate with Koji.

6. `ostree.sh`
   
   This test case creates an OSTree commit, boots it, then it creates a commit with an upgrade on top of the previous commit and makes sure the VM can upgrade to the new one.
   
   * Uses libvirt to run the VM

7. `qemu.sh`
   
   Create a qcow2 image and boot it using libvirt.

### Leaking resources

The cloud-cleaner binary was created to clean up all artifacts (like images, but also registered AMIs, security groups, etc.) that could be left behind. Not all executables in our CI have proper error handling and clean up code and what is even worse, if Jenkins fails and takes down all running jobs, it is possible that the clean-up code will not run even if it is implemented.

**Possibly leaking resources:**

1. `api.sh` test case:
   
   * Image uploaded to AWS, Azure or GCP

2. `aws.sh` test case:
   
   * Image uploaded to EC2
   
   * VM running in EC2

### RPM Repository Snapshots

In order to provide a stable base for the tests, the maintainer team created [the RPMRepo project](../../projects/rpmrepo/index.md) that periodically snapshots repositories of selected distributions.
