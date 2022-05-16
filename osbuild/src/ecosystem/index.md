# Ecosystem

The `osbuild` projects contains several parts, each responsible for their own
things.

## osbuild

At the core of the project is the [osbuild](https://github.com/osbuild/osbuild)
project. This provides the build pipelines.

## osbuild-composer

[osbuild-composer](https://github.com/osbuild/osbuild-composer) is the higher
level translation layer between front end tools and osbuild. It provides APIs
that can be used by web-frontends, cli-tools, and provides osbuild workers
that perform the build pipelines.

## weldr-client

## cockpit-composer

## image-builder

This is the console.redhat.com HTTP API that speaks to `osbuild-composer` and
translates it for the `image-builder-frontend`.

## image-builder-frontend

The frontend for console.redhat.com, it gets data from `image-builder`.
