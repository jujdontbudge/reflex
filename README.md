# Reflex

Send your marks, words and screen from an iPad to Claude Code or Codex on your Mac. The agent answers in its own session; it cannot draw or edit the canvas.

## Connect

Use Node 26 or newer. From this checkout:

```sh
node agent/bin/reflex.mjs setup
```

Setup installs packages, builds the optional Apple peer-to-peer helper, configures the chosen harness, and starts the Link. Scan its pairing code in the iPad app and tap Allow when the agent asks. Use `node agent/bin/reflex.mjs pair` for a fresh code.

Pairing and agent approval are separate. Only an approving iPad, or the Link terminal before an approving iPad exists, grants agent access. Shell commands can deny or revoke access.

## Receive a Send

```sh
node agent/bin/reflex.mjs wait --timeout 0 --json
node agent/bin/reflex.mjs mcp
node agent/bin/reflex.mjs mcp --channel
node agent/bin/reflex.mjs link status
```

`wait_for_user` is the sole MCP tool. Claude Code can also receive channel pushes. An approved receiver can register a signed webhook with `reflex adapters add webhook --url <url>`. Acknowledge a Send with `wait_for_user(after: '<send>', timeout_s: 0)` and answer in your session.

## Repository

- `apps/ipad/`: iPad app, owned by T25 during the cut-down.
- `agent/`: MCP/CLI receivers and local RPC client.
- `link/`: pairing, approval, Mac connection, durable Inbox and delivery.
- `relay/`: authenticated forwarding of live encrypted device connections.

No agent writing API, shared Spaces/invites, web viewer, author kit, protocol package, conformance suite, encrypted cloud log or cloud agent session remains. The full earlier product is preserved on branch `archive/canvas-v0`.

## Checks

```sh
npm install --no-audit --no-fund
npm run typecheck
```

There is no lint script. See [architecture](docs/ARCHITECTURE.md), [spec](docs/SPEC.md) and package READMEs. Relay changes are source-only: deploying reflex.eilmer.si requires the person's approval. Never change relay.eilmer.si DNS.
