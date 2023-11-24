# Contributing

`osbuild` is written in Python and supports Python version `>= 3.6`.

## Modules

### Stages

When contributing a new stage to `osbuild` it's important to keep in mind our deprecation policies. For new stages to be contributed we require unit tests and integration tests to be included in your pull request.

#### Testing

##### Unit Tests

Stage unit tests are tests that validate the stages correct handling of (in)valid schemas. The unit tests mock out everything where the stage might want to touch the system.

You can find the unit tests in the `stages/test` [directory](https://github.com/osbuild/osbuild/tree/main/stages/test) from the repository root.

##### Integration Tests

When testing a stage an integration test will build both a base image and that same image with the stage under test applied. Both images are then diffed against eachother.

Stage integration tests will define the files that are allowed to change and their new contents and will fail if anything has changed between the two images that isn't whitelisted.

You can find the stage integration test manifests in the repository root under the `test/data/stages` [directory](https://github.com/osbuild/osbuild/blob/main/test/data/stages/). We generally write an `a.mpp.yaml` which is the state before the stage, then copy it over into `b.mpp.yaml` where the stage is applied. Differences are listed in `diff.json`.

The `.mpp.yaml` files are processed using `osbuild-mpp` into JSON manifests, you can use `make test-data` to reprocess all `.mpp.yaml` files in the integration tests.

The [README.md](https://github.com/osbuild/osbuild/blob/main/test/data/stages/README.md) for the stage integration tests provides detail on how to set up your manifests.
