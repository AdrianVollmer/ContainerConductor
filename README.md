# Container Conductor

A single shell script for running isolated tools based on mise.

## Dependencies

- bash
- podman/docker
- jq

## Configuration

Each tool must be configured in `$XDG_CONFIG_HOME/coco/config.json`.

To manage the tools, call `coco` directly.

`coco up` will create a symlink for each tool to `coco`.

## License

MIT
