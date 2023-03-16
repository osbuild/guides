# Stages

[Pipelines](./pipelines.md) consist of stages. Each stage takes the current chroot as an input and transforms it. Stages always behave the same, given the same input chroot they will produce the same output. There is a large amount of stages available as they are the core building blocks of transforming the chroot into something that resembles an [operating system artifact](/terminology.md).

You can make things outside the chroot available to a stage through the use of [devices](./inputs.md), [mounts](./mounts.md), or [devices](./devices.md).

## Properties

| property  | value |
|-----------|-------|
| `type` | Stage type. |
| `options` | Options to pass to the stage, each stage defines their own schema for the options it takes. |
| `devices` | Optional: A list of [devices](./devices.md). |
| `mounts`  | Optional: A list of [mounts](./mounts.md). |
| `inputs`  | Optional: A list of [inputs](./inputs.md). |

## Example JSON

This uses the `org.osbuild.locale` stage to set the locale in a pipelines' tree.

```json
{
  "type": "org.osbuild.locale",
  "options": {
    "language": "en_US.UTF8"
  }
}
```

## Default Stages

`osbuild` comes with a large variety of stages, here's an overview of them and what they do or what they can be used for. This list is sorted alphabetically.

Note that stages provided by `osbuild` upstream are prefixed with `org.osbuild.` if you want to add your own stages you can read the [developer documentation on stage development](../developer-guide/development/stages.md).
