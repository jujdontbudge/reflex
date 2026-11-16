# Reflex: Spec

Status: 2026-11-14, written from the grill on the new product shape.

## 1. Goal

Showing an AI coding agent what you mean is easier by hand than by typing. You can circle the broken button, sketch the flow, and say "this one, but blue".

The first version of Reflex let agents draw on the canvas too, and that went badly. The person's verdict: the agent was "slow to use and it doesn't know how to use it well. everything looks shit and bland. it doesn't understand it at all... it's just way too much freedom it can't control itself." What worked was the input side: the person's marks, words and screen reaching the agent.

Reflex is now a native iPad board app for the person. You write and draw with the Pencil and place sticky notes, shapes, text, tables and images on endless boards. You hold the mic to say what you want, then send the current screen to an approved agent: Claude Code or Codex on your Mac. The agent answers in its own session. It never writes on your board.

It is for people who direct coding agents and think better with a pen. It feels like Apple's Freeform, but every screen leads to one action, Send.

## 2. Principles

- **The board is the person's.** Agents never draw, edit or delete anything on it. Too much freedom made the first version's agent useless; the person's input is the product.
- **Native, like Apple's own apps.** Ink appears under the Pencil and stays on the iPad. It never streams anywhere while you write, and leaves the iPad only inside a Send. The person asked for ink "native local like regular apple apps instead of trying to stream it through a server."
- **Apple-native look.** SF Symbols in place of the old hand-drawn icons, the system's glass materials and colours, light and dark mode, portrait and landscape. The person called the old hand-drawn icons "pretty bad."
- **Freeform's way of working, with its weak spots fixed.** The person wants it "more like freeform UI UX wise". Where Freeform hides things or loses work, Reflex doesn't.
- **Use Apple's tools before building our own.** Apple's tool picker brings the pens, colours, ruler, lasso, and the Pencil Pro squeeze and double-tap for free, and it's what Freeform uses.
- **Strip hard.** Anything that doesn't serve writing, organizing, speaking or Send goes.
- **The person decides which agents get their Sends.** Pairing never approves an agent by itself.

## 3. What it does

### Boards and the board browser
- A Board is an endless canvas. Each board reopens where you left it.
- The browser shows All Boards, Recents and Favorites, with nested folders. View them as a grid or a list, sorted by date or name, with optional date groups.
- Search finds boards by title and by typed text on them.
- A board you leave empty is thrown away, so the browser never fills with blank boards.

### The board's chrome
- A clean canvas with small glass button groups at the edges, and very few controls at once.
- The insert tools fold into one "+" when space is tight.
- A zoom menu runs from 10% to 400%, with Zoom to Fit.

### Writing
- Apple's tool picker holds the pens, highlighter, colours, eraser, lasso and ruler. Reflex's own tools (shapes, sticky notes, text) are added to the same palette.
- The palette stays open until you close it, docks on the right or the bottom, and remembers a colour for each tool.
- Pencil Pro squeeze and double-tap work as they do in Apple's apps. You can also draw with a finger.
- Ink is instant and stays on the iPad.

### Paper
- Each board has a background: blank, dots, lines or squares, each in three spacings, on a few paper colours including a dark one.
- All of it is set from one popover.

### Things on a board
- Sticky notes, text boxes, tables and images (pasted, from Photos, from the camera, or from Files).
- The 12 basic shapes, plus a searchable library of SF Symbols.
- Connectors, dragged straight from handles on a selected object.
- Every object gets the same small toolbar: two to four buttons for its type, then Duplicate, Delete and a short "…" menu, with Bring to Front and Send to Back at its top.

### Selecting and arranging
- Select several things at once with the lasso.
- Alignment guides for centres, edges and spacing, plus snap to grid, each with its own switch.
- Group, lock, and change the order.
- Undo and redo work per board, and are kept when you leave and come back.

### Scenes
- Save a view of the board as a Scene. Step through Scenes with ‹ ›, and export them as PDF.

### Voice and Send
- A mic button and a Send button belong to the board's design.
- Send captures the current screen, with everything on it, together with the person's marks and spoken words. It goes to the approved agents.
- A Send made while the Mac is out of reach waits on the iPad and goes out once the Mac is back.

### Pairing and connection
- On the Mac, `reflex setup` gets everything ready and shows a pairing code. You scan it once on the iPad.
- The first time an agent connects, the iPad asks whether to allow it. Pairing never approves an agent by itself.
- A small status capsule in one corner shows whether the Mac is connected and which agents are approved. Tapping it opens pairing and approvals. The pairing sheet appears only the first time.

### Delivery on the Mac
- An approved agent gets Sends by waiting for the person (`wait_for_user`), as a Claude Code channel push that starts a new turn, or as a signed webhook.
- Sends are kept until the agent acknowledges them, including across restarts.
- An agent whose approval is revoked stops receiving Sends.

### Connectivity
- The iPad reaches the Mac over the same Wi-Fi network, or over Apple's peer-to-peer Wi-Fi when they are near each other.
- An optional Relay forwards encrypted traffic between them when nothing closer works. It holds no boards and runs no agents.

## 4. Scenarios

**Fixing a screen in the app you're building.**
1. You screenshot the app on your iPad and paste it into a board.
2. You circle the misaligned button in red, hold the mic and say "this should line up with the search field and be blue".
3. You press Send.
4. Claude Code, running in your terminal with Reflex connected, gets a new turn holding the screen, your circle and your words, and fixes the button.

**Sketching a plan before the agent builds it.**
1. On a board with a dotted background, you lay out sticky notes for each screen and drag connectors between them. You add a quick hand-drawn sketch of the main screen.
2. You zoom so the whole flow fits, say "build this flow, starting with the login screen", and press Send.
3. Codex gets the screen and your words.

**Setting it up the first time.**
1. You run `reflex setup` on the Mac.
2. You install Reflex on the iPad and scan the code it shows.
3. Claude Code connects, the iPad asks whether to allow it, and you tap Allow.

**Keeping boards in order.**
1. You keep a folder per project and star the boards you use daily.
2. You find last week's board by typing a word you typed on it.
3. Boards you opened and left empty never show up.

## 5. Names

The key words are defined in [GLOSSARY.md](../GLOSSARY.md):
- **Board**: an endless canvas where the person writes, draws and places things.
- **Send**: one deliberate message of the person's screen, marks and words.
- **Mark**: a group of the person's ink strokes.
- **Person** and **Agent**.
- **Link**: the Mac companion.
- **Relay**, **Inbox**, **Pairing** and **Agent approval**.

## 6. Stack

- **A native iPad app built on Apple's PencilKit and its tool picker,** so ink feels like Apple's own apps and the Pencil features come for free.
- **SF Symbols and the system's look,** which is Apple-native and replaces the hand-drawn icons.
- **The Link on the Mac,** connecting to Claude Code and Codex. Both are the harnesses the person uses.
- **TestFlight** to put builds on the person's iPad.
- **GitHub** (github.com/jujdontbudge/reflex) for the code.

## 7. Drafted

Drafted: this shows intent. The architecture decides.

- A Send travels from the iPad to the Link as one message after its images. The Link stores it once, keyed by its ID, and acknowledges it. The iPad keeps it in an outbox and retries until it gets that acknowledgement.
- Handwriting search could use the stroke recognizer in Apple's iPadOS 27 SDK. That needs a released Xcode 27 to ship through TestFlight.

## 8. Not doing

- **Agents writing on the board.** It made the first version slow, bland and uncontrollable. The earlier version is kept on the branch `archive/canvas-v0`.
- **Neater handwriting (Auto-Refine).** Apple's Smart Script works only in Apple's own apps. Neither the released SDK nor the iPadOS 27 beta lets other apps turn it on, and building our own would eat the whole effort.
- **Handwriting search, for now.** It waits for the iPadOS 27 API to be shippable. Search covers titles and typed text until then.
- **Sharing boards and iCloud sync.** Boards stay on the iPad. Both would bring back the sync machinery that was just removed.
- **Premium content and AI image generation**, as in Freeform's Content Hub.
- **Notebooks of fixed pages, Goodnotes-style.** Endless boards fit "send the current screen" and the Freeform-style work.
- **Cloud agents** such as claude.ai through the Relay, shared Spaces and invites, the web viewer, the author kit, the protocol package and its conformance suite.

## 9. Open questions

- **Keeping the Relay.** Whether it stays as the off-network path at all. Without it, the iPad can send only when it can reach the Mac directly.
- **Typed text in a Send.** Whether a Send also carries the typed text of sticky notes and text boxes as text, not just inside the screen image. This belongs with the later work on voice and Send.
- **The later voice and Send work.** What a better voice prompt and Send look like beyond where the buttons sit and how they look.

## 10. Notes from the grill

- **How the redesign is judged.** A fresh Opus 5.5 judge uses the app and compares it with Freeform (screenshots and the map in `docs/research/freeform/`) and with how it ideally would be.
  - The judge scores 1–10, each with a reason, the gap to ideal and a ranked fix list, on:
    - the board browser and organizing;
    - the board chrome and look;
    - the writing tools and switching;
    - paper and backgrounds;
    - the things on a board;
    - selecting and arranging;
    - how the mic and Send look and where they sit;
    - pairing and connection status;
    - overall feel.
  - An Opus 5.5 builder then fixes relentlessly, tests by using the app, and the next judge re-rates. That's three fix rounds and four ratings, so the last fixes get judged too.
  - One builder carries through all three rounds. Each judge is new and never sees the builder's notes. Round 3's judge also uses the person's real iPad if it's free.
  - The person gets a TestFlight build after the final rating.
- **Goodnotes.** The person admired Goodnotes' pen and paper settings and the way it switches tools, which is why the paper options go beyond Freeform's. Its in-depth research was skipped.
- **The person's other complaints about the first version.** Pencil lag ("the pencil drawing comes in too late"), and "a bunch of syntax and sizing issues that are weird" in what the agent drew. Removing agent writing ends the second.
- **A later step** puts a lot of effort into making the voice prompt and sending the current screen better.
