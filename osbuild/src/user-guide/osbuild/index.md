# osbuild

The `osbuild` program is the core program that builds image files. It takes a manifest as an input and uses it to determine how to build the image and how to output it.

`osbuild` is often not used directly (though certainly possible to do so) but instead used through `osbuild-composer` which provides amongst others the APIs for higher level tooling such as the [weldr-client](/user-guide/weldr-client/index.md) command line interface or the [cockpit-composer](/user-guide/cockpit-composer/index.md) web interface.
