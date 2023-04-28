# Third-party Repositories

`osbuild-composer` supports adding packages from third-party repositories and saving the repository customizations
to an image. This guide aims to clarify each usecase and how to configure `osbuild-composer` and
the blueprints accordingly.

Very importantly, `osbuild-composer` has two distinct definitions of third-party repositories. Firstly, **payload repositories** which can be used to install third-party packages at build time and,
secondly, **custom repositories** which are used to persist the repository configurations to the image.

This could lead to the following desired usecases:
1. Install a third-party package
2. Save the third-party repository configurations to the image
3. Install a third-party package **and** save the configurations

## 1. Install a third-party package
To install a third-party package at build time, it is necessary to enable the required third-party repository as a payload repository. This will not save any of the repository configurations
to the image and will not make the repositories available to users on the system after the image has been built. For further information on how to install and configure `osbuild-composer`
to use custom repositories for installing third-party packages, continue reading [here](./managing-repositories.md).


## 2. Save repository configurations
In the second scenario, to make third-party repository configurations persistent and make the repositories available to users on the system, one would use the blueprint `custom repository`
configurations to enable this. The repository will be configured and saved to `/etc/yum.repos.d` as a `.repo` file. GPG keys are not imported at build time, but are imported when first 
installing a third-party package from the desired repository. You can find the blueprint reference for custom repositories [here](../blueprint-reference/blueprint-reference.md).

## 3. Install a third-party package and save configurations
In this case it is necessary to use a combination of payload repositories and custom repositories in order to achieve the desired outcome. This will ensure that the package is installed during 
build time and the repository configuration is saved to disk for future use. If the user only needs the package or the configuration file, they can use the appropriate repository type to achieve
their goal.
