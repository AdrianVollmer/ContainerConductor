# Container Conductor

A single shell script for running isolated tools based on mise.

I've been taught to never run untrusted programs. Programs I run must be
reviewed either by me or someone I trust. Well, times have changed, and
there are lots of valuable programs either in binary form or installable
via pip, cargo, npm, go, ruby, etc. for which neither I nor the
maintainers of my preferred Linux distribution have the bandwidth to
review the code in a timely manner.

We could simply pass on lots of these genuinely useful tools, or we can
run them anyway. To mitigate supply chain attacks, I'd like to limit the
blast radius. The tool of my choice to achieve this is podman.
[`mise`](https://mise.jdx.dev/) already handles the management side of
things, podman will handle the isolation.

Container Conductor, or coco in short, is a bash script with minimal
dependencies that enables the user to run all tools of a vast universe
of software repositories with just enough permissions.

## Dependencies

- bash
- podman or docker
- jq

# Installation

Copy or symlink the `coco` executable to `~/.local/bin` or somewhere
else in your path.

## Configuration

Each tool must be configured in `$XDG_CONFIG_HOME/coco/config.json`.

To manage the tools, call `coco` directly. To run tools via coco, create
a symlink with the tools name to the coco executable in
`$HOME/.local/bin`.

`coco up` will create a symlink for each tool to `coco`.

## License

MIT
