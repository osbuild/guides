# osbuild-mpp

`osbuild-mpp` is a manifest preprocessor. It allows you to write a smaller [manifest](../manifests/index.md) that you can then expand into a full manifest as understood by `osbuild`. 

## Installation

`osbuild-mpp` is packaged as part of the `osbuild-tools` package for Fedora, RHEL, and CentOS. You can also run it directly from your source checkout where it is located at `/tools/osbuild-mpp`.

## Usage

```
€ osbuild-mpp [SRC] [DST]
```

### Dependency Solving

One of the core things to do with `osbuild-mpp` is to solve dependencies. Manifests must contain the list of all RPMs to be installed with their content hashes and URLs to get them from. This is an especially cumbersome list to write by hand. `osbuild-mpp` can generate the relevant [sources](../modules/sources.md) list for you and the RPM stage to install these.

See for example the following for the RPM stage in a [pipeline](../modules/pipelines.md):

```json
{
	"name": "org.osbuild.rpm",
	"options": {
		"gpgkeys": ["..."],
		"mpp-depsolve": {
			"architecture": "$arch",
			"module-platform-id": "$releasever",
			"releasever": "$releasever",
			"repos": [
				{
					"id": "default",
					"baseurl": "..."
				}
			],
			"packages": [
			  "@cloud-server-environment",
			  "chrony",
			  "dracut-config-generic",
			  "grub2-pc",
			  "kernel-core",
			  "langpacks-en",
			  "polkit",
			  "selinux-policy-targeted",
			  "systemd-udev"
			]
		}
	}
}
```

When this is fed through `osbuild-mpp` the resulting manifest will have all RPMs and dependencies resolved at the versions as they currently exist on the selected repositories.

### Variables

You can define variables in `osbuild-mpp` that can later be used in the rest of the manifest. Variables are defined in a `mpp-vars` object that is at the root of the manifest. A common usecase is to refer to the architecture of a system. 

```
€ jq '."mpp-vars"' samples/fedora-boot.mpp.json
{
  "arch": "x86_64",
  "release": 34,
  "releasever": "f$release",
  "snapshot": "20210512"
}
```

```
€ jq '.pipeline.stages[1].options."mpp-depsolve".architecture' samples/fedora-boot.mpp.json
"$arch"
```

`osbuild-mpp` will replace any occurences of `$name` with the `mpp-vars` defined `name` value.

You can also access variables by using an `mpp-format-string` object. This object will be replaced by the value it generates. 

```
€ rg 'mpp-format-string' samples/fedora-boot.mpp.json
14: "mpp-format-string": "org.osbuild.fedora{release}"
81: "mpp-format-string": "{'f'*32}-{rpms['stages']['kernel-core'].evra}"
```

`mpp-format-string` allows you write a [Python f-string](https://peps.python.org/pep-0498/) that will be replaced with its value. This offers more possibilities than using the `$name` syntax. With `mpp-format-string` you can also refer to internal data structures for the manifest as shown in the second line of the output.


## Examples

You can take a look at the examples in your source checkout at `/samples/*.mpp.json` or look at the [examples on github](https://github.com/osbuild/osbuild/tree/main/test/data/manifests). Files that have `*.mpp.json` are meant to be preprocessed with `osbuild-mpp` to generate a full manifest.
