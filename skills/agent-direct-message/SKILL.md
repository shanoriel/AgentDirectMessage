---
name: agent-direct-message
description: >-
  Cross-device real-time collaboration tool for strict turn-based 1:1 messaging
  with another AI agent via a relay server. Use when the user asks you to
  collaborate with an agent on another device, send a direct message to a remote
  agent, or coordinate work across machines using a shared token. Short name:
  Agent DM.
---

# Agent Direct Message

Use this skill when two agents need a simple shared relay channel for direct 1:1 collaboration across devices.

## Important Defaults

- The bundled client defaults to `http://api.junshanhuang.com:11451`.
- That endpoint is a public debug server for testing and demos.
- For privacy, security, and reliability, prefer a self-hosted relay and set `AGENT_DM_GATEWAY` before use.

## Install The CLI First

Resolve the absolute path to the directory containing this `SKILL.md` file. In the examples below, that path is written as `SKILL_DIR`.

Before running ADM commands, ensure the `agentdm` CLI is installed from the local skill directory:

```bash
command -v agentdm >/dev/null 2>&1 || python3 -m pip install --no-build-isolation SKILL_DIR
```

This keeps the command prefix stable for approvals and ensures the agent can bootstrap the CLI from the skill's own files.

## Workflow

Always check first, then send only after you know whether a message is already waiting.

### Step 1: Check the channel

```bash
agentdm check --token <TOKEN>
```

Output:

```json
{"role": "A", "message": null}
```

- If `message` is `null`, no one has sent anything yet and you are the initiator.
- If `message` is a string, the other agent already sent something and you are the responder.

### Step 2: Send a message and wait for the reply

```bash
agentdm send --token <TOKEN> --message "<YOUR_MESSAGE>"
```

This sends your message, then blocks until the other agent replies.

```json
{"message": "their reply", "from": "A"}
```

### Step 3: Close the session

```bash
agentdm close --token <TOKEN>
```

## Environment

- `AGENT_DM_GATEWAY`: Relay base URL. Defaults to `http://api.junshanhuang.com:11451`.
- `AGENT_DM_STATE_DIR`: Optional override for local PID/session state storage.
- By default, the client stores session PID state under `./.agent_dm/`.
- On session close or `SIGINT`/`SIGTERM`, the client removes only its own `<TOKEN>.pid` file.
- If you need privacy or stronger operational guarantees, point this at your own deployment.

## Typical Flows

### Initiator

1. Run `check`.
2. If `message` is `null`, send the first request with `send`.
3. Wait for the reply and continue only if another round is needed.
4. Finish with `close`.

### Responder

1. Run `check`.
2. If `message` contains a request, process it.
3. Reply with `send`.
4. Continue turn by turn until the exchange is complete, then use `close`.

## Guidelines

- Always check before sending.
- Use one message at a time.
- Parse JSON output instead of guessing state.
- Expect blocking waits of minutes when the other agent is busy.
- Do not use the public debug server for sensitive data.
- End the session cleanly with `close`.
