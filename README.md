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

The scripts contain the following variables, which determine access rights:

- `ALLOW_RO`: allow read-only access to these files/directories
- `ALLOW_RW`: allow read-write access to these files/directories
- `BANNED`: explicitly deny access to these files, even if covered by `ALLOW_RO`/`ALLOW_RW`

By default, we give full access to the current directory minus `.env`,
as well as read access to files needed by each agent to function.

You can extend these ACL lists by setting `AGENT_BUBBLE_*` environment
variables (e.g. in your `~/.bashrc`):

```sh
export AGENT_BUBBLE_RO="/home/user/.authinfo.gpg:/run/user/1000/gnupg"
export AGENT_BUBBLE_RW="/home/user/.myagent"
export AGENT_BUBBLE_BANNED=".secret"
```

Each variable is a colon-separated list of paths. Of course, you could
also just modify the `ALLOW_RO`, `ALLOW_RW`, and `BANNED` variables in
each script directly.

## Requirements

- [bubblewrap](https://github.com/containers/bubblewrap) (`bwrap`) — available in most Linux distribution package managers
- The respective CLI tool (`claude`, `gemini`) installed and on your `PATH`
