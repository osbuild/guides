# Concepts

`osbuild` has some terms that you'll encounter a lot while using it. Here's a high level overview of those.

## Assemblers

## Manifest

The manifest files are JSON documents that `osbuild` uses as its input to generate images. These files describe everything you want in your image and how to output the image.

## Pipelines

Manifest files are built up of pipelines, each pipeline is an exportable object and consists of a group of stages followed by an assembler.

## Stages

Inside pipelines are stages. Stages are the smallest unit of work `osbuild` performs. The contain, for example:

1. installing packages
2. adding users
3. enabling services

A stage operates on a read-write filesystem tree and modifies it.
