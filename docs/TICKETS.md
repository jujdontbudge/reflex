# Reflex tickets

The build is split into 22 tickets, one agent each. [ticket-map.html](./ticket-map.html) shows which can run at the same time. Steps and their outcomes are summarized in [ARCHITECTURE.md §12](./ARCHITECTURE.md).

Every ticket is proven by using the real thing: the app in the iPad Simulator, a real Claude Code or Codex session, or the real CLI. No ticket builds benchmarks, eval harnesses or test suites.

## All tickets

There are 55 work items (H1–H55), and each one is in exactly one ticket.

| Ticket | What's in it | Can start |
|---|---|---|
| T1 Protocol v0 | H9 schemas for every message, including `data`, Behaviors, `define`, Embed state and added files, with a few correct examples · H10 hand-written tool text and server instructions · H51 owners as people or Agents, and `from` on every Send | Now |
| T2 Kernel | H5 Element types · H6 `apply`, `scene`, `freeze_send` · H7 Swift and WASM bindings, with a stop-and-report if they don't work or the copies don't converge · H8 Automerge mapping · H43 the `data` map on every Element · H52 person IDs on every owner and undo per person | Now |
| T3 iPad canvas | H1 PencilKit Ink, Goodnotes-style eraser and undo, reload, stable IDs · H2 minimum iPadOS · H3 drawing Elements · H4 Send button | Now |
| T4 Agent layer | H11 `open_space`, `apply`, `read`, `look`, `wait_for_user` over MCP and the CLI · H12 one real wait in Claude Code and one in Codex, recording each one's timeout and MCP version | After T1, T2 |
| T5 Link | H13 journal, LAN, pairing, approval | After T2 |
| T6 Round trip | H14 connect the iPad, Link and Agent; the Agent draws, you circle, the Agent replies; kill the Link once after a Send | After T3, T4, T5 |
| T7 Plan review iPad | H15 arrow, strikethrough and writing Marks · H16 voice · H17 undo UI · H44 Behavior triggers (tap, long-press, drag end, Ink inside, Panel change) · H46 adding photos, camera captures, PDFs and files | After T6 |
| T8 Plan review core | H18 Mark grouping and Anchors · H19 transcript interleaved with Marks · H20 revert · H21 `look` without the iPad · H22 CLI parity · H45 Behavior actions (open Panel, notify, Send, set, apply Recipe) · H47 added files in the Send Bundle | After T6 |
| T9 Relay | H23 encrypted log, OAuth, grants · H24 run a real Space in a Durable Object · H25 self-hosted Node Relay | After T6 |
| T10 Web + author kit | H26 web viewer · H27 `reflex create/test/dev` · H50 `define` and the Library | After T6 |
| T11 Peer-to-peer | H28 Apple peer-to-peer Wi-Fi, including the same change arriving over two paths | After T6 |
| T12 Math tutor iPad | H29 LaTeX on iPad · H30 Region and Region Send button | After T7 |
| T13 Math tutor core | H31 Regions, line-level evidence, grading Recipe and Guide, watch mode | After T8 |
| T14 Deploy Relay | H32 reflex.eilmer.si | After T9 |
| T15 Claude channel | H33 push, then pull out the shared adapter contract | After T8 |
| T16 Chart Embed | H34 chart Embed Template and anchor bridge · H35 HTML sandbox hardening · H36 an old app keeps an Element type it doesn't know · H48 `setState` and `emit` on the bridge · H49 Embeds inside Panels | After T10, T12 |
| T17 Webhook, MCP App | H37 webhook adapter · H38 MCP App adapter | After T14, T15 |
| T18 3D Embed | H39 RealityKit, glTF/USDZ part maps | After T16 |
| T19 Scenarios 3–5 | H40 CAD, chart and grading Recipes | After T18, T13 |
| T20 Scenarios 6–13 | H41 the remaining Recipes and Guides | After T18 |
| T22 Shared Spaces | H53 invites, roles and membership checks · H54 Space key sharing and rotation on removal · H55 live presence, and each person's Sends carrying only their Marks | After T17 and T12 |
| T21 Protocol v1 | H42 publish the protocol and conformance profiles | Last |
| T23 Setup | first-time setup in one command, one scan and one tap; approval that can't get lost; quiet logs; re-pairing with old data | After T21 |

Tickets for later steps are written when their step is next, so they can build on what earlier tickets found.

## Step 1 tickets

```md
# Protocol v0: the shapes and words every Agent sees

## Goal
Write the first version of Reflex's public contract: the JSON Schemas for every message, and the hand-written text every Agent reads. When this is done, `protocol/` holds:
- schemas for `apply` and its operations (`add`, `patch`, `remove`, `arrange`, `focus`, `define`) and its results
- schemas for errors with `fix` hints
- schemas for the Send Bundle
- schemas for the other tools' inputs and outputs
- owners as principals: a person or an Agent, each with a stable ID, and `from` on every Send naming the person who sent it
- one correct, hand-written example for each message type

Next to them sit the descriptions of the eleven tools and server instructions of about 400 tokens or fewer, all written by hand.

Everything else in Reflex builds on these shapes, and the Agent is Reflex's real user. The shapes must also leave room for uses nobody designed: every Element has a free `data` map, Behaviors pair a trigger with an action, `define` saves an Agent's Recipe or Embed Template to the Library, and the Send Bundle carries Embed state and files the user added. Owners are people or Agents from the start, so shared Spaces can come later without migrating anyone's Spaces (ADR-0008).

## Context
- Read docs/ARCHITECTURE.md §4 (the Agent contract, with example `apply` and Send Bundle JSON), §5 (the Embed bridge) and §9 (schema rules); docs/SPEC.md §2, §4, §5, §7, §9 and §11; docs/adr/0003, 0006 (including its amendment), 0007 and 0008; and docs/architecture/review-fable.md §2, the source of the tool list, the op and Bundle shapes, and ten ways models get confused.
- The user decided:
  - The eleven tools are fixed, and plugins never add tools.
  - The Inbox is reached through `wait_for_user(timeout_s: 0)`.
  - A Send is acknowledged through `after` or `in_reply_to`, with `redelivered: true` on resends.
  - Each Agent has one stable coordinate space, and placement is relative by default.
  - Built-in Element types have typed, top-level fields.
- Schema rules from ARCHITECTURE §9: JSON Schema 2020-12 in a restricted subset; one level of `oneOf`, because some harnesses' strict modes handle deep unions badly; patches are explicit operations; integers stay in the shared safe range; text ranges are UTF-16.
- T2 (the Rust kernel) runs at the same time and generates its types from these schemas once they land. T3 (the iPad canvas) draws from scene JSON in these shapes.
- T4 builds the MCP and CLI surfaces on this text and runs it in real Claude Code and Codex sessions. That run is how the text gets proven.

## Constraints
- The tool list is exactly the eleven tools in SPEC §11.
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

```md
# Kernel: the Rust core that owns every Space

## Goal
Build the Rust Space kernel and bind it to both Swift and TypeScript. When this is done:
- The kernel crate maps Reflex's Elements onto Automerge.
- It covers the Element envelope (ID, owner, layer, the free `data` map), Ink, text, shapes and connectors.
- Every owner is a person or an Agent with a stable ID, and undo covers one person's own actions across all their devices (ADR-0008).
- It applies `apply_agent_batch`, projects `scene`, and freezes a Send with `freeze_send`.
- Swift calls it through UniFFI in an app running in the iPad Simulator.
- TypeScript calls it through WASM, in Node and in Cloudflare Workers.

A Swift copy and a TypeScript copy of the same Space must still agree after edits made to both at the same time, including create, patch and delete, and after one restart.

If the bindings don't work, or the copies don't converge, stop and report what failed. ADR-0005's fallback then applies. That go/no-go is the point of building the kernel first: a disagreement here would be a malformed document copied to every device.

## Context
- Read docs/adr/0001, 0004, 0005 and 0008; docs/ARCHITECTURE.md §3 (the kernel API and guarantees), §7 (correctness rules), §9, §11 and §13; and docs/architecture/gpt-sol.md §1.2 for the data model.
- The kernel is sans-IO. Events and bytes go in; new state, effects and bytes come out. It contains no sockets, files, clocks or threads, and callers inject time and random seeds.
- The engine is Automerge. Loro is the fallback only if Automerge visibly fails while you build and use the kernel.
- Pencil samples never cross into the kernel. A finished stroke crosses once.
- The Library (saved Recipes and Embed Templates) is an ordinary Space of kind `library`.
- Cloudflare Workers cap memory at 128 MB per isolate, including WASM.
- T1 is writing the protocol schemas at the same time. Use stand-in types for Element data until they land, then generate from them. T3 is building the iPad canvas at the same time.

## Constraints
- The kernel crate does no IO.
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

```md
# iPad canvas: Ink, eraser, undo and drawing Elements

## Goal
Build the first real version of the Reflex iPad app's canvas. When this is done, the app runs in the iPad Simulator. You can:
- draw Ink with PencilKit
- erase with a stroke eraser, a precise eraser and a lasso
- undo and redo with the toolbar, a two-finger tap and a three-finger tap
- close and reopen the app with every stroke intact

Every stroke, and every fragment left after a partial erase, keeps a stable ID through all of that. The canvas also draws Elements (text, shapes, connectors, images) from scene JSON, and has a floating Send button.

Ink is how users talk to Reflex. Every Mark, Send and Anchor is built on stroke identity, so this has to be right before anything builds on it. The ticket also settles the minimum iPadOS version.

## Context
- Read docs/SPEC.md §5 and §6 (Ink, eraser and undo); docs/ARCHITECTURE.md §3, §7.5, §8.2 and §13 (the Ink row); and docs/architecture/gpt-sol.md §1.2 and §10.2.
- The user decided:
  - The eraser and undo work like Goodnotes. Pencil double-tap or squeeze switches to the eraser.
  - Undo covers only that person's own actions.
  - Agents never edit or erase user Ink.
- From the architecture:
  - A precise erase records new fragments or a mask that point back to the parent stroke, not a rewrite of the whole drawing.
  - Reflex keeps a native PencilKit archive for fidelity, plus a portable stroke form for the web viewer, export, and the Agent's `read(detail: "strokes")`.
  - PencilKit is never re-serialized when Agent content changes.
- Apple's WWDC26 session "Read between the strokes with PencilKit" (https://developer.apple.com/videos/play/wwdc2026/203/) describes stable stroke and path IDs, on-device handwriting recognition, programmatic erase and substroke extraction. The minimum iPadOS is whatever those APIs require.
- Use the Xcode iPad Simulator through T3 Code's device tools, so the user can watch in the Device panel. The user has an Apple Developer account. A real Pencil pass (pressure, tilt, squeeze) comes later and is not part of this ticket.
- T1 is writing the scene JSON shapes and T2 is building the kernel at the same time. Draw from stand-in scene JSON until T1's examples land. Connecting to the kernel happens in T6.

## Constraints
- PencilKit is the Ink engine. Report anything that blocks it; don't build an alternative.
- The app lives in apps/ipad/, laid out as in ARCHITECTURE §10.
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

## Step 2 tickets

```md
# Agent layer: the MCP server and CLI that Agents use

## Goal
Build the TypeScript Agent layer, the part of Reflex every Agent talks to. When this is done, a real Claude Code session and a real Codex session can each connect to Reflex, through MCP and through the CLI, and use `open_space`, `apply`, `read`, `look` and `wait_for_user` against a real Space held by the Rust kernel (through its WASM build). The kernel accepts exactly the shapes in `protocol/`: its Rust op and scene types are generated from `protocol/schema`, so T1's `apply` example goes through it. `protocol/` is part of the root npm workspace.

You also record what each harness actually does with a long-held `wait_for_user` call, from one real run in each: the default tool timeout and how to change it, whether progress notifications extend it, what cancelling does, what happens when the pipe closes mid-wait, and which MCP version or binding it speaks. Write that down in docs/harness-notes.md. It decides the timeout policy and which MCP binding ships first.

This is the first time an Agent touches a real Space. Everything after it (the round trip, every scenario, every push adapter) is built on this layer.

## Context
- Read docs/ARCHITECTURE.md §1, §2, §4 (the Agent contract), §6 (Send delivery), §8.1, §9, §10 and §11 (the MCP row); docs/SPEC.md §7–§9, §11 and §12 (the CLI, following https://vercel.com/design/cli); docs/adr/0002, 0003 and 0007; protocol/ (schemas, examples, tool text, `pnpm check`); and kernel/README.md.
- What exists on main: `protocol/` (T1: schemas, 31 examples, hand-written tool text and server instructions), `kernel/` (T2: Rust kernel on Automerge, UniFFI and WASM bindings, convergence proven), and `apps/ipad/` (T3).
- T2's kernel still parses its own stand-in op shapes. T1's `apply` example is rejected because T1 sends `map_put`/`text_insert` as lists with a `field` and adds `behaviors`, `math` and `embed`. Generating the Rust types from `protocol/schema` changes op parsing and scene output, not how Elements map onto Automerge.
- `protocol/` has its own `package.json` and pnpm lockfile; the root uses npm workspaces.
- T1's worker measured the bundled `apply` input schema at about 26 KB (roughly 6,500 tokens) sent with every request. See what the harnesses do with it.
- T5 (the Link) runs at the same time and owns `link/`: durable storage, the Inbox journal, LAN sync and pairing. Until T6 joins the two, run the Agent layer from a small entry point with an in-memory Space and Inbox. Keep storage and transport behind the interfaces in ARCHITECTURE §6 so T6 can plug T5's in.
- `look` goes through a renderer interface. A basic scene-to-image renderer is enough here; the full headless renderer is T8.
- Claude Code and Codex are both installed on this machine.

## Constraints
- The tool list is exactly the eleven tools in SPEC §11, with the text from `protocol/tool-text/`. This ticket implements five of them; the rest can return a clear "not yet available" error.
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

```md
# Link: the companion process that keeps Spaces safe and in sync

## Goal
Build Reflex Link, the process on the user's computer that keeps a copy of every Space, stores it durably, and syncs it with other devices over the local network. When this is done, `npx reflex link` (or the repo's equivalent while unpublished) starts the Link. It:
- keeps each Space in a durable journal plus a content-addressed asset store, using the Rust kernel through WASM
- keeps a durable Inbox index, so a Send that arrives is never lost
- finds peers with Bonjour and syncs Spaces with them over LAN/TCP, through the kernel's `sync_step`
- pairs a new device by showing a QR code that exchanges keys, and binds a connection to a device only after it proves its key
- asks the user to approve each new Agent the first time it connects, and remembers the decision in a credential store

Two Links (or a Link and a second peer process) on this machine must pair, sync a Space both ways, keep syncing after edits made on both sides at the same time, and come back intact after one of them is killed and restarted.

The Link is what lets local Agents and the iPad meet. The round trip (T6) joins it with the Agent layer and the iPad app.

## Context
- Read docs/ARCHITECTURE.md §1, §2 (the `link` unit), §6 (transport, storage, identity), §7 (correctness rules, especially the Send state machine and retention), §8 and §10; docs/SPEC.md §3, §4 and §13; docs/adr/0001, 0004, 0005 and 0008; and kernel/README.md.
- What exists on main: `protocol/` (T1), `kernel/` (T2: Rust kernel with WASM bindings; convergence and restart were proven across Swift, Node and Workers), and `apps/ipad/` (T3).
- T2 noted the kernel does not yet check authorization on changes it receives, and the WASM build is 3.0 MB.
- Every owner is a person or an Agent with a stable ID (ADR-0008). Devices are bound to a person.
- Fastest-path selection waits until a second real transport exists (T11). Build LAN/Bonjour first, behind the transport interface in ARCHITECTURE §6.
- T4 (the Agent layer) runs at the same time and owns `agent/`. Don't build MCP or CLI tools here; T6 plugs the Agent layer into the Link. Both of you add workspaces to the root `package.json`; whoever merges second rebases.
- The iPad's side of pairing and sync is T6.

## Constraints
- Use the kernel for every Space operation; the Link never edits Space data itself.
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

## Step 3 ticket

```md
# Round trip: an Agent draws, you circle, the Agent answers

## Goal
Join the iPad app, the Link and the Agent layer into the first real Reflex round trip. When this is done:
- `reflex link` (from the repo; publishing comes later) starts the Link with the Agent layer inside it. A harness connects through `reflex mcp` or the CLI to the running Link, so Agents read and write the Link's durable Spaces and Inbox instead of T4's in-memory ones.
- The iPad app pairs with the Link (scanning the QR code, or typing the code where there is no camera, such as the Simulator), syncs Spaces with it over the local network, and talks to the Rust kernel through T2's Swift bindings.
- The iPad draws Elements from the real Space in the Protocol v0 Element format, replacing T3's stand-in scene JSON, and shows Agent changes live as they sync.
- When the person presses Send, the iPad freezes the Send in the kernel with its stroke IDs, viewport and the crops it rendered, and syncs it to the Link. The Link groups the strokes into Marks and finds what a circle encloses. A waiting `wait_for_user` call is released with a Send Bundle that names the circled Element's ID and includes the crop.

Prove the whole loop with a real Claude Code session and the app in the iPad Simulator: the Agent draws a small graph, you circle one node and press Send, the Agent gets the Mark with that node's ID and a crop, and it answers on its own layer next to the circle, which you then see on the iPad. Then press Send once more and kill the Link right after; when the Link restarts, that Send is still delivered.

This is the first time the whole product works end to end. Every scenario after it builds on this loop.

## Context
- Read docs/ARCHITECTURE.md §1–§4, §6, §7 (especially the Send state machine and marking-time evidence) and §8.1–§8.2; docs/SPEC.md §3, §5, §8, §9 and §13; docs/adr/0002 and 0008; docs/harness-notes.md; and the READMEs in agent/, link/, kernel/ and apps/ipad/.
- What exists on main:
  - `protocol/` (T1): schemas, examples and tool text.
  - `kernel/` (T2): Rust kernel on Automerge with Swift (UniFFI) and WASM bindings. `freeze_send` exists; Mark grouping and Anchor resolution don't yet.
  - `apps/ipad/` (T3): PencilKit Ink with stable stroke IDs, Goodnotes-style eraser and undo, an Element renderer on stand-in scene JSON, and a Send button. It targets iPadOS 26.
  - `agent/` (T4): MCP and CLI surfaces for `open_space`, `apply`, `read`, `look` and `wait_for_user`, running on an in-memory Space and Inbox behind storage interfaces. Codex cuts MCP calls at 300 s, so waits are held for at most 280 s there.
  - `link/` (T5): durable SQLite journal and Inbox, LAN sync over Bonjour and TCP, QR pairing with encrypted connections, Agent approval. Its `dev` subcommands stand in for the iPad and an Agent.
- Carried over from earlier tickets (docs/TICKETS.md, "Follow-ups"):
  - The kernel doesn't check authorization on changes it receives from peers, so a paired device is trusted for everything it sends. Check owners and roles on received changes at the writer's frontier (ARCHITECTURE §7.2).
  - `link/`'s `admin.key` is readable by the user's account, so a local Agent with shell access could read it and approve itself. Close that.
- A circle around one Element is enough Mark recognition here. Arrows, strikethrough and writing come in T7 and T8.
- Drive the Simulator with T3 Code's device tools so the user can watch. Claude Code is installed on this machine.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

## Step 4 tickets

```md
# Plan review on the iPad: arrows, strikethrough, writing, voice and Behaviors

## Goal
Make the iPad capture everything scenario 1 (the coding plan review in SPEC §16) needs from the person, and show everything Agents and other devices put in the Space. When this is done, on the iPad:
- Arrows, strikethroughs and handwriting next to an Element are captured with the stroke evidence and Apple's on-device handwriting hint that the Link needs to recognize them.
- Holding the mic button records speech, transcribed on the iPad with on-device recognition and word timings; releasing it sends the voice and the Ink together.
- Each Agent has a color chip that reverts its last change, and each Space has a history timeline showing who did what.
- Behavior triggers fire from the iPad: tap, long-press, drag end, Ink drawn inside, and a Panel value changing.
- The person can add photos, camera captures, PDFs and files (Photos, Files, the camera, drag and drop). Photos and PDF pages become image Elements the person can write on; other files become file Elements.
- Ink drawn on other devices is drawn on this iPad too (T6 only drew the iPad's own Ink).

## Context
- Read docs/SPEC.md §5, §6, §9, §10 and §16 (scenario 1); docs/ARCHITECTURE.md §4.3, §7.5 and §8.2; docs/adr/0006 (its amendment) and 0008.
- What exists on main: `protocol/` (T1), `kernel/` (T2), `apps/ipad/` (T3), `agent/` (T4), `link/` (T5) and the working round trip (T6): an Agent draws, the person circles an Element on the iPad and presses Send, and the Agent gets the circled Element's ID and a crop. Read the READMEs in each folder and docs/harness-notes.md.
- T8 runs at the same time and owns the Link and Agent side: recognizing these Marks, the interleaved transcript in the Send Bundle, revert, Behavior actions and added files in the Bundle. You own `apps/ipad/`. Agree on shapes through `protocol/`.
- Scenario 1 must work end to end once both T7 and T8 land; prove your half by running the real app against a real `reflex link`.
- Drive the Simulator with T3 Code's device tools so the user can watch. A real iPad (iPad13,6, paired) is connected over USB; use it for anything the Simulator can't show, such as the microphone or the camera. T6 added a debug-only `reflex://debug/ellipse` link because the automation tool only drags in straight lines; use debug links only for stroke shapes the tool can't draw, and say so.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

```md
# Plan review core: Marks, transcript, revert, headless look and CLI parity

## Goal
Make scenario 1 (the coding plan review in SPEC §16) work end to end for Agents, in Claude Code and Codex, through both MCP and the CLI. When this is done:
- The Link groups strokes into Marks and resolves their Anchors: circles, arrows from one Element to another, strikethroughs, and writing next to an Element with its recognized text and confidence.
- The Send Bundle interleaves the voice transcript with the Marks it refers to ("merge these [m1] [m2]"), as in ARCHITECTURE §4.3.
- `revert` undoes an Agent's own batches through the kernel's revert planning.
- `look` works without the iPad: the Link renders the Space headlessly.
- Every capability is reachable from the CLI as well as MCP, including `read_me` and `recipe`, so all eleven tools work.
- Behavior actions run: open a Panel, notify, Send, set a field, apply a Recipe.
- Files the person added appear in the Send Bundle (photos and PDF pages as images, other files by reference, read with `read`).

## Context
- Read docs/SPEC.md §7–§9, §11, §12 and §16; docs/ARCHITECTURE.md §3, §4, §5, §7 and §8; docs/adr/0003, 0006 and 0007.
- What exists on main: `protocol/` (T1), `kernel/` (T2), `apps/ipad/` (T3), `agent/` (T4), `link/` (T5) and the working round trip (T6): an Agent draws, the person circles an Element on the iPad and presses Send, and the Agent gets the circled Element's ID and a crop. Read the READMEs in each folder and docs/harness-notes.md.
- Carried over (docs/TICKETS.md, "Follow-ups"): T4's open review items S3, S5, S7 and S8; `read_me` and `recipe` aren't built though the server instructions point Agents at `read_me`; the CLI's image `file` path has no schema field; Agent Element IDs show as `agent:id`; T2's kernel follow-ups (the digest skips Send contents, connector moves do nothing, two undo/redo edge cases); and from T6, an Agent turned away isn't throttled when it keeps asking.
- T7 runs at the same time and owns `apps/ipad/`: capturing these Marks, voice and imports on the iPad. You own `agent/`, `link/` and `kernel/` for this work. Agree on shapes through `protocol/`.
- T9 (Relay) and T10 (web viewer and author kit) also run now. T10 adds its own CLI command group; whichever of you merges second rebases.
- If Codex's usage limit runs out, prove the Codex half in Claude Code only and say so (the user decided this).

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

```md
# Relay: reach a Space from anywhere, with an encrypted log

## Goal
Build the Relay, the hosted service that lets Agents and devices outside the local network reach a Space. When this is done:
- A Cloudflare Worker with Durable Objects stores each Space as an encrypted log it cannot read, plus encrypted assets in R2, and routes encrypted changes between a person's devices.
- Remote MCP over OAuth lets a cloud Agent connect. A device issues a scoped, expiring grant for one Space; only under that grant does a cloud session decrypt and materialize the Space and run the same `kernel` and `agent` code.
- The Link and the iPad can sync through the Relay over WebSocket, as a second path next to the LAN.
- A self-hosted Node Relay runs the same code outside Cloudflare.
- A real materialized Space runs inside the Durable Object within Workers' 128 MB. If it runs out of memory there, cloud execution moves to a separate service and the Durable Object stays the encrypted router and log.

Run everything locally (`wrangler dev` and Node). Deploying to reflex.eilmer.si is T14.

## Context
- Read docs/adr/0001, 0004 and 0008; docs/SPEC.md §3 and §13; docs/ARCHITECTURE.md §2 (the `relay` unit), §6 (transport, storage, identity), §8.4 and §13 (the Relay row).
- What exists on main: `protocol/` (T1), `kernel/` (T2), `apps/ipad/` (T3), `agent/` (T4), `link/` (T5) and the working round trip (T6): an Agent draws, the person circles an Element on the iPad and presses Send, and the Agent gets the circled Element's ID and a crop. Read the READMEs in each folder and docs/harness-notes.md.
- The kernel's WASM build is about 3.8 MB.
- T8 (core), T7 (iPad), T10 (web) and T11 (peer-to-peer) run at the same time. You own `relay/` and the Relay transport behind the interface in ARCHITECTURE §6. T11 adds another transport; keep yours in its own module.
- Never deploy to Cloudflare in this ticket.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

```md
# Web viewer, author kit and the Library

## Goal
Build the web viewer and the tools for people and Agents who extend Reflex. When this is done:
- A web viewer opens a Space from the Link in a browser and draws it, live.
- `reflex create guide|recipe|embed|adapter <name>` writes a working example; `reflex test` runs that kind's contract checks; `reflex dev` opens the viewer on an in-memory Space with hot reload and three panes (what the Agent reads, what a Send produces when you draw with the mouse in dev mode, and live lints and Anchors), with a "save example" button.
- An Agent saves a Recipe or Embed Template it invented with `apply`'s `define` op. It lands in the person's Library (an ordinary Space of kind `library`), is checked like any extension, and appears in `read_me("index")` for every Agent in every Space.

## Context
- Read docs/SPEC.md §7 ("Agents save what they invent"), §12 and §14; docs/ARCHITECTURE.md §3 (the Library), §4.2 (`define`), §5 (the extension ladder and author kit) and §10; docs/adr/0006 and its amendment.
- What exists on main: `protocol/` (T1), `kernel/` (T2), `apps/ipad/` (T3), `agent/` (T4), `link/` (T5) and the working round trip (T6): an Agent draws, the person circles an Element on the iPad and presses Send, and the Agent gets the circled Element's ID and a crop. Read the READMEs in each folder and docs/harness-notes.md.
- T8 runs at the same time and builds `read_me` and `recipe` and CLI parity in `agent/`. Add your CLI commands as their own group and keep `define`'s Library storage separate from T8's files where you can; whichever of you merges second rebases.
- None of this needs an iPad or Xcode.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

```md
# Peer-to-peer: a second path between the iPad and the Mac

## Goal
Add a second real transport between the iPad and the Link: Apple peer-to-peer Wi-Fi, so they stay connected with no shared Wi-Fi network. When this is done:
- The iPad and the Mac find each other and connect over peer-to-peer Wi-Fi (Network.framework), with the Mac side as a small Apple-only Swift helper the Link talks to.
- With both LAN and peer-to-peer up, each change takes the fastest live path, chosen with hysteresis, and moves to the other path when one drops.
- The same change arriving over both paths is harmless.
- A fault-injection transport exists for dropping, delaying and duplicating frames.

## Context
- Read docs/SPEC.md §3 and §13; docs/ARCHITECTURE.md §6 (the transport interface and build order) and §7; docs/adr/0001.
- What exists on main: `protocol/` (T1), `kernel/` (T2), `apps/ipad/` (T3), `agent/` (T4), `link/` (T5) and the working round trip (T6): an Agent draws, the person circles an Element on the iPad and presses Send, and the Agent gets the circled Element's ID and a crop. Read the READMEs in each folder and docs/harness-notes.md.
- A real iPad (iPad13,6) is paired and connected to this Mac over USB, and the user has an Apple Developer account. Peer-to-peer Wi-Fi only works on real hardware, so prove it on that iPad with the Mac. Build and install with Xcode's command-line tools; don't run anything that triggers macOS password or keychain prompts.
- T9 (Relay) adds a WebSocket transport at the same time; T7 edits the iPad app's screens. Keep your networking in its own module on both sides.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

## Step 5 tickets (core side)

```md
# Math tutor core: Regions, line-level grading and watch mode

## Goal
Make scenario 2 (the math and physics tutor in SPEC §16) work for Agents. When this is done:
- Regions have their own rules: whether Ink is allowed, whether Agents may write there, whether watch mode is on, and a Send button scoped to that Region ("Check my work") whose Send carries only that Region's work.
- A LaTeX problem can sit next to a writable Region.
- When the person writes a solution by hand, the Send gives the Agent line-level evidence: the handwritten lines in order, each with its strokes, bounds and recognized text, so the Agent can put a margin mark at the exact line and tell a sign error from a gap in understanding.
- The `check-my-work` Recipe places a problem, a writing Region and its Send button in one call, and a math-tutoring Guide teaches Agents to grade step by step with hints next to the Ink.
- An Agent can turn on watch mode for one Region: debounced snapshots of only that Region, with rate and size limits, marked `user_still_writing` while the person is mid-stroke.

Prove it with a real Claude Code session tutoring a scripted person through one problem with a deliberate sign error, using both the Send button and watch mode.

## Context
- Read docs/SPEC.md §5 (Regions), §9 (watch mode) and §16 (scenario 2); docs/ARCHITECTURE.md §4.2–§4.3 and §7 (Region policy at the writer's frontier, retention and watch-mode budgets); docs/adr/0006 and 0007.
- What exists on main: `protocol/`, `kernel/`, `apps/ipad/`, `agent/` and `link/`, with the round trip (T6), scenario 1's core (T8: Marks, the interleaved transcript, revert, headless `look`, all eleven tools on MCP and the CLI, Behavior actions, added files) and the web viewer, author kit and Library (T10). Read the READMEs in each folder and docs/harness-notes.md. T8 used scripted "person" commands on a `reflex link --dev`; reuse them.
- Carried over (docs/TICKETS.md, "Follow-ups"): nothing caps how many Elements one Mark lists, and Marks re-fork the Space once per Mark (T8).
- T12 does the iPad side later (drawing LaTeX, the Region and its Send button on the iPad). T15 (Claude Code channel push) runs at the same time and changes Send delivery in `agent/` and `link/`; keep your changes to Regions, Marks, Recipes and Guides, and whoever merges second rebases.
- Codex's usage limit is exhausted until 2026-11-12 13:06; prove in Claude Code and say so.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

```md
# Claude Code channel: push a Send straight into the session

## Goal
Deliver Sends to Claude Code by push, so the person's Send reaches a running Claude Code session without the Agent sitting in `wait_for_user`. When this is done:
- Reflex speaks Claude Code's channel mechanism (`notifications/claude/channel`, a research preview over stdio): when a Send for that Agent arrives, the session receives it as a new turn with the Send Bundle.
- The push adapter contract is pulled out of this first adapter: its config, secrets, callback endpoint, and delivery attempts recorded durably outside the adapter. T17 builds the webhook and MCP App adapters on it.
- A push that fails or isn't supported falls back to the Inbox, so `wait_for_user` still gets the Send and nothing is lost or delivered twice.

Prove it with a real Claude Code session: the person sends while Claude is idle, and Claude answers without having called `wait_for_user`. Also kill the session mid-push once and show the Send waiting in the Inbox afterwards.

## Context
- Read docs/adr/0002; docs/SPEC.md §9 (delivery) and §11; docs/ARCHITECTURE.md §4.3, §6 (Send delivery) and §7.4 (the Send state machine); docs/harness-notes.md.
- What exists on main: `protocol/`, `kernel/`, `apps/ipad/`, `agent/` and `link/`, with the round trip (T6), scenario 1's core (T8: Marks, the interleaved transcript, revert, headless `look`, all eleven tools on MCP and the CLI, Behavior actions, added files) and the web viewer, author kit and Library (T10). Read the READMEs in each folder and docs/harness-notes.md. T8 used scripted "person" commands on a `reflex link --dev`; reuse them.
- Claude Code channels are a research preview and work over stdio only. Check the installed Claude Code's own docs and flags for how to enable them.
- Carried over (docs/TICKETS.md, "Follow-ups"): tap actions on a revoked Agent's Elements still run its `set` and `apply_recipe` (the runner doesn't check approval); `wait_for_user(element:)` misses taps whose action is `send`; `request_attention` never reports that the person is already viewing that Space (T8).
- T13 (math tutor core) runs at the same time in `agent/` and `link/`; keep your changes to delivery and adapters, and whoever merges second rebases.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

```md
# Deploy the Relay to reflex.eilmer.si

## Goal
Run the official Relay on Cloudflare at `reflex.eilmer.si`, so cloud Agents and devices away from home can reach a Space. When this is done:
- The T9 Worker, its Durable Objects and its R2 bucket are deployed to the user's Cloudflare account, served at `https://reflex.eilmer.si` (the `eilmer.si` zone is already active on that account).
- Remote MCP over OAuth works at that address: a real cloud Agent signs in, a device issues a grant for one Space, and the Agent reads and answers a Send.
- The Link and the iPad sync through the deployed Relay, and the Relay URL they use by default is `reflex.eilmer.si` (a self-hosted URL still works).
- Secrets live in Cloudflare (`wrangler secret`), never in the repo; a short deploy guide in `relay/README.md` says how to redeploy and how to self-host instead.

Prove it from outside the LAN: an iPad Simulator or Link reaching a cloud Claude Code session through `reflex.eilmer.si`, with the same steps T9 ran locally.

## Context
- Read docs/adr/0004 and 0008; docs/SPEC.md §3 and §13; docs/ARCHITECTURE.md §2 (the `relay` unit), §6 and §8.4; `relay/README.md` and the T9 PR (#10) description.
- What exists on main: everything through T9, T8 and T10. T9 ran only on `wrangler dev` and the Node Relay.
- The address was `relay.eilmer.si` until 2026-11-11. That name already belongs to another project (Lamp T24), so the Relay uses `reflex.eilmer.si`; leave the `relay` record alone.
- `wrangler` on this Mac is already logged in to the user's account with an OAuth token; use it. Don't create new Cloudflare accounts or change billing. If a Cloudflare product needs a paid plan or a dashboard toggle you can't make from wrangler, stop and report exactly what's needed.
- Carried over (docs/TICKETS.md, "Follow-ups"): a Space above about 2,000 strokes doesn't fit in a Worker's 128 MB and gets a 507 pointing to a self-hosted Relay (T9). Keep that behavior; check it on the deployed Worker.
- T7, T13 and T15 run at the same time. You own `relay/` and the Relay URL default; keep other changes small.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.
- Leave the deployment running when you're done; remove any test grants and test Spaces you created on it.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

## Step 6 ticket (iPad side)

```md
# Math tutor on the iPad: LaTeX, Regions and their Check button

## Goal
Make scenario 2 (the math and physics tutor in SPEC §16) work on the iPad, on top of T13's core. When this is done:
- The iPad draws LaTeX Elements crisply at any zoom, next to Ink.
- Regions show on the iPad with their rules: a writing area the person can see, Ink refused where a Region has `ink: false`, and a Region's own Send button ("Check my work") that sends only that Region's work.
- The iPad sends its handwriting recognizer's text and line grouping for the strokes in a Region, both on a Region Send and while watch mode is on, so the Agent's line-level evidence comes from the real recognizer, not from scripted hints.
- `user_still_writing` comes from a live pen-down signal on the iPad, not only from timing.
- The iPad reports which Space is on screen, so `request_attention` can say the person is already looking at it.

Prove it in the iPad Simulator against a real `reflex link` and a real Claude Code session: the Agent places `check-my-work`, you write a solution with a deliberate sign error with the Pencil (or simulated Pencil input), press Check, and see ✓ and ✗ marks and a hint at the exact line; then turn on watch mode and see a snapshot while writing.

## Context
- Read docs/SPEC.md §5 (Regions), §9 (watch mode) and §16 (scenario 2); docs/ARCHITECTURE.md §4.2–§4.3 and §7; docs/adr/0006 and 0007.
- What exists on main: everything through T7 (iPad plan review: Marks, voice, revert chips, Behavior triggers, imports, remote Ink), T13 (math tutor core: Region rules, line-level evidence, watch mode, `check-my-work`, the `math-tutoring` Guide) and T15 (Claude Code channel push). Read `apps/ipad/README.md`, `link/README.md`, `agent/README.md` and the T13 PR (#11) description.
- Carried over (docs/TICKETS.md, "Follow-ups"): watch snapshots use scripted hints for handwriting text, and `user_still_writing` is timing-based (T13); `request_attention` can't report that the person is already viewing that Space (T15).
- T14 (deploy the Relay) runs at the same time and owns `relay/`. You own `apps/ipad/` and the small Link and kernel changes this needs.
- Codex's usage limit is exhausted until 2026-11-12 13:06; prove in Claude Code and say so.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

## Step 6 ticket (adapters)

```md
# Webhook and MCP App: the other two ways a Send can reach an Agent

## Goal
Build the two remaining push Send Adapters on T15's adapter contract (`link/adapters/`), so harnesses other than Claude Code can hear the person without sitting in `wait_for_user`. When this is done:
- **Webhook.** `reflex adapters add webhook --url …` sets one up. The Link POSTs each new Send Bundle to that URL, signed with a per-target secret, with retries and backoff, and the receiver acknowledges it. An Agent SDK program (or any custom harness) can start a turn from it. A webhook that fails or never answers leaves the Send in the Inbox, delivered once.
- **MCP App.** Reflex's MCP server offers a `ui://` view for chat hosts such as claude.ai and ChatGPT. The view shows the Space, and when the person presses Send it starts a new turn in the chat with `ui/message`. It works through the Relay at `reflex.eilmer.si`, for cloud Agents. Find out whether the view can be live or only read-only (SPEC open question) and report it.
- The adapter contract still holds: config, secrets, callback and attempts live in the Link, outside each adapter, and `reflex link status` lists all three adapters' targets.

Prove the webhook with a real receiver and a real Agent SDK (or `claude -p`) program answering a Send it was pushed. Prove the MCP App in a real MCP Apps host: claude.ai with a custom connector to `https://reflex.eilmer.si` if you can reach it, otherwise the MCP Apps reference host, and say which. Kill the receiver once mid-push and show the Send waiting in the Inbox afterwards.

## Context
- Read docs/adr/0002 and 0004; docs/SPEC.md §9 (delivery) and §15 (open questions); docs/ARCHITECTURE.md §2 (Send Adapter row), §6 (Send delivery) and §8.4; docs/harness-notes.md; `link/README.md`, `agent/README.md`, `relay/README.md`.
- What exists on main: everything through T15 (the Claude Code channel and the adapter contract in `link/adapters/`), T13 (math tutor core) and T14 (the Relay live at `https://reflex.eilmer.si`, the default for every Link).
- Carried over (docs/TICKETS.md, "Follow-ups"): `link/test/reapprove.test.ts` and the ask-again check in `link/test/terminal-queue.test.ts` fail unless `REFLEX_REASK_S=0` (T15).
- T12 (math tutor iPad) runs at the same time in `apps/ipad/` with small Link changes; whoever merges second rebases. Redeploy the Relay with `wrangler deploy` from `relay/` only if your change needs it, and leave it running.
- Codex's usage limit is exhausted until 2026-11-12 13:06; after that, also check the webhook path from a Codex session if it fits.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.
- Don't publish anything to npm or a public registry.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

## Step 6 ticket (Embeds)

```md
# Chart Embed: a live chart the person can point at, inside a hardened sandbox

## Goal
Make HTML Embeds first-class on the iPad and the web viewer, with a chart as the first real Embed Template. When this is done:
- **Chart Embed Template.** A chart Embed (bar, line, scatter) lives in the Library. An Agent places it with data in one call, and it draws on the iPad and in the web viewer.
- **Anchor bridge.** The Embed reports named parts (a bar, a point, a series, an axis tick) with their bounds, so a circle or arrow the person draws over the chart becomes a Mark that names the part (for example "bar 2024-Q3"), not just the Element.
- **Bridge API.** `reflex.element.setState(patch)` saves Embed state; it syncs to every device, survives reloads and shows in the next Send's `changes`. `reflex.element.emit(...)` sends "notify agent" events and events that fire the Element's Behaviors.
- **Embeds inside Panels.** A Panel can host an Embed, and the values a person enters in a Panel reach the Agent in the Send Bundle.
- **Sandbox hardening.** HTML Embeds run in a sandboxed WKWebView (and a sandboxed iframe on the web) with no network by default, no access to other Elements or the host, size and CPU limits, and a visible diagnostic when an Embed is blocked. MCP Apps `ui://` resources are accepted as Embed Templates, with upstream tool calls denied unless the person allows that server.
- **Unknown Element types.** An older app that meets an Element type it doesn't know keeps it intact (round-trips it untouched) and draws a placeholder.

Prove it with a real Claude Code session: the Agent places a chart of real-looking data; on the iPad Simulator you circle one bar and press Send, and the Agent answers about that bar; you move a slider in an Embed inside a Panel, and the new value reaches the Agent on the next Send and shows on a second device.

## Context
- Read docs/SPEC.md §5 (Embeds, Panels, `setState`, `emit`) and §13; docs/ARCHITECTURE.md §4.3, §5 (the extension ladder and the bridge API) and §7; docs/adr/0006.
- What exists on main: everything through T12. T10 built the web viewer, `reflex create/test/dev`, `define` and the Library; T7 built Panels and Behavior triggers on the iPad; T8 built Behavior actions; T12 built LaTeX and Regions on the iPad. Read the READMEs in each folder.
- Carried over (docs/TICKETS.md, "Follow-ups"): Embed state changes don't sync yet (T10); the Link drops a Panel's `values` when it hands a Send to the Agent (T7).
- T17 (webhook and MCP App adapters) runs at the same time in `agent/` and `link/adapters/`. It serves Reflex's own `ui://` view; you make `ui://` resources from other servers work as Embed Templates. Whoever merges second rebases.
- Codex's usage limit is exhausted until 2026-11-12 13:06; after that, also check the chart flow from a Codex session if it fits.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

## Step 7 ticket (sharing)

```md
# Shared Spaces: several people drawing on one Space from their own devices

## Goal
Let more than one person share a Space, each on their own devices, as ADR-0008 lays out. When this is done:
- **Invites and roles.** A Space's owner creates an invite with a role (editor or viewer), as a link or QR code. The invited person's device presents it to an existing member's device, directly or through the Relay; that device checks it and records the membership in the signed control catalog. Roles are checked on every received change: a viewer's strokes are refused, an editor can draw and Send, and only the owner can invite or remove.
- **Keys.** Each Space has one key held by every member's devices. An invite wraps it for the new device through an existing member's device; the Relay only carries the wrapped key. Removing a person records the removal and rotates the key, and later changes use the new key, which the removed person's devices never get.
- **Presence.** Live cursors, names and selections travel over the same paths as changes, as short-lived messages never written into the Space or its history. The iPad and the web viewer show who else is here.
- **Sends per person.** Each person's Send carries only their own Marks and Ink, records who sent it (`from`), and their undo stays their own. Agents see each person's name in the Send Bundle.
- **Speed.** Each person's own strokes commit locally first and never wait on the network; people nearby sync over the LAN, people elsewhere through the Relay at `reflex.eilmer.si`.

Prove it with two people: two iPad Simulators paired to two different Links as two different people, one through the LAN and one only through the deployed Relay. The owner invites the second person as an editor, both draw at once and see each other's cursors, each presses Send and a real Claude Code session answers each one by name. Then the owner downgrades them to viewer (their next stroke is refused) and removes them (the key rotates, and their device stops getting new changes).

## Context
- Read docs/adr/0008 (the design), 0001 and 0004; docs/SPEC.md §4 (Spaces and Agents) and §13; docs/ARCHITECTURE.md §6 (identity, grants, presence), §7 (authorization at the writer's frontier) and §8.3 (inviting a person).
- What exists on main: everything through T17. Owners have been people or Agents from T1; person IDs, per-person undo and `from` on every Send exist (T1, T2); received changes are checked (`receive_sync_message_checked`, T6), with roles left for this ticket; pairing and the signed control catalog are in `link/` (T5); the Relay is live (T14).
- Carried over (docs/TICKETS.md, "Follow-ups"): a pairing code cut short in the middle of the person ID was accepted, so a device joined under a person that doesn't exist (T14).
- T16 (Chart Embed) and T18 (3D Embed) run at the same time in `apps/ipad/` and Embed code. Keep your iPad changes to presence, invites and membership UI; whoever merges second rebases. If you redeploy the Relay, leave it running.
- Codex's usage limit is exhausted until 2026-11-12 13:06; after that, also check one Send from a Codex session if it fits.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

## Step 7 ticket (3D)

```md
# 3D Embed: a model the person can rotate, grab by its parts and mark up

## Goal
Make 3D a first-class Embed on the iPad and the web viewer, next to T16's HTML Embeds. When this is done:
- **3D Embed.** An Agent places a USDZ or glTF model in one call (glTF is converted on import; a link or an uploaded file both work). It draws with RealityKit on the iPad and with a WebGL viewer in the web viewer. The person can rotate, zoom and pan it.
- **Part maps.** The model's named nodes become parts with their bounds in screen space. A circle or arrow the person draws over a part becomes a Mark that names it (for example "part bracket_left"), through the same Anchor path T16 built for charts. The Mark records the camera pose at marking time, so the Agent sees what the person saw.
- **Grab parts.** The person can drag a part to move it. The move is saved as Embed state, syncs to every device and shows in the next Send's `changes`. An Agent can set a part's transform, highlight it, or hide it.
- **Import.** A glTF or USDZ file the person adds (Files, drag and drop) becomes a 3D Embed, and so does one an Agent adds.
- **Guide.** `read_me("3d")` explains how to place a model, read part Marks and set parts.

Prove it with a real Claude Code session: the Agent places a multi-part model (an assembly or an anatomy model with named parts); on the iPad Simulator you rotate it, circle one part and press Send, and the Agent answers about that part; you drag a part, and a second device and the next Send both show the new position.

## Context
- Read docs/SPEC.md §5 (Embeds, 3D) and scenarios 3 and 11; docs/ARCHITECTURE.md §4.3, §5 (the extension ladder, Anchors, marking-time evidence) and §7; docs/adr/0006.
- What exists on main: everything through T17 plus T16. T16 built HTML Embeds with the Anchor bridge (`reflex.element.setState`, `emit`, Anchors with bounds), Embeds in Panels and the hardened sandbox; reuse its Anchor and state paths rather than adding new ones. T10 built the web viewer and the Library.
- Carried over (docs/TICKETS.md, "Follow-ups"), from T16: the web viewer accepts an Embed's `setState` without saving it; a refused Panel save on the iPad isn't retried; Embeds on the iPad canvas don't take touches (Embeds in Panels do), which a 3D Embed needs; Embeds show as grey boxes in `look` and in Send images, so render a snapshot of the model there.
- T22 (Shared Spaces) runs at the same time in `link/` and the iPad's presence and invite UI. Keep your iPad changes to Embeds; whoever merges second rebases.
- Codex's usage limit is exhausted until 2026-11-12 13:06; after that, also check the 3D flow from a Codex session if it fits.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

## Step 8 tickets (scenarios)

```md
# Scenarios 3–5: CAD redlines, chart markup and teacher grading, end to end

## Goal
Make SPEC §16 scenarios 3, 4 and 5 work end to end with a real Agent, using only what Reflex already has plus the Guides, Recipes and Embed Templates each one needs (no new tools). When this is done:
- **3. CAD / 3D.** A model next to its feature tree (a Panel or a native list). The person moves a part, redlines a pocket with a written note ("offset 10 mm from outline"), and sketches a dimensioned rib. The Agent gets each mark tied to the part or feature it names, with the written text recognised, and answers or updates the tree.
- **4. Chart markup.** A grid of exploratory charts. The person lassoes outliers ("what are these?"), crosses out a chart, and draws the expected trend on another. Marks come back in data coordinates (the rows lassoed, the trend as data points), and the Agent answers about those rows.
- **5. Teacher grading.** A scanned paper (an image or PDF page) with the Agent's proposed marks as an overlay. The teacher taps to accept a mark, strikes one out and writes their own. Scores and feedback come back against a rubric.
- Each scenario has a Guide (`read_me`) and the Recipes an Agent needs to set it up in one or two calls.

Prove each scenario with a real Claude Code session on the iPad Simulator, from an empty Space: the Agent sets it up, you do what the scenario says, and the Agent answers correctly from the Send alone.

## Context
- Read docs/SPEC.md §16 (scenarios 3–5), §5 and §11; docs/ARCHITECTURE.md §4.2 (Recipes, `define`), §5 (the extension ladder); extensions/README.md.
- What exists on main: everything through T22 and T18. T18 built the 3D Embed with part maps (`read_me("3d")`); T16 the chart Embed with data Anchors; T13 Regions, grading and `check-my-work`; T12 LaTeX and Regions on the iPad; T7 Panels, voice and imports. Guides live in `extensions/guides/`, Recipes in `extensions/recipes/`, Embed Templates in `extensions/embeds/`.
- Carried over (docs/TICKETS.md, "Follow-ups"): Mark IDs are looked up per Agent, not per Space; Marks re-fork the Space once per Mark and nothing caps how many Elements one Mark lists (T13); the T16 and T18 open nits; a refused Panel save on the iPad isn't retried; each device keeps its own 3D camera.
- T20 (scenarios 6–13) runs at the same time and also adds Guides and Recipes. Keep to your scenarios' files; whoever merges second rebases. T11 (peer-to-peer) may also be open in `link/` and the iPad's connection code.
- Codex's usage limit resets at 2026-11-12 13:06; after that, also run one scenario from a Codex session.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

```md
# Scenarios 6–13: the remaining Recipes and Guides

## Goal
Make SPEC §16 scenarios 6 to 13 work end to end with a real Agent, each through Guides, Recipes and Embed Templates (no new tools): UI/HTML prototype markup (marks anchor to DOM nodes), writing/paper markup (the Agent changes only what was marked), morning experiment review (loss curves, an ablation table, tick or kill queued runs), kanji/hanzi practice (guide boxes, stroke order and proportion feedback), sketch critique (corrections on a toggleable overlay, the original never touched, one drill), 3D anatomy learning (rotate a heart, label vessels by Pencil, a quiz), weekly planning/journal (drag time blocks, handwritten journal, the Agent updates a calendar), and a lab plate map (tap wells, "row C contaminated", the notebook entry comes back).
- Each scenario has a Guide and the Recipes or Embed Templates it needs, listed in `read_me("index")`.
- Fix the T10 follow-ups carried here.

Prove each scenario with a real Claude Code session on the iPad Simulator, from an empty Space: the Agent sets it up, you do what the scenario says, and the Agent answers correctly from the Send alone. Where a scenario needs real handwriting or a Pencil-only gesture, use the closest Simulator stand-in and list what's left for a real-iPad pass.

## Context
- Read docs/SPEC.md §16 (scenarios 6–13), §5 and §11; docs/ARCHITECTURE.md §4.2, §5; extensions/README.md.
- What exists on main: everything through T22 and T18 (3D Embed with part maps, chart Embed with data Anchors, HTML Embeds with Anchors and `setState`, Panels, Behaviors, Regions, LaTeX, voice, imports, shared Spaces). Guides live in `extensions/guides/`, Recipes in `extensions/recipes/`, Embed Templates in `extensions/embeds/`.
- Carried over (docs/TICKETS.md, "Follow-ups"): Library entries can't be renamed or deleted yet; `read_me(<type>)` returns `unknown_topic` for Element types listed in the index; example values for a number with maximum below 1 come out as 1; the web viewer says "Offline, retrying" after a 401 (T10). Also the T16 and T18 open nits that touch your scenarios.
- T19 (scenarios 3–5) runs at the same time and also adds Guides and Recipes. Keep to your scenarios' files; whoever merges second rebases. T11 (peer-to-peer) may also be open.
- Codex's usage limit resets at 2026-11-12 13:06; after that, also run one scenario from a Codex session.

## Constraints
- Don't edit SPEC, ARCHITECTURE or the ADRs. If the work shows one needs to change, say so in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

## Step 9 ticket (protocol)

```md
# Protocol v1: the protocol and its conformance profiles, ready to publish

## Goal
Turn Protocol v0 into Protocol v1 that someone outside the project could build a client or Send Adapter from (SPEC §8.3, ARCHITECTURE §12 step 9). When this is done:
- `protocol/` holds a versioned v1: schemas, generated types, one hand-written example per message type, and a written spec covering Elements, operations, Send Bundles, people and Agent identity, shared Space membership, and transports.
- A conformance suite in `tests/conformance/` with named profiles (for example: viewer client, drawing client, Send Adapter, Relay), each a set of fixtures and traces a third-party implementation runs to prove it conforms. The SPEC §16 scenarios supply the fixtures; Reflex's own Link, web viewer and kernel pass every profile.
- Schema `$id`s resolve, or are changed to something that will.
- A release checklist (what to tag, what to publish, where) is written down but not executed.

## Context
- Read docs/SPEC.md §8.3, §16; docs/ARCHITECTURE.md §4, §8, §11, §12; protocol/README.md.
- What exists on main: every ticket T1–T20 and T22. Every scenario (1–13) works end to end; the Send Bundle schema was widened by T19 (`behavior` change kind, anchor `points`) and T20 (the new templates' anchor fields).
- Carried over (docs/TICKETS.md, "Follow-ups"): the schemas' `$id`s point at `https://reflex.eilmer.si/protocol/v0/…`, which now serves the Relay and returns 404 (T14). ARCHITECTURE §4.3 doesn't yet list the `behavior` change kind and the anchor `points` field (T19); §7.2's role check differs from what T22 built; SPEC's CDN sentence differs from T16. `reflex test` has no saved anchor examples, so it can't catch a template returning a field the schema lacks (T20). `npm run typecheck -w @reflex/agent` fails on a Buffer type at `agent/src/model.ts:199`. Codex's usage limit resets at 2026-11-12 13:06; after that, run one scenario from a Codex session.

## Constraints
- **Stop before anything public.** Do not make the repo public, publish to npm or any registry, push tags, create a GitHub release, or deploy anything to reflex.eilmer.si or any other host. Prepare everything, then list exactly what would be published and stop, so the person can decide.
- Never touch the relay.eilmer.si DNS record.
- Don't edit SPEC, ARCHITECTURE or the ADRs. List the doc changes v1 needs in your report.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output.
```

## Step 10 ticket (setup)

```md
# T23: Setup that just works

## Goal
Someone tried to connect a real iPad (TestFlight "Reflex Canvas", bundle si.eilmer.reflex) to a Claude Code session. It took over an hour and never worked. Make first-time setup one command, one scan and one tap, and make every failure along the way say in plain words what to do. When this is done:
1. **One setup command**, from the repo and with no npx: `node agent/bin/reflex.mjs setup`. It puts a `reflex` shim on the PATH (~/.local/bin/reflex). It registers the Reflex MCP server in Claude Code (`claude mcp add`, user scope, with `--channel` push when the session is started with channels) and in Codex (~/.codex/config.toml), when each is installed, without duplicating an existing entry. It makes sure a Link is running: it starts one in the background if none is, so no terminal has to stay open, and `reflex mcp` and the CLI also start one when they need it. Then it prints the pairing QR code and one line saying what to do on the iPad. `reflex pair` shows a fresh QR code at any time, from any terminal, for the running Link. It never uses launchctl for system items, sfltool, keychain writes, or anything that asks for a password.
2. **Approval can't get lost.** The iPad keeps a decision until the Link confirms it (the `agents` list comes back with that status), and sends it again on every new path. The prompt on the iPad stays up while a request is pending, through reconnects and app relaunches. An Agent call waiting for approval keeps waiting, or returns a clear "waiting for you to tap Allow on the iPad" that the CLI and MCP surfaces show to the model and that tells it to call again rather than give up. The `y/N` question in the Link's terminal is shown again when logs bury it. An Agent with the user's shell still can't approve itself.
3. **Quiet, honest logs.** When the Mac at the iPad's paired address answers with a different key, the iPad stops redialing it and shows "This Mac's Link changed. Scan its code again." with the scan button. The pairing code leaves out loopback. Repeated identical refusals are logged once, with a count. A log line always gives the likely cause and the fix. Dialing several addresses no longer closes the losers before the winner's handshake succeeds (T11).
4. **Re-pairing into a fresh Link works with old data on the iPad.** Old Spaces from another person stay on the iPad only and aren't offered to the new Link. The iPad says so once, and offers to delete them. No log spam.
5. **Docs.** In README and apps/ipad/README, "Use it" becomes: run the setup command, scan, tap Allow, done. Add "If it doesn't connect" with the three likeliest causes and their fixes.

## Context
- Read README.md, docs/ARCHITECTURE.md §1 and §6, link/README.md, agent/README.md, apps/ipad/README.md and protocol/spec/transports.md.
- What happened: Agent requests timed out after about 2 min with `not_approved`. Allow was tapped three times and the Link still showed both Agents as pending, because `decide()` in apps/ipad/Packages/ReflexLink/Sources/ReflexLink/LinkSync.swift drops the request and sends `agent_decision` on whatever path is active. The Link logged dozens of "refused a connection … closed the connection during the handshake": the iPad still held a pairing with an older Link identity on the same Mac and redialed it every 2 s, and the pairing code lists 127.0.0.1 and VPN addresses. Old Space `p2p-proof` was refused with "The peer created Element … for someone else".
- The forge is GitHub (`gh`), at https://github.com/jujdontbudge/reflex.

## Constraints
- Never delete or move the person's ~/.reflex: a running Link uses it. Prove with a temporary --home.
- No secrets in commits. Don't touch relay.eilmer.si DNS. Don't redeploy the Relay unless the change needs it. Don't publish anything. Don't turn off the Mac's network. No TestFlight build.

## Evidence of done
From a fresh Link home and a fresh iPad app in the Simulator: run setup, pair, approve, and have a real Claude Code session open a Space and draw a box. Report the time and the steps. Also show that an Allow tapped while the Link connection is dropped (`reflex link fault "lan:drop=1"`, then clear) still lands, and that an iPad paired with an older Link identity shows "scan again" while the Link's log stays quiet.
```

## Step 11 tickets (the new shape: no agent writing)

The person is cutting Reflex down to a native iPad writing app. You write with the Pencil, prompt by voice, and send the current screen to an agent. Agents no longer write on the canvas. T24 and T25 run at the same time. Steps 3 (a UI and UX redesign from the Freeform and Goodnotes research in `docs/research/`) and 4 (a better voice prompt and Send) come after them, in later tickets.

# T24: Strip everything that only served agent writing (Mac side)

## Goal
Cut everything outside the iPad app that existed for agents writing on the canvas or for syncing Spaces. When this is done, the repo holds only what the new product needs. The Link does one job: it receives Sends from the paired iPad and hands them to Claude Code or Codex (`wait_for_user`, Claude Code channel push, webhook). Pairing, agent approval and `reflex setup` (T23) keep working.

Why: in the first version the agent was slow and didn't know how to use the canvas. In the person's words: "everything looks shit and bland. it doesn't understand it at all... it's just way too much freedom it can't control itself." What worked was the input side: the person's marks, words and screen reaching the agent. The person: "i want to totally strip down as much as possible hard. bc with no agent writing a whole part of this is taken out." Less code means the next steps (native ink, the redesign, voice and Send) move fast.

## Context
- First, save today's full version on a branch, `archive/canvas-v0`, pushed to the repo, so nothing is lost.
- From the plan the person approved, these are removed:
  - every way for the agent to write on the canvas: drawing, editing, undo, Recipes, Behaviors, Regions, Embeds, 3D and math layout;
  - shared Spaces and invites;
  - the web viewer and the author kit;
  - the protocol package, the conformance tests and the 13 scenarios;
  - the Relay's encrypted log and cloud agents (claude.ai through the Relay).
- These are kept: pairing, agent approval, a connection to the Mac, and a small Link.
- The person didn't decide whether the Relay stays as a way for the iPad to reach the Mac off the same network. Keep that path working as it does now.
- T25 runs at the same time. It owns everything under `apps/ipad/`. It also removes the Rust kernel and Automerge everywhere, and replaces how a Send travels from the iPad to the Link, which today rides on the kernel's synced document.
  - Leave the kernel and today's Send path in place, so Sends keep arriving from today's iPad app while you work.
  - Both tickets edit `link/` and `agent/`. Whoever merges second rebases onto the other.
- Update README, ARCHITECTURE, SPEC and the package READMEs so they describe the new shape, not the old one.

## Constraints
- Don't edit anything under `apps/ipad/` (T25).
- Nothing public beyond the GitHub repo: no npm publish, tags or releases. Redeploying the Relay at reflex.eilmer.si needs the person's OK. If your change needs a redeploy, say so in the PR and don't deploy.
- Never touch the relay.eilmer.si DNS record.
- Don't run anything that prompts for a password or a keychain approval, and don't turn off the Mac's network.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output. Follow the `agent-testing` skill for all testing.

# T25: Native ink, and the iPad cut down to write, speak and Send

## Goal
Make writing on the iPad feel like Apple's own apps, and cut the iPad app down to the new product. When this is done:
- Strokes stay on the iPad in PencilKit, saved locally the way Apple's own apps do it, and nothing streams. The Pencil only talks to the screen.
- Ink leaves the iPad only inside a Send.
- The iPad no longer uses the Rust kernel or Automerge. They're removed from the repo, along with anything on the Link side that only fed them.
- A Send still reaches the agent over the paired connection with what it carries today: what the person marked, their words, and images of the screen. It goes to Claude Code or Codex through the Link (`wait_for_user`, channel push, webhook).
- Everything on the iPad that showed or handled what agents wrote is gone: agent elements, Recipes, Behaviors, Regions, Embeds, 3D, math layout, and shared Spaces and invites.
- Pairing and agent approval work as T23 left them.

Why: the person's first complaint about the TestFlight build: "it is a bit laggy sometimes i draw with pencil and the pencil drawing comes in too late. a bit unsmooth." Their direction: "fix the pencil lag by making it native local like regular apple apps instead of trying to stream it through a server." Ink that feels instant is the base of the whole product, because the person writes and then sends what they wrote to an agent.

## Context
- The plan promised measured numbers, not "feels better": profile ink on the person's real iPad (iPad13,6, iPadOS 27.2, connected to this Mac) before and after.
  - You can't use an Apple Pencil, so draw with a finger. The person can draw with the Pencil for a final check if you ask.
  - Two research agents drive that iPad through `agent-device`, Freeform now and Goodnotes next, one session at a time. Take your turn, and don't take the device over.
- T24 runs at the same time and strips everything else outside `apps/ipad/`: the agent writing tools, shared Spaces on the Link, the web viewer, the protocol package, the conformance suite, the scenarios, and the Relay's log.
  - You own `apps/ipad/`, removing the kernel and Automerge, and the new way a Send travels from the iPad to the Link.
  - Both tickets edit `link/` and `agent/`. Whoever merges second rebases onto the other.
- The person's current TestFlight build is 202611121334 (app "Reflex Canvas", bundle si.eilmer.reflex). People's existing pages on the iPad were made under the old storage.

## Constraints
- Steps 3 and 4 come later: don't redesign the UI, and don't rework voice or the Send flow beyond what removing the kernel needs.
- Don't build or upload to TestFlight; the parent does that.
- Don't run anything that prompts for a password or a keychain approval, and don't turn off the Mac's network.

## Evidence of done
When you are done, use it yourself the way a user would: run the real app and go through it like they do. If it has no user, use it the way its real client would, such as a program calling the API or an agent calling the tool. Check that it works, that it does everything this ticket asks, and that you hit no other problems. Show what you saw: screenshots, a video, or the output. Follow the `agent-testing` skill for all testing.

# T26: Send shows the agent the whole screen

## Goal
When the person presses Send (or lets go of the mic), the agent sees what the person saw. When this is done, every Send also carries:
- **A picture of the full visible screen plus a margin around it.** It shows the board as it looks (paper, stickies, text, tables, images, shapes, connectors and ink), without Reflex's buttons and capsules, and with about 15% more of the board on every side than fits on screen. The margin has to show what's really there, even though it's off-screen. Keep it a reasonable size for an agent to read.
- **The close-up picture of the recent strokes**, as today: a crop around the strokes since the last Send. When there are no new strokes, the full picture alone is enough.
- **The recent strokes themselves**, as today.
- **Everything else it already sends:** the words and their timing, handwriting guesses, the viewport, the marks, the outbox and `send_ack`.

The agent has to know which picture is which:
- Today the close-up is labelled "What the person saw when they pressed Send", which is false. Label the full picture as the screen plus a margin, and say where the visible screen sits inside it.
- Label the close-up as the recent strokes.
- Link each mark to the image it appears in.
- This applies to `wait_for_user`, the Claude Code channel push and the webhook.

Why: the person's agent kept replying "Each Send only shows me the area around the new marks, so I also haven't seen the handwriting higher up on the page." The person's words: "update to send the full visible screen plus a bit around that as well as the recent stroke picture and recent strokes and the other things it already does."

## Context
- How a Send is built: `send(trigger:said:words:)` and `Crop.capture` in `apps/ipad/Reflex/SpaceSession.swift`. `Crop.capture` snapshots the key window, so it includes chrome and can't see off-screen content. The full picture needs rendering from the board itself, its content view or the PencilKit drawing plus the object layer, so that the margin is real.
- How the Link checks and shapes a Send: `link/storage/sends.ts` (`NativeSend`, validation) and `agent/src/capabilities/native-bundle.ts` (`nativeBundle`, `images`, the `shows` text, mark to image links). The channel writes images to files in `agent/src/surfaces/mcp.ts`, and the webhook is in `link/adapters/webhook.ts`.
- An iPad on the old build may still send a Send without the new picture. The Link must still accept it.
- The person's iPad gets builds through TestFlight. The parent uploads.

## Constraints
- Keep Send fast: rendering must not block the Pencil or the UI noticeably.
- Don't change the outbox, `send_ack` or pairing.
- Don't build or upload to TestFlight.
- Don't run anything that prompts for a password or keychain approval, and don't turn off the Mac's network.

## Evidence of done
Use it the way the person does:
1. Run the app on a simulator, paired to a real Link on this Mac.
2. Make a board with handwriting near the top and stickies or images, some of them just off-screen.
3. Draw a new small mark and press Send.
4. Receive the Send as an agent would (`reflex wait --json` or the MCP tool), and open both images.
5. Show that the full picture has the whole screen and the off-screen margin, with no chrome, and that the close-up and labels are right.

Follow the `agent-testing` skill for all testing.

## Follow-ups carried into later tickets

Found while building earlier tickets. Each goes into the named ticket's brief.

- **T6:** move the iPad's Element drawing from T3's stand-in scene JSON onto the Protocol v0 Element format.
- **T6 (done):** the kernel checks changes it receives from peers (`receive_sync_message_checked`, `kernel/core/src/auth.rs`): an iPad may write only as its person, and a Link also for Agents. It checks against the receiver's state before applying, not at the writer's frontier; ownership never changes, so the two agree for owners. Roles wait for T22.
- **T6 (done, narrowed):** no Reflex command can approve an Agent any more. Only the Link's terminal or an iPad that paired by scanning its code can, and `admin.key` can only deny or revoke. A local process running as the user can still read the Link's files and its terminal; OS-level separation and platform credential stores are left for later.
- **T11:** the fault-injection transport isn't built (T5).
- **T8:** T4 review items left open: Mark IDs are looked up per Agent, not per Space (S3); result indexes are off when a `define` op comes first (S5); a Space can't be named `library` and the error doesn't say why (S7); an `add` with type `ink` or `file` reports a missing type (S8).
- **T8:** `read_me` and `recipe` aren't built, but the server instructions tell Agents to call `read_me` first.
- **T8:** the CLI adds a `file` path to images that the schema has no field for. Agent Element IDs show to other Agents as `agent:id`, because the protocol's ID pattern doesn't allow `/`.
- **T2 follow-ups for the kernel owner (T8):** the state digest covers Send IDs but not their contents; moving a connector is accepted but changes nothing; undo, redo, undo in the same millisecond can hide a restored Element; concurrent undo and a new action on two of one person's devices can make redo replay an older entry. Text measurement, Mark grouping, revert planning and migrations aren't built.
- **T16:** Embed state changes don't sync yet (T10). **T20:** Library entries can't be renamed or deleted yet; `read_me(<type>)` returns `unknown_topic` for Element types listed in the index; example values for a number with maximum below 1 come out as 1; the web viewer says "Offline, retrying" after a 401 (T10).
- **T15:** tap actions on a revoked Agent's Elements still run its `set` and `apply_recipe`, because the runner doesn't check approval; `wait_for_user(element:)` misses taps whose action is `send`; `request_attention` never reports that the person is already viewing that Space (T8). **T13:** nothing caps how many Elements one Mark lists, and Marks re-fork the Space once per Mark (T8).
- **Codex:** its usage limit ran out until 2026-11-12 13:06, so T8's scenario 1 was proven in Claude Code only; recheck in Codex in a later ticket.
- **T9 (done):** a Space above about 2,000 strokes doesn't fit in a Worker's 128 MB (the kernel loads the whole document, peaking near 3x its size). The Worker answers 507 and points to a self-hosted Relay. Moving cloud execution to a separate service (ADR-0004) or loading less in the kernel is open.
- **Real iPad pass:** three-finger redo, Pencil double-tap and squeeze. Native WWDC26 stroke IDs need Xcode 27 (the app targets iPadOS 26 for now).
- **T12 (done):** watch snapshots take handwriting text from the iPad's recognizer and `user_still_writing` from its live pen signal; `request_attention` answers `already_viewing` from the Space the iPad reports on screen. Left for a real iPad pass: the pen signal and Vision on real Pencil handwriting (the Simulator run used outline-font strokes).
- **T16:** the Link drops a Panel's `values` when it hands a Send to the Agent; they are in the frozen Send but not in the Bundle (T7).
- **T17:** `link/test/reapprove.test.ts` and the ask-again check in `link/test/terminal-queue.test.ts` fail unless `REFLEX_REASK_S=0` is set, on main too (T15).
- **Declined review nits, open:** with more than 300 history entries a revert chip can miss an older Agent change; the iPad doesn't enforce the schema's size limits; remote Ink isn't refetched after a lasso move on another device (T7). The no-Ink check counts a stroke by its center; an explicit stroke list on a Region Send isn't limited to the Region (T13).
- **Real iPad pass:** live mic transcription, camera capture, and drag-and-drop from another app (T7).
- **T22:** a pairing code cut short in the middle of the person ID was accepted; the new device joined under a person ID that doesn't exist (T14).
- **T21:** the protocol schemas' `$id`s are `https://reflex.eilmer.si/protocol/v0/…`, and that host now serves the Relay, so they return 404 (T14). Publishing the protocol should serve them or change them.
- **Declined review nit, open:** lifting the pen after the app goes to the background can report the Space as still on screen until the next update (T12).
- **Real iPad pass:** the live pen-down signal and recognition of real handwriting (T12).
- **T17 (done):** the reapprove and terminal-queue tests pass without `REFLEX_REASK_S=0`. **Open nits (T17):** a webhook URL removed and added again doesn't push Sends still pending from before; two narrow cases in passing a held Send's acknowledgement to the Link, if the Link restarts or a deadline lands during that call. Whether claude.ai or ChatGPT let an MCP App view hold a 25 s call isn't measured; the MCP App was proven in the MCP Apps reference host only.
- **T18 (from T16):** the web viewer accepts an Embed's `setState` without saving it; a refused Panel save on the iPad isn't retried; Embeds on the iPad canvas don't take touches (Embeds in Panels do); Embeds show as grey boxes in `look` and in Send images. **Open nits (T16):** an Embed whose navigation was blocked can escape the busy-loop check; a stale Library copy of a built-in template isn't refreshed when the built-in's summary changes; `reflex dev` matches strokes by position rather than by ID; `reflex test` examples don't save the Mark's target; an allowed MCP server is remembered per iPad, not per person.
- **SPEC decision (T16):** SPEC says approved CDNs are allowed by default; T16 allows CDNs for loading code only and blocks every other network call unless the Embed's allow list names the host. Update the SPEC sentence.
- **T22 (done):** a pairing code cut short inside the person ID is refused, on the Link and the iPad (T14). **Open (T22):** people share only through the owner's Link (members' Links don't sync directly); Agents of someone a Space is shared with can't be approved for it; the web viewer shows others but doesn't report its own cursor; key ordering between one person's two Links; a Send's image arrives late on the owner's side, so the Bundle uses a re-rendered image; a one-time "refused a connection … not valid JSON" when two Links dial each other through the Relay at once; one more presence test the reviewer suggested.
- **ARCHITECTURE decision (T22):** §7.2 checks roles against the writer's own history; T22 checks against the receiver's newest share record, so edits made offline before a downgrade are refused when they arrive. Add a note to §7.2 or make it a follow-up.
- **Build note:** the iPad app's kernel xcframework has no x86_64 Simulator slice, so `generic/platform=iOS Simulator` builds fail to link; build with `ARCHS=arm64` or a named Simulator.
- **T18 (done):** `look` and Send images draw 3D models. **Open (T18):** a refused Panel save on the iPad isn't retried (T16); on the iPad, model textures, skins and animation aren't read and a `.gltf` must embed its buffers; parts move only on the iPad and only by position; each device keeps its own camera, so a circle drawn on the iPad doesn't line up in the web viewer; `look` shows small glitches on some faces and a USDZ as a placeholder; a metallic part looked off-colour after a drag. Declined nits: an `http://` link gets a misleading error; the web viewer lets a Mark name a hidden part; an Element on top of a 3D Embed doesn't get the touch; Ink in Send images is misplaced when zoomed.
- **T11 (done):** peer-to-peer Wi-Fi is a second path between the iPad and the Link, proven on a real iPad with no shared network; the fault-injection transport is built (`lan:drop=1`, `p2p:drop=1`). **Open (T11):** dialing several addresses at once closes the losers before the winner's handshake succeeds; a Link that pairs into another person stays in its old Relay room until restart (T9); the 1 ms minimum gain keeps a 2 ms loopback Relay over a 1.1 ms LAN, so local paths may deserve preference; turning off the Mac's Wi-Fi service or radio also takes peer-to-peer down (only leaving the network keeps it). Nits: `const s` shadows the session in the `spaces` case; `Object.hasOwn` in the fault parser; `p2p_port` reports 0 instead of null; the iPad's `transports` and `activeTransport` are published but unread.
- **Real iPad pass (still open):** native PencilKit stroke IDs, three-finger redo, Pencil double-tap and squeeze, mic, camera, drag-and-drop, the pen-down signal and real handwriting. They need a person at the iPad. Debug builds now keep the screen awake while Reflex is in front.
- **T19 (done):** scenarios 3–5 (CAD review, chart markup, teacher grading) work end to end with a real Agent from an empty Space using only Guides, Recipes and Embed Templates. The fixes on main: the Link offers new assets to connected devices right away, a line from an Element onto one inside it strikes the inner one, the iPad draws Markdown headings and bullets, and the chart and grading Guides are corrected. **Open (T19):** dragging a 3D part moves it on a camera-facing plane, so it gains a vertical component; writing on a model targets the model, not the nearby circled part; a vertical line on a chart returns rows but no `points`; a lasso with writing over it comes back as one writing Mark; on the Simulator, two debug strokes more than 1.5 s apart come back as two strikes, not a cross; `read_me("review-legend")` shows its Params and Template headings twice; decide whether untapped grading proposals count as accepted. **Doc decision:** ARCHITECTURE §4.3 should list the new `behavior` change kind and the anchor `points` field.
- **T20 (done):** scenarios 6–13 run on Guides, Recipes and Embed Templates (prototype, document, table, plate, overlay, guide-box; the built-in `builtin:heart` model), each proven on the iPad Simulator in a real Claude Code session from an empty Space. Fixed on the way: Library rename and delete (`reflex link library`), `read_me(<type>)` for every indexed type, example numbers under a maximum below 1, the web viewer's "Key not accepted" after a 401, Region Sends without Embed anchors, flat ovals not read as circles, strikes across part of an Embed read as ink. (T19 fixed the same asset-offer and iPad Markdown bugs on main; T20 confirmed both.) **Open (T20):** HTML Embeds are still grey boxes in `look` and Send images (Agents say so); snap is iPad only (the web viewer drags freely); a stale Library copy of a built-in template is never refreshed, so an Embed fix reaches a person only after they delete the copy (`reflex link library delete`); the Simulator's speech recognizer hung, so scenario 13's "row C contaminated" was handwritten. **Real iPad pass (T20):** voice for scenario 13, real Pencil handwriting for kanji (9) and labels (11), a finger turning the heart (11, Simulator flings didn't reach it), and dragging blocks by finger (12).
- **Open (T20, review round 2):** a snap target owned by someone other than the block's owner gets the wrong ID prefix; `reflex test` has no saved anchor examples, so it can't catch a template returning a field the schema lacks; `npm run typecheck -w @reflex/agent` fails on a Buffer type at `agent/src/model.ts:199` (also on main). The document template's `line` field is now `gesture`, so devices with an old Library copy keep sending `line` until it is deleted. Declined: a Region Send asks every Embed on the canvas, not just those inside it; renaming a Library entry to its own name errors.
- **Moved to GitLab (2026-11-12):** GitHub suspended the account, so the project now lives at https://gitlab.com/johansardar/reflex (public), with merge requests in place of pull requests (`glab mr`). Old GitHub PR links in this file are historical.
- **Moved to GitHub (2026-11-16):** the project now lives at https://github.com/jujdontbudge/reflex (public), with pull requests (`gh pr`) in place of merge requests. Every branch and tag came across at the same commit. GitLab merge request links in this file are historical; the GitLab project stays as a read-only backup.
- **T21 (done):** Protocol v1 (1.0.0) is in `protocol/` with a written spec (`protocol/spec/`), six new schemas, an example for every message, a CHANGELOG and `protocol/RELEASE.md`. The conformance suite is in `tests/conformance/`: five profiles, a JSON-lines driver for any language, and the 13 scenarios as fixtures. Reflex's own kernel, web viewer, Link, webhook adapter and both Relays pass every profile. Fixed on the way: a `[` on a document read as a self-arrow; guide-box handwriting split across lines in Region Sends; the Workers Relay never sent offline presence. **Nothing is published:** the npm package, the tag, the GitLab release and the Relay deploy that makes the `$id`s resolve wait on the person's decisions in RELEASE.md (license, the `@reflex` npm scope, whether the kernel ships compiled). **Open (T21):** the kernel accepts any Agent ID though the schema requires `a_…`; frozen Sends name Elements `claude/block-gym` while scenes use `claude:block-gym`; the Send Bundle lists a `bracket` Mark kind the kernel never produces; the Link never runs the author kit's Send Adapter extension modules; a request ID reused in another Space can still hit `request_reused` through the Library; the web viewer drew nothing after about 13 page loads in one tab; the iPad's kernel xcframework needs rebuilding for the two kernel fixes; run one scenario from Codex once its limit resets (2026-11-12 13:06); the round 2 review fixes weren't re-reviewed. **Doc decisions:** ARCHITECTURE §4.3 (`behavior` change kind, anchor `points`), §7.2 (role check as `link/sharing.ts` does it), §10 (conformance layout), §11 (versioning, and why wire strings keep `v0`); SPEC's CDN sentence (T16) and a note in §14.
- **T23 follow-up:** iPad builds from before T23 don't send `requested_at` with `agent_decision`, so the Link can't tell their answers from answers to a later request and takes them as they are (link/sync/coordinator.ts). Once every iPad in use runs a T23 build, require `requested_at` and drop answers without it.
