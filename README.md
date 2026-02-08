# Container Conductor

Sandboxed execution for untrusted dev tools in one script.

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

**Dependencies**:

- bash
- podman or docker
- jq

## Usage

Run `coco help` to learn more:

``` console
$ coco help
Usage: coco <command> [args...]

    Commands:
      run <tool> [args...]   Run a tool in a container
      list                   List all configured tools
      up                     Create symlinks for all tools
      down                   Remove symlinks for all tools
      prune                  Remove mise cache volumes
      rebuild                Rebuild the container image

    Symlink invocation:
      <tool> [args...]       Run via symlink (after 'up')

    Environment:
      COCO_RUNTIME        Container runtime: podman (default) or docker
      COCO_IMAGE          Container image (default: localhost/coco:latest)
      COCO_DEBUG          Enable debug output (set to 'true' or '1')
      COCO_ALLOW_NETWORK  Override network restrictions (set to 'true' or '1')
      COCO_EXTRA_PORTS    Additional ports to publish (CSV, e.g. '8080:8080,3000:3000')
```

## Configuration

Each tool must be configured in `$HOME/.config/coco/config.json` (or
`$XDG_CONFIG_HOME`). You can also split configs across multiple files
in `config.d/*.json`.

To manage the tools, call `coco` directly. To run tools via `coco`,
create a symlink with the tool's name to the `coco` executable in
`$HOME/.local/bin`.

`coco up` will create a symlink for each tool to `coco`.

**Config options:**

| Option | Description |
|--------|-------------|
| `tool` | mise tool specifier (e.g., `npm:eslint`, `cargo:ripgrep`) |
| `command` | Command to run (defaults to tool name) |
| `mounts` | Array of `{src, dst, opts}` bind mounts |
| `env` | Environment variables to set |
| `env_passthrough` | Host env vars to pass through |
| `workdir` | Working directory in container |
| `network` | Network mode (`none` to disable) |
| `allowed_hosts` | DNS whitelist (enables network with restrictions) |
| `isolated` | Use separate volumes for this tool (better sandboxing) |
| `ports` | Ports to publish |
| `podman_args` | Extra arguments to pass to podman/docker |

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
  }
}
```

Or let's take [Claude Code](https://code.claude.com/docs/en/overview).
We mount the current working directory as well as the config directory
and the data directory (we can even fix the missed opportunity of
adhering to the XDG Desktop Standard):

``` json
{
  "claude": {
    "tool": "npm:@anthropic-ai/claude-code",
    "command": "claude",
    "mounts": [
      {
        "src": ".",
        "dst": "/workspace"
      },
      {
        "src": "~/.local/share/claude",
        "dst": "/root/.claude"
      },
      {
        "src": "~/.config/claude",
        "dst": "/root/.config/claude"
      }
    ],
    "env_passthrough": [
      "TERM",
      "ANTHROPIC_API_KEY"
    ],
    "workdir": "/workspace"
  }
}
```

Now tell it to use `mise` for running all kinds of dev tools in your
`CLAUDE.md`.

See `config.sample.json` for more examples.

## Security

Coco raises the bar for supply chain attacks, so a malicious dependency
can't silently exfiltrate your SSH keys or AWS credentials. But it's not
a perfect sandbox.

**Known limitations:**

- Shared volumes (like `~/.local/bin`) let one tool modify another
  tool's executables, potentially gaining its permissions.
- `allowed_hosts` filters DNS only. A determined attacker could use
  hardcoded IPs or alternative resolution methods.

## License

MIT
