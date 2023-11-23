# Deprecation

`osbuild` gives the guarantee that a given manifest will produce the same image as long as any external sources remain available in any future versions of `osbuild`.

## Modules 

As `osbuild` guarantees that the same manifest will produce the same image modules cannot change their schema or behavior after introduction. They can only be expanded upon.
