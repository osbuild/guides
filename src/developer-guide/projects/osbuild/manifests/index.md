# Manifests

`osbuild`'s main input is manifests. Manifests are JSON documents that represent all the actions to be performed to build an Operating System artifact. On this page we will take an example manifest and go through what's in it.

The manifest used on this page is `samples/fedora-container.json`. This is a version 2 manifest (more on that later).

## Example Manifest

In this example manifest repeating elements have been snipped out, comments have also been added which makes this manifest now invalid JSON but easier to follow along.

```json
// A manifest contains a single JSON object
{
  // The version property specifies the version of the manifest to use. Generally
  // you want to use version 2. Leaving out the version property defaults the
  // manifest to version 1.
  "version": "2",

  // Pipelines are logical groupings of stages to produce a filesystem tree or to
  // transform a filesystem tree into something else. Pipelines can depend on other
  // pipelines.
  "pipelines": [
    {
      // This pipeline will be execute in a buildroot produced by a runner.
      "runner": "org.osbuild.fedora34",

      // Pipeline name.
      "name": "build",

      // Stages are single units of logic that change something in the pipeline
      // tree.
      "stages": [
        {
          // This is the rpm stage, it installs RPM files into the filesystem tree.
          "type": "org.osbuild.rpm",

          // Inputs are things made available to the stage that come from outside the
          // filesystem tree.
          "inputs": {
			// Name of the input
            "packages": {
			  // Input type, in this case a bunch of files.
              "type": "org.osbuild.files",
              // These files have been produced by the source section of the manifest.
              "origin": "org.osbuild.source",
              // An the files are referenced by the hash of their content. This indirection
		      // ensures that the source can cache the files for multiple builds.
              "references": {
                "sha256:c540ca8c5e21ba5f063286c94a088af2aac0b15bc40df6fd562d40154c10f4a1": {},
                "sha256:15a654d32efaa75b5df3e2481939d0393fe1746696cc858ca094ccf8b76073cd": {},
				// ... rest left out
              }
            }
          },
          // Each stage takes its own options, these are the options for the rpm stage.
          "options": {
            "gpgkeys": [
              "-----BEGIN PGP PUBLIC KEY BLOCK-----\n\nmQINBF1RVqsBEADWMBqYv/G1r4PwyiPQCfg5fXFGXV1FCZ32qMi9gLUTv1CX7rYy\nH4Inj93oic+lt1kQ0kQCkINOwQczOkm6XDkEekmMrHknJpFLwrTK4AS28bYF2RjL\nM+QJ/dGXDMPYsP0tkLvoxaHr9WTRq89A+AmONcUAQIMJg3JxXAAafBi2UszUUEPI\nU35MyufFt2ePd1k/6hVAO8S2VT72TxXSY7Ha4X2J0pGzbqQ6Dq3AVzogsnoIi09A\n7fYutYZPVVAEGRUqavl0th8LyuZShASZ38CdAHBMvWV4bVZghd/wDV5ev3LXUE0o\nitLAqNSeiDJ3grKWN6v0qdU0l3Ya60sugABd3xaE+ROe8kDCy3WmAaO51Q880ZA2\niXOTJFObqkBTP9j9+ZeQ+KNE8SBoiH1EybKtBU8HmygZvu8ZC1TKUyL5gwGUJt8v\nergy5Bw3Q7av520sNGD3cIWr4fBAVYwdBoZT8RcsnU1PP67NmOGFcwSFJ/LpiOMC\npZ1IBvjOC7KyKEZY2/63kjW73mB7OHOd18BHtGVkA3QAdVlcSule/z68VOAy6bih\nE6mdxP28D4INsts8w6yr4G+3aEIN8u0qRQq66Ri5mOXTyle+ONudtfGg3U9lgicg\nz6oVk17RT0jV9uL6K41sGZ1sH/6yTXQKagdAYr3w1ix2L46JgzC+/+6SSwARAQAB\ntDFGZWRvcmEgKDMyKSA8ZmVkb3JhLTMyLXByaW1hcnlAZmVkb3JhcHJvamVjdC5v\ncmc+iQI4BBMBAgAiBQJdUVarAhsPBgsJCAcDAgYVCAIJCgsEFgIDAQIeAQIXgAAK\nCRBsEwJtEslE0LdAD/wKdAMtfzr7O2y06/sOPnrb3D39Y2DXbB8y0iEmRdBL29Bq\n5btxwmAka7JZRJVFxPsOVqZ6KARjS0/oCBmJc0jCRANFCtM4UjVHTSsxrJfuPkel\nvrlNE9tcR6OCRpuj/PZgUa39iifF/FTUfDgh4Q91xiQoLqfBxOJzravQHoK9VzrM\nNTOu6J6l4zeGzY/ocj6DpT+5fdUO/3HgGFNiNYPC6GVzeiA3AAVR0sCyGENuqqdg\nwUxV3BIht05M5Wcdvxg1U9x5I3yjkLQw+idvX4pevTiCh9/0u+4g80cT/21Cxsdx\n7+DVHaewXbF87QQIcOAing0S5QE67r2uPVxmWy/56TKUqDoyP8SNsV62lT2jutsj\nLevNxUky011g5w3bc61UeaeKrrurFdRs+RwBVkXmtqm/i6g0ZTWZyWGO6gJd+HWA\nqY1NYiq4+cMvNLatmA2sOoCsRNmE9q6jM/ESVgaH8hSp8GcLuzt9/r4PZZGl5CvU\neldOiD221u8rzuHmLs4dsgwJJ9pgLT0cUAsOpbMPI0JpGIPQ2SG6yK7LmO6HFOxb\nAkz7IGUt0gy1MzPTyBvnB+WgD1I+IQXXsJbhP5+d+d3mOnqsd6oDM/grKBzrhoUe\noNadc9uzjqKlOrmrdIR3Bz38SSiWlde5fu6xPqJdmGZRNjXtcyJlbSPVDIloxw==\n=QWRO\n-----END PGP PUBLIC KEY BLOCK-----\n"
            ],
            "exclude": {
              "docs": true
            }
          }
        },
        {
          "type": "org.osbuild.selinux",
          "options": {
            "file_contexts": "etc/selinux/targeted/contexts/files/file_contexts",
            "labels": {
              "/usr/bin/cp": "system_u:object_r:install_exec_t:s0",
              "/usr/bin/tar": "system_u:object_r:install_exec_t:s0"
            }
          }
        }
      ]
    },
    {
      // Name of the pipeline.
      "name": "tree",
      // This pipeline will use the tree created by the `build` pipeline (the previous one in this
      // example) as its buildroot.
      "build": "name:build",
      "stages": [
        {
          "type": "org.osbuild.rpm",
          "inputs": {
            "packages": {
              "type": "org.osbuild.files",
              "origin": "org.osbuild.source",
              "references": {
                "sha256:c540ca8c5e21ba5f063286c94a088af2aac0b15bc40df6fd562d40154c10f4a1": {},
                "sha256:15a654d32efaa75b5df3e2481939d0393fe1746696cc858ca094ccf8b76073cd": {},
				// ... rest left out
              }
            }
          },
          "options": {
            "gpgkeys": [
              "-----BEGIN PGP PUBLIC KEY BLOCK-----\n\nmQINBF1RVqsBEADWMBqYv/G1r4PwyiPQCfg5fXFGXV1FCZ32qMi9gLUTv1CX7rYy\nH4Inj93oic+lt1kQ0kQCkINOwQczOkm6XDkEekmMrHknJpFLwrTK4AS28bYF2RjL\nM+QJ/dGXDMPYsP0tkLvoxaHr9WTRq89A+AmONcUAQIMJg3JxXAAafBi2UszUUEPI\nU35MyufFt2ePd1k/6hVAO8S2VT72TxXSY7Ha4X2J0pGzbqQ6Dq3AVzogsnoIi09A\n7fYutYZPVVAEGRUqavl0th8LyuZShASZ38CdAHBMvWV4bVZghd/wDV5ev3LXUE0o\nitLAqNSeiDJ3grKWN6v0qdU0l3Ya60sugABd3xaE+ROe8kDCy3WmAaO51Q880ZA2\niXOTJFObqkBTP9j9+ZeQ+KNE8SBoiH1EybKtBU8HmygZvu8ZC1TKUyL5gwGUJt8v\nergy5Bw3Q7av520sNGD3cIWr4fBAVYwdBoZT8RcsnU1PP67NmOGFcwSFJ/LpiOMC\npZ1IBvjOC7KyKEZY2/63kjW73mB7OHOd18BHtGVkA3QAdVlcSule/z68VOAy6bih\nE6mdxP28D4INsts8w6yr4G+3aEIN8u0qRQq66Ri5mOXTyle+ONudtfGg3U9lgicg\nz6oVk17RT0jV9uL6K41sGZ1sH/6yTXQKagdAYr3w1ix2L46JgzC+/+6SSwARAQAB\ntDFGZWRvcmEgKDMyKSA8ZmVkb3JhLTMyLXByaW1hcnlAZmVkb3JhcHJvamVjdC5v\ncmc+iQI4BBMBAgAiBQJdUVarAhsPBgsJCAcDAgYVCAIJCgsEFgIDAQIeAQIXgAAK\nCRBsEwJtEslE0LdAD/wKdAMtfzr7O2y06/sOPnrb3D39Y2DXbB8y0iEmRdBL29Bq\n5btxwmAka7JZRJVFxPsOVqZ6KARjS0/oCBmJc0jCRANFCtM4UjVHTSsxrJfuPkel\nvrlNE9tcR6OCRpuj/PZgUa39iifF/FTUfDgh4Q91xiQoLqfBxOJzravQHoK9VzrM\nNTOu6J6l4zeGzY/ocj6DpT+5fdUO/3HgGFNiNYPC6GVzeiA3AAVR0sCyGENuqqdg\nwUxV3BIht05M5Wcdvxg1U9x5I3yjkLQw+idvX4pevTiCh9/0u+4g80cT/21Cxsdx\n7+DVHaewXbF87QQIcOAing0S5QE67r2uPVxmWy/56TKUqDoyP8SNsV62lT2jutsj\nLevNxUky011g5w3bc61UeaeKrrurFdRs+RwBVkXmtqm/i6g0ZTWZyWGO6gJd+HWA\nqY1NYiq4+cMvNLatmA2sOoCsRNmE9q6jM/ESVgaH8hSp8GcLuzt9/r4PZZGl5CvU\neldOiD221u8rzuHmLs4dsgwJJ9pgLT0cUAsOpbMPI0JpGIPQ2SG6yK7LmO6HFOxb\nAkz7IGUt0gy1MzPTyBvnB+WgD1I+IQXXsJbhP5+d+d3mOnqsd6oDM/grKBzrhoUe\noNadc9uzjqKlOrmrdIR3Bz38SSiWlde5fu6xPqJdmGZRNjXtcyJlbSPVDIloxw==\n=QWRO\n-----END PGP PUBLIC KEY BLOCK-----\n"
            ],
            "exclude": {
              "docs": true
            },
            "install_langs": [
              "en_US"
            ]
          }
        },
        {
          "type": "org.osbuild.rpm.macros",
          "options": {
            "filename": "/etc/rpm/macros.image-language-conf",
            "macros": {
              "_install_langs": [
                "en_US"
              ]
            }
          }
        },
        {
          "type": "org.osbuild.locale",
          "options": {
            "language": "en_US.UTF-8"
          }
        }
      ]
    },
    {
      "name": "container",
      "build": "name:build",
      "stages": [
        {
          "type": "org.osbuild.oci-archive",
          "inputs": {
            "base": {
              "type": "org.osbuild.tree",
              "origin": "org.osbuild.pipeline",
              "references": [
                "name:tree"
              ]
            }
          },
          "options": {
            "architecture": "amd64",
            "filename": "fedora-container.tar",
            "config": {
              "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
              ],
              "ExposedPorts": [
                "80"
              ]
            }
          }
        }
      ]
    }
  ],
  "sources": {
    "org.osbuild.curl": {
      "items": {
        "sha256:c540ca8c5e21ba5f063286c94a088af2aac0b15bc40df6fd562d40154c10f4a1": "https://rpmrepo.osbuild.org/v2/mirror/public/f34/f34-x86_64-fedora-20210512/Packages/a/acl-2.3.1-1.fc34.x86_64.rpm",
        "sha256:15a654d32efaa75b5df3e2481939d0393fe1746696cc858ca094ccf8b76073cd": "https://rpmrepo.osbuild.org/v2/mirror/public/f34/f34-x86_64-fedora-20210512/Packages/a/alternatives-1.15-2.fc34.x86_64.rpm",
		// ... rest left out
      }
    }
  }
}

```
