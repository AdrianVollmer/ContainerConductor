# Container Conductor

A single bash script for running isolated tools based on mise.

## Motivation

Modern development relies on tools installed via pip, uv, cargo, npm,
go, and others, often pulling in dozens or hundreds of dependencies.
Neither you nor your distro maintainers can realistically audit all that
code.

The pragmatic choice is to run these tools anyway, but limit the blast
radius of a potential supply chain attack. Containers provide that
isolation. [`mise`](https://mise.jdx.dev/) handles tool versioning;
podman (or docker) handles the sandboxing.

Container Conductor (coco) ties them together: a short, readable bash
script that lets you run tools from the npm/pip/cargo universe with
minimal permissions. No network access, no home directory, just the
files they need.

## Installation

Copy or symlink the `coco` executable to `~/.local/bin` or somewhere
else in your search path.

*Dependencies*:

- bash
- podman or docker
- jq

## Configuration

Each tool must be configured in `$XDG_CONFIG_HOME/coco/config.json`.

To manage the tools, call `coco` directly. To run tools via coco, create
a symlink with the tools name to the coco executable in
`$HOME/.local/bin`.

`coco up` will create a symlink for each tool to `coco`.

## Example

Let's take [`eslint`](https://eslint.org/) as an example. As a static
code analyzer, it mostly needs access to the current working directory
and not much else.

``` json
{
  "eslint": {
    "tool": "npm:eslint",
    "command": "eslint",
    "mounts": [
      {
        "src": ".",
        "dst": "/workspace"
      }
    ],
    "env_passthrough": [
      "TERM",
      "NODE_ENV"
    ],
    "env": {
      "NPM_CONFIG_PREFIX": "/root/.npm-global"
    },
    "workdir": "/workspace",
    "network": "none"
  },
}
```

## License

MIT
