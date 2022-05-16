# Architecture

`osbuild` builds images based on manifest files. It builds these images in a [bubblewrap](https://github.com/containers/bubblewrap) sandbox.

To communicate between the host system and processes running in the sandbox an `AF_UNIX` socket is used.
