# agent-cli-bubblewrap

Scripts to run AI agent CLIs inside [bubblewrap](https://github.com/containers/bubblewrap) sandboxes, limiting their access to your home directory and secrets.

Inspired by [Patrick McCanna's article](https://patrickmccanna.net/a-better-way-to-limit-claude-code-and-other-coding-agents-access-to-secrets/).

The end-user-scripts are:

- `claude-bubble`
- `gemini-bubble`
- `pi-bubble`

## Usage

```
claude-bubble [bwrap-options] [-y|c|h] [-- <claude-options>]
gemini-bubble [bwrap-options] [-y|c|h] [-- <gemini-options>]
```

- `-y`: YOLO mode; auto-approve all actions, no confirmation prompts.
- `-c`: Continue; resume the most recent conversation/session.
- `-h`: Show help.

Any [bwrap](https://github.com/containers/bubblewrap) flags placed before `--` are forwarded directly to bwrap.

Examples:

```
claude-bubble -y
claude-bubble -c
claude-bubble -y -c
claude-bubble --unsetenv GITHUB_TOKEN -y
gemini-bubble -y -- --model gemini-2.0-flash
```

Internally, the scripts call `bubble-run`, which sets up bubblewrap. Default sandbox permissions:

- Read-only access to system paths (`/usr`, `/lib`, `/etc/ssl`, etc.)
- Read-write access to the current directory
- All environment variables passed through (see below on how to prevent that)
- A tmpfs `/tmp` and isolated PID namespace (`--unshare-pid`).
  Note that this means you cannot run from within your local `/tmp` :)
- Network access (`--share-net`)
- No access to `.env.*`

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
export AGENT_BUBBLE_RO="$HOME/.authinfo.gpg:/run/user/1000/gnupg"
export AGENT_BUBBLE_RW="$HOME/.myagent"
export AGENT_BUBBLE_BANNED=".secret"
```

Each variable is a colon-separated list of paths. Of course, you could
also just modify the `ALLOW_RO`, `ALLOW_RW`, and `BANNED` variables in
each script directly.

### Hiding environment variables

All environment variables are inherited by the sandbox. To hide specific
ones, pass `--unsetenv` before `--`:

```sh
claude-bubble --unsetenv GITHUB_TOKEN --unsetenv AWS_SECRET_ACCESS_KEY
```

To start with a clean environment and allow only specific variables through:

```sh
claude-bubble --clearenv --setenv PATH "$PATH" --setenv HOME "$HOME"
```

## Requirements

- [bubblewrap](https://github.com/containers/bubblewrap) (`bwrap`) — available in most Linux distribution package managers
- The respective CLI tool (`claude`, `gemini`) installed and on your `PATH`
