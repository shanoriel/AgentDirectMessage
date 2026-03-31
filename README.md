# Agent Direct Message

`agent-direct-message` is an installable agent skill plus a reference relay server for strict turn-based 1:1 messaging between two AI agents on different devices. Short name: `Agent DM`.

It ships a local-installable Python CLI named `agentdm`, so agents and humans can use a stable command instead of calling ad hoc repo-local scripts.

The skill can be installed with the open `skills` ecosystem:

```bash
npx skills add https://github.com/shanoriel/agent-dm --skill agent-direct-message
```

## What This Repository Contains

- `skills/agent-direct-message/`: the installable skill package and embedded Python CLI source
- `server.py`: a sample relay server you can self-host
- `requirements.txt`: server dependencies

## Install The CLI

If Python is already available, the simplest setup is:

```bash
python3 -m pip install --no-build-isolation ./skills/agent-direct-message
```

That exposes a stable command:

```bash
agentdm --help
```

For local development, editable install also works:

```bash
python3 -m pip install --no-build-isolation -e ./skills/agent-direct-message
```

## Default Debug Server

The bundled client defaults to this public debug relay:

```text
http://api.junshanhuang.com:11451
```

This is provided for evaluation, demos, and quick testing only.

## Recommended Deployment Model

For privacy, security, and reliability, users should run their own relay server and point the skill at that endpoint with `AGENT_DM_GATEWAY`.

Reasons to self-host:

- conversation content passes through the relay server
- session availability depends on that server staying up
- public shared infrastructure is not appropriate for sensitive or regulated work
- self-hosting lets you add your own TLS, auth, logging, rate limiting, and network controls

If you expose the server on the public internet, put it behind HTTPS and the access controls you expect for your environment.

## Self-Hosting The Server

Install dependencies:

```bash
python3 -m pip install -r requirements.txt
```

Run the relay:

```bash
python3 server.py
```

The sample server listens on port `11451` by default. To change it:

```bash
AGENT_DM_PORT=8080 python3 server.py
```

Then point the CLI or skill to your server:

```bash
export AGENT_DM_GATEWAY=http://your-host:11451
```

## CLI Usage

Check the channel:

```bash
agentdm check --token 114514
```

Send a message and wait for the reply:

```bash
agentdm send --token 114514 --message "Hello"
```

Close the session:

```bash
agentdm close --token 114514
```

## How The Skill Works

Two agents share a token and communicate in alternating turns:

1. Both agents use the same token.
2. Each agent runs `check` first to join the channel.
3. The initiator sends a message with `send`.
4. The sender blocks until the partner replies.
5. Either side ends the session with `close`.

The client stores its local participant ID in `./.agent_dm/` by default, or in `AGENT_DM_STATE_DIR` if you override it. When a session closes or the client receives `SIGINT`/`SIGTERM`, it removes only its own `<TOKEN>.pid` file and does not delete other files in that directory.

## Local Development

You can inspect the skill before publishing:

```bash
npx skills add /path/to/agent-dm --list
```

You can also install it into a local project:

```bash
npx skills add /path/to/agent-dm --skill agent-direct-message -a codex
```
