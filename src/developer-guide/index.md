# Developer Guide

In this section, you will find a description of the source code in `osbuild` organization.

The following scheme describes how separate components communicate with each other:
![](osbuild-composer.svg)

In the very basic use case where `osbuild-composer` is running locally, the "pool of workers" also lives on the user's host machine. The `osbuild-composer` and `osbuild-worker` processes are spawned by systemd. We don't support any other means of spawning these processes, as they rely on systemd to open sockets, create state directories etc. Additionally, `osbuild-worker` spawns osbuild as a subprocess to create the image itself. The whole image building machinery is spawned from a user process, for example, `composer-cli`.
