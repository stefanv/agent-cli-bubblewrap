# agent-cli-bubblewrap

Scripts to run AI agent CLIs inside [bubblewrap](https://github.com/containers/bubblewrap) sandboxes, limiting their access to your home directory and secrets.

Inspired by [Patrick McCanna's article](https://patrickmccanna.net/a-better-way-to-limit-claude-code-and-other-coding-agents-access-to-secrets/).

The end-user-scripts are:

- `claude-bubble`
- `gemini-bubble`

## Usage

```
claude-bubble [-y] [-- <claude-options>]
gemini-bubble [-y] [-- <gemini-options>]
```

By default, each launches the agent CLI in its normal (approval-required) mode.
Pass `-y` to enable yolo mode (auto-approve all actions, no confirmation prompts).
Regular CLI options follow after `--`:

```
claude-bubble -y
gemini-bubble -y -- --model gemini-2.0-flash
```

Internally, the scripts call `bubble-run`, which sets up bubblewrap. Default sandbox permissions:

- Read-only access to system paths (`/usr`, `/lib`, `/etc/ssl`, etc.)
- Read-write access to the current directory
- A tmpfs `/tmp` and isolated PID namespace (`--unshare-pid`)
- Network access (`--share-net`)
- No access to .env

## Configuration

The scripts all contain the follow variables, that can be customized to your need.
By default, we give access to the current directory, minus `.env`.
We also give read access to the files needed by each agent to function.

- `ALLOW_RO`: allow read-only access to these files/directories
- `ALLOW_RW`: allow read-write access to these files/directories
- `BANNED`: explicitly deny access to these files, even if covered by `ALLOW_RO`/`ALLOW_RW`

## Requirements

- [bubblewrap](https://github.com/containers/bubblewrap) (`bwrap`) — available in most Linux distribution package managers
- The respective CLI tool (`claude`, `gemini`) installed and on your `PATH`
