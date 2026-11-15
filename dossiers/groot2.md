# Reconstruction dossier — Groot2 (target #14)

> Worked from the kickoff (issue #9) and the verified findings of its work
> packages 1–3 of 2026-11-13. ESTABLISHED = observed on the reference
> environment or cited from a public source; NEEDS-OBSERVATION = the next
> fixture grade. A dossier is a research record, not an engineering spec.

- **Target:** Groot2 (Auryn Robotics) — the proprietary integrated development
  environment for the open-source BehaviorTree.CPP behavior-tree ecosystem
- **Status:** research · **Last updated:** 2026-11-15 (§E landscape corrected: open Groot2-protocol clients, including breakpoint support, already exist)
- **One-line objective:** an independent Groot2-*compatible* behavior-tree IDE
  that unifies the three lives of a tree — authored (`.btproj`), executing
  (ZMQ monitor), and recorded (`.btlog`) — over the publicly specified
  formats, with log replay, live monitoring, and breakpoints/fault injection.
- **Kickoff:** issue #9 — "Reconstruction kickoff: Groot2 (target #14)".
- **Compatibility-surface study:** [`docs/groot2-compat-surface.md`](../docs/groot2-compat-surface.md)
  (the pinned byte-level contract, with source citations).

## First verified findings (2026-11-13)

All facts below were established 2026-11-13 by observing the vendor
application's own behavior on self-authored and vendor-shipped data, and by
cross-referencing it against the BehaviorTree.CPP 4.9.0 open-source
specification of the interface the IDE consumes. The reference version is
**Groot2 v1.10.0** run on its free tier: the full editor, with the monitor
and log-visualizer capped at **20 nodes**; the PRO tier (€590/yr) adds
breakpoints, fault injection, blackboard visualization, and search.

**The compatibility surface is ~90% publicly specified.** The open-source
BehaviorTree.CPP pins, byte for byte, everything a compatible IDE must
interoperate with: the `.btlog` grammar, the ZMQ live-monitor protocol, the
XML tree dialect and `TreeNodesModel`, the SQLite `.db3` schema, and — now
verified bidirectionally — the `.btproj` save-shape, whose exact serializer
delta is pinned below (work package G-1, 2026-11-13). The compatibility-
surface study on this branch cites each of these against the 4.9.0 source and
against an independent open re-implementation of the protocol as cross-check.
What remains proprietary-only is small: the free-tier feature gating, and
product/UX behavior.

### WP1 — the vendor sample corpus (15 files, provenance recorded)

- All 15 files embedded in the v1.10.0 binary parse cleanly per the
  documented BT.CPP 4 XML grammar; all `<include>` references resolve.
- **`BTCPP_format` is advisory in practice, not a hard gate.** One shipped
  file omits it, and at BehaviorTree.CPP 4.9.0 a missing attribute produces
  only a warning and no value check exists — a file declaring `"3"` parses
  untouched. (This corrects an earlier kickoff claim that recorded it as a
  gate; the compatibility-surface study carries the correction.) Only the
  emitted XSD marks the attribute required. A compatible implementation
  should treat it as metadata, and the vendor's own samples show it is never
  re-asserted on save.
- **Three shipped files throw at instantiate under 4.9.0** — a regression
  *between vendor generations* (they loaded in the earlier generation the
  samples target). Recorded as fixtures, not as a defect of either side.
- The v3/v4 sample pairs provide **one-attribute delta fixtures**: minimal,
  side-by-side pairs that isolate the exact format differences the IDE's
  compatibility mode must absorb.

### WP2 — spec-conformance tooling

A `.btlog` writer/reader/verifier was built strictly from the pinned public
grammar and round-trips against the 4.9.0 writer (**PASS**, 2026-11-13). Five
grammar refinements were then verified against the 4.9.0 source:

- 18-byte magic string, `BTCPP4-FileLogger2`, ASCII, no NUL;
- **status `4` = SKIPPED** (the `NodeStatus` enum value, written verbatim —
  no idle-remembrance encoding in logs, unlike the live-monitor status
  buffer);
- transition timestamps are the **low 48 bits** of the relative microsecond
  offset (≈5.4 years of uptime before truncation);
- the file is **always little-endian**, on all supported targets;
- the file-logger's header **doc-comment is stale** against its writer — the
  writer, not the comment, is the authority.

### WP3 — interop proofs (the headline)

1. **The vendor's own v4 project loads.** A v4 `.btproj` authored in Groot2
   opens and renders in full: all **7 trees**, all **10 custom node models**,
   graph rendering intact — PASS.
2. **A log written purely from the public grammar plays back in the
   proprietary IDE.** A `.btlog` generated with no reference to the vendor
   application loaded and played back: **16 transitions decoded, timestamps
   exact, playback UI functional** — PASS. Getting there surfaced a recorded
   behavior of the IDE itself: its strict **port-vs-model validation**
   rejected a `Script` node whose port name did not match the model's
   declared port (`code`) — the IDE validates instance ports against the
   model **at log-load time**, which is a compatibility surface of its own.
3. **The v3 project loads behind a documented mode switch.** Presenting the
   v3 project to the v4-default IDE offers a **"Switch to V3"** action on
   mismatch; on acceptance the project is retitled **"(V3 Compatibility)"**
   and a warning advises closing the project before switching. PASS — with
   the mode-switch behavior recorded as a compatibility surface in its own
   right.

### G-1 — the `.btproj` save-shape, pinned by save-and-byte-diff (2026-11-13)

Work package G-1 is **DONE** (2026-11-13). Save-and-byte-diff against the
banked sample corpus — save-as from the reference IDE to new paths,
same-directory controlled copies, originals re-hashed untouched after —
establishes the following.

**Serializer delta.** In **V4 mode** the reference IDE unconditionally adds
`BTCPP_version="4.10"` to the `<root>` tag on save; in **V3 mode** it adds
nothing. The attribute's *placement* depends on the file type: in a
`.btproj` (project file) it is inserted **inline** on the same line as the
existing attributes; in a tree XML file it goes on its **own indented
continuation line**. Include paths are rewritten **relative to the saved
file's own location** (bare filename when saved alongside, relative path
otherwise). Everything else round-trips **byte-exact** — the
`<TreeNodesModel>` declaration block, comments, attribute order, indentation,
and XML escaping — and the **V3 in-place round-trip is byte-identical** to
the input.

**Two structural findings.** (1) The IDE only ever saves **projects**:
a single opened tree is not rewritten in place on Save As — it is wrapped as
a 1-include `.btproj` — and a bare tree XML is rewritten only when its
*content* has been modified in the editor. (2) **Node layout is not
persisted**: tree-node canvas coordinates are not stored in either file type
and are recomputed on load, so a save-and-reopen cycle is layout-neutral.

The evidence (all saved artifacts, byte-for-byte, with the full method and
per-file diffs) is banked in the private campaign repository; the dossier
records the established result here, per the public/private wall.

### G-2 — the recording-container question, resolved (2026-11-13)

Work package G-2 is **DONE** (2026-11-13), and it removes the single largest
remaining format risk for a log-replay replacement: **the reference IDE's
monitor recording is the plain public `.btlog` format, end to end.** The
zstd-container hypothesis is **closed for the log format** — the zstd
bundled in the AppImage serves something else in the application, not the log
file.

**Method.** A **live recording was captured from the reference IDE itself**:
Groot2 v1.10.0 (free tier) connected as a live client to the spec-conformance
monitor server (the same spec-built server that proved the G-4 interop), with
the known-content scenario animating in the monitor. The IDE's own Record
control was toggled on and off; its dialog-driven save produced the recording
on disk. The IDE's writer was thus **observed, not inferred**.

**ESTABLISHED — the recording is a plain spec-conformant `.btlog`:**

- **The recording's own save dialog declares the format.** The Record toggle
  opens a "Save Log file as…" dialog *before* recording starts, and the
  dialog's file-type filter itself names the format: `FileLogger2 Log file
  (*.btlog)` — the FileLogger2 logger of BehaviorTree.CPP, unmodified.
- **A captured recording parses with zero errors by an independent reader.**
  A reader built **purely from the pinned public grammar** (the campaign's
  own `.btlog` tool, per the 4.9.0 FileLogger2 source) read the captured
  recording end to end: **196 transitions, zero parse errors**, file ends
  exactly on a record boundary. Its embedded tree XML **matches the served
  tree exactly**, and the transition timestamps are monotonic within a tick
  (~200 µs apart) across the scenario's ~3-second cycles over roughly ten
  seconds of live monitoring.
- **No container bytes.** The file begins with the 18-byte ASCII magic
  `BTCPP4-FileLogger2` at offset 0 — the 19th byte is the protocol version,
  and the 4-byte little-endian XML length follows at offset 19 — then the
  tree XML, then the binary transition records. No compression magic
  anywhere: the plain grammar is intact from offset 0.

**Record-flow behavior (compatibility-relevant, observed).** A compatible
recording implementation should match the IDE's flow as well as its bytes:

- the **Record toggle opens the save dialog before recording starts** — the
  destination path is chosen first;
- the chosen file is **created at 0 bytes and fully buffered during
  recording**, then **flushed atomically when Record is toggled off** — a
  file on disk mid-recording is empty by design, so incremental or partial
  files are not the reference behavior.

The recording artifact, its hash, the first-bytes analysis, the reader
output, and the full method are banked in the private campaign tree; the
dossier records the established result here, per the public/private wall.

### G-3 — the breakpoint / fault-injection surface, mapped (2026-11-13)

Work package G-3 is **DONE** (2026-11-13): the breakpoint and
fault-injection behavior of the reference IDE's two flagship PRO features
is **ESTABLISHED as observed behavior**, verified against the
spec-conformance monitor server with correlated server-log evidence.

**Route to the PRO tier:** the vendor's own **free 30-day PRO trial** —
the reference IDE's PRO activation was started from the vendor's own
trial flow, a plain reference-license route with no circumvention. That is
what G-3 needed.

**Method.** The reference IDE — **Groot2 v1.10.0, PRO tier** — was
connected as a live client to the same spec-conformance monitor server
(the server built purely from the pinned public specification that proved
the G-4 interop), with the known-content scenario animating. Breakpoints
and fault injections were armed, re-armed, cleared, and removed through
the IDE's UI, and every wire message was correlated against the
server's own request/PUB log for the whole session (a full census over
~90,000 log lines: six request types and one PUB message kind were
observed — the IDE's complete wire vocabulary in this scenario).

**ESTABLISHED — the breakpoint / fault-injection wire behavior:**

- **Arming any hook** (breakpoint or fault injection) sends a
  **`HOOK_INSERT`** request on the REQ/REP socket **immediately on
  "Hook Config … OK"** — before the armed node ticks.
- **A breakpoint (interactive mode) additionally publishes
  `BREAKPOINT_REACHED` on the PUB socket exactly once, at hook-insert
  time** — the publisher-side publish the open-source protocol specifies.
  Against the spec-conformance server, **the tree does not pause**: the
  reference IDE keeps animating, with no halt, dialog, or request stall.
- **The IDE never sends a `BREAKPOINT_UNLOCK`** — over the whole
  session the IDE's vocabulary was FULLTREE, STATUS,
  GET_TRANSITIONS, HOOK_INSERT, HOOK_REMOVE, TOGGLE_RECORDING, with
  the single `BREAKPOINT_REACHED` PUB kind; no unlock request appeared.
  The protocol's interactive halt-unlock cycle was therefore **never
  exercised by the IDE against a spec server**. An independent
  implementation's breakpoint semantics are **server-authoritative per
  the open-source protocol**: whether an armed breakpoint pauses the
  tree is a decision the *server's* pre-tick callback makes, and the
  reference client behaves as though that callback never blocks.
- **Fault injection = Replace Node (Success | Failure):** a
  `HOOK_INSERT` carrying the node-replacement mode, **silent on the PUB
  socket** (no `BREAKPOINT_REACHED` for it). In the UI the node shows a
  ⊗ marker and renders FAILURE red each cycle with red dashed
  propagation to the parent — a fault with **no distinct wire message**;
  only the server-side status records differ.
- **The "Once" checkbox is invisible on the wire** — the insert carries
  no distinguishing field, and the IDE does not retire the entry itself;
  any fire-once enforcement is **server-side**.
- **Removal happens only via `Hook Config → None → OK`**, which sends
  **`HOOK_REMOVE`**. The Breakpoints panel's own delete/trash/X controls
  sent nothing on the wire — entries clear client-side, leaving the
  server-side hook in place until an explicit `None → OK`.
- **The Breakpoints panel** (the vendor's vocabulary is "queries")
  exposes load/save of query files (file dialogs, not exercised here),
  delete/trash and X controls, momentary Action buttons C/S/F (no wire
  effect against this server), an auto-zoom control, and a list showing
  both hook kinds with renumbering.
- **The per-node Hook Config dialog** offers None / Breakpoint
  (Interactive) / Replace Node (Success | Failure) with a Once
  checkbox, Cancel/OK, and **pre-selects the node's current
  configuration on reopen**.

### G-4 — the live-monitor interop, proven end-to-end (2026-11-13)

Work package G-4 is **DONE** (2026-11-13), and it inverts the usual
direction of the proof. The compatibility surface is specified on the
*server* side by BehaviorTree.CPP; the client side — the reference IDE's
live monitor — has no public spec. G-4 verified it by building the server
**purely from the pinned public specification** (a spec-conformance
publisher, no reference to the vendor application) and running the
reference IDE against it:

- **The IDE rendered the served tree exactly** — the served scenario, a
  root `Sequence` with three `Script` children, appeared in the monitor
  as authored by the spec-built server.
- **The live transitions animated.** The server replayed a scripted
  transition scenario on a loop; the IDE animated the status changes over
  **thousands of requests** with zero server-side exceptions.
- **The IDE's own "Transitions" panel decoded the status records** — the
  3-bytes-per-node records (`uint16` node id + `uint8` status) —
  **including the `10 + previous_status` IDLE encodings** (11–14), which
  the panel rendered as the corresponding idle-after-success /
  idle-after-failure / idle-after-skipped states.

So the live-monitor wire protocol interop is **ESTABLISHED**: a server
built from the public spec drives the reference IDE's monitor UI
end-to-end — tree rendered, transitions animated, status encodings
decoded by the IDE's own UI. The evidence, with the client's observed
validation bounds and the free-tier boundary observations, is banked in
the private campaign tree.

**The client's status-validation bounds (observed behavior).** The open
spec's decoder note treats values outside 0–4 and 11–14 as IDLE; the
reference IDE is stricter and *validates*: it accepts status bytes
**0–4 and 11–14** and rejects everything else, surfacing an error dialog
and disconnecting/reconnecting on the bad record. In particular **byte 10
("IDLE-from-IDLE") is rejected** — and in a conformant stream it is
unreachable: a node's status byte starts at 0 (IDLE) and moves to
`10 + previous` only on a *real transition to IDLE*, so a never-ticked
node reads 0, never 10. A compatible monitor must never emit 10.

**Mode agreement is a client-side gate (client quirk).** Running the
server in a request-mode the IDE had not agreed to (the V3/V4 mode
mismatch) did not fail cleanly: the IDE **stacked 296 identical error
dialogs** — it kept polling and kept erroring instead of backing off. A
client quirk recorded as a compatibility note: mode must match, or the
session degrades into a dialog flood rather than a clean refusal.

**Free-tier boundary observations (first G-6 input).** Observed on the
v1.10.0 free tier during this work: the **blackboard visualization is
absent from the free-tier UI** (PRO-gated — there is no blackboard
control, not a disabled one); triggering the breakpoint feature presents
the **PRO-upgrade dialog** (breakpoints are not armable on free tier);
and the **log-recapture ("Record") control is present and toggleable** on
free tier, offering a **log export** on stop. So the free-tier gate is
not a single node-count check — some features are gated by *absence from
the UI*, while others (record/export) remain present.

### G-5 — the v3-compat mode, resolved (2026-11-14)

Work package G-5 is **DONE** (2026-11-14). The v3-compat surface was first
pinned from the open-source side (the BehaviorTree.CPP 3.x wire, file and
XML dialects — the v3 counterpart of the 4.9.0 compatibility-surface
study), then three live rounds were run: the reference IDE (Groot2 v1.10.0)
as client against reference servers built by the campaign, in V3
Compatibility mode and in V4 mode.

**Headline: the monitor wire is v4 in both modes; V3 Compatibility is an
editor-side switch — one wire protocol, two XML dialects.**

- **The v3-protocol server was rejected.** A reference server speaking the
  v3 wire exactly (the v3 publisher: REP answering with the tree
  flatbuffer on every request, the v3 binary status/transition stream on
  the v3 PUB socket) was **rejected by the V3-mode client** — no render,
  no animation. Observed connect behavior shows why: in V3 mode the client
  dials **REQ on the REP port and SUB on REP+1 — the v4 port layout —
  regardless of mode**. The v3 wire's sockets and grammars never meet the
  client's sockets, in either mode.
- **The control experiment connected and animated.** A v4-wire reference
  server whose FULLTREE payload was the **v3-dialect tree XML**
  (`BTCPP_format="3"`, the v3 node set) connected **instantly and
  animated** in V3 mode — and rendered identically in V4 mode. The mode
  switch changes the dialect the editor serves and accepts, not the wire.
- **The hybrid was refuted.** A hybrid reference server (v4 REQ + v3-
  dialect XML on the REQ side, **v3 binary frames** on the PUB socket)
  connected and rendered via the v4 REQ half — one FULLTREE on connect,
  ~10 Hz STATUS polling for the session, a clean remove-all-hooks teardown
  on close — but **every v3-binary PUB frame was rejected** ("Invalid
  monitor notification format", ~1474 times over ~45 s) with zero display
  animation from the PUB stream. Live-state animation rides the STATUS
  replies, so the PUB half was never needed for the display.

**THE CONCLUSION for the reconstruction:** a compatible monitor needs
**ONE wire protocol** (the v4 publisher surface, in both editor modes) and
**TWO XML dialects** (v4 and v3 — `SetBlackboard`, `SubTree`/
`SubTreePlus`, `BTCPP_format="3"`, which the v3 parser accepts but does
not enforce). A v3 robot talking the v3 binary wire needs a **translator**
between its publisher and the client — the same reason the community's v3
monitor ships its own translation layer.

**NEEDS-OBSERVATION corner:** the V3-mode *notification* (SUB) parse
rejected both observed frame formats — the v4 notification style and the
v3 binary stream. What satisfies it is unknown, and practically
irrelevant: V4 mode handles notifications, and live animation does not
depend on the PUB socket.

**v3 save-side findings (observed in the live round).** Saving in V3 mode
from the reference IDE: fresh saves **declare `BTCPP_format="3"`**; the
serializer **preserves `SubTree` spelling** — it never translates to
`SubTreePlus`; fresh saves embed **zero `TreeNodesModel` entries** — the
built-in models are not persisted (a significant serializer behavior: a
compatible implementation must reproduce the drop, and only models loaded
from included files get embedded); the default new-file name is
**`untitled_v3_1`** (v4: `untitled_1`).

### G-6 — the free-tier boundary map, compiled (2026-11-14)

Work package G-6 is **DONE** (2026-11-14) as a compiled boundary map:
exactly what the free tier's 20-node cap gates, and whether anything else
is licensed-gated at the protocol level rather than the UI. The compilation
covers the G-4 free-tier interop session, the G-2 recording session (both
on the free tier), the G-3 PRO-trial round on the other side of the line,
and the vendor's published tier description as recorded in the
compatibility-surface study and the version table below. No new
observation was taken; the gate strings not yet in the record are
NEEDS-OBSERVATION, not banked facts.

**Headline: the free/PRO line is a product decision at the UI level, not a
protocol boundary.** The cap is not a single 20-node gate: some PRO
features are gated by **absence from the UI** (blackboard visualization —
there is no blackboard element in the free-tier monitor UI at all, while
the protocol still serves blackboard dumps), others by an **upgrade
dialog** (breakpoints — not armable; the dialog reads *"To unlock this
feature, please upgrade to the PRO version of Groot2."*), and a third set
remains present and functional on free (record/export — the Record control
is toggleable, offers export on stop, and the free tier's own writer
declared `FileLogger2 Log file (*.btlog)` unmodified).

**The boundary table (free tier, Groot2 v1.10.0, observed):**

| Capability | Free tier, observed | Gate shape |
|---|---|---|
| Editor & project authoring | full — every G-1 save-and-byte-diff run performed from the free-tier IDE | free |
| Multi-tree `.btproj` load | the vendor's own 7-tree, 10-custom-model v4 project loads and renders | free |
| v3 compatibility mode | the "Switch to V3" mode switch and "(V3 Compatibility)" retitling work | free |
| `.btlog` replay | a public-grammar log plays back — 16 transitions, timestamps exact | free |
| `.btlog` record + export | Record control present and toggleable; export offered on stop; the IDE's own writer produced a clean 196-transition recording | free |
| Live monitor | full interop — tree rendered exactly, transitions animated over thousands of requests, status encodings decoded by the IDE's own Transitions panel | free, under the cap |
| Monitor/log node cap | per the vendor's published tier description; its enforcement at the edge not yet exercised | **20 nodes** — **NEEDS-OBSERVATION** at 19/20/21 nodes |
| Blackboard visualization | absent from the free-tier monitor UI (not disabled; the protocol still serves dumps) | PRO by absence |
| Breakpoints | not armable; the PRO-upgrade dialog is presented on trigger (the ZMQ hook path is not reachable from the free-tier UI) | PRO by dialog |
| Node search | not exercised on free | PRO per the tier description — gate text **NEEDS-OBSERVATION** |
| Transitions limit | a 500-transition free-tier limit is asserted in the G-6 brief but appears in no banked log; the largest observed recording is 196 transitions | unverified — **NEEDS-OBSERVATION** |

**The PRO side of the line (ESTABLISHED).** The vendor's published tier
description puts **five** things behind PRO: **breakpoints
(interactive mode)**, **fault injection (Replace Node, Success |
Failure)**, **blackboard visualization**, **node search**, and the
**uncapped monitor** (free caps the monitor and log visualizer at 20
nodes). Two of the five were exercised end-to-end under the trial
(G-3 — the complete wire lifecycle of both is banked there); the
blackboard's protocol half is exercised on the server side (the `'B'`
dump, MessagePack-JSON — G-4).

**The trial route — the reference-license path (ESTABLISHED).** The PRO
tier was reached the vendor's own way: the **free 30-day PRO trial**,
started from the preferences dialog — one click, no payment, no
circumvention, no key cracking. Observed activation behavior:

- **Activation is an online check with a short bounded wait** — the
  application's own log at first PRO-tier start (2026-11-13): `[STARTUP]
  Waiting for license activation (max 5s)...` followed by `[STARTUP]
  License activation completed`.
- **The trial re-validates locally by default.** With the vendor endpoints
  unreachable, the activated application kept working: license state is
  held and checked locally between contacts — machine-bound license state
  (a fingerprint at the licensing layer; the fingerprint itself was not
  observed, and the record says no more than that).
- **Days-left indicator: NEEDS-OBSERVATION** — asserted in the G-6 brief,
  present in no banked log; the trial round's license-tab capture can be
  transcribed to settle it.

**Pricing context (ESTABLISHED, cited).** PRO is **€590/yr, sold as a
floating license** — the vendor's public product page and README
changelog, as recorded in the compatibility-surface study and carried in
the version table's v1.10.0 row.

**What the boundary means for the reconstruction.** Because the
compatibility surface is specified almost entirely on the open-source side
of the wall, everything the free/PRO line gates, an independent
implementation can simply provide — all of it. The two flagship PRO
features are, on the wire, two requests the public protocol fully
specifies (`HOOK_INSERT` / `HOOK_REMOVE`, plus the one-time
`BREAKPOINT_REACHED` publish), and their semantics are
**server-authoritative**: the reference client never sent a
`BREAKPOINT_UNLOCK` against a spec-conformance server, so pause-or-not is
whatever the server's pre-tick callback decides (G-3). The blackboard the
free tier hides is a standard protocol dump the reference interop server
already serves (G-4). The 20-node cap sits far below the protocol's own
ceiling — `node_uid` is `uint16`, i.e. up to 65,535 nodes. The free/PRO
line is therefore a **reimplementation decision**, not a technical wall:
compatibility requires matching the publicly specified wire and file
behavior, not reproducing the gate, and which features to gate (if any)
is ours to choose.

**Remaining NEEDS-OBSERVATION:** (1) the **19/20/21-node cap-edge
sweep** — G-6's verification target, the last unexercised cell of the
table; (2) the **unbanked gate strings** — four vendor strings carried in
the G-6 brief appear in no banked observation log and are quarantined
here: a monitor-cap dialog ("The free version of Groot2 can monitor only
trees with 20 or less nodes."), a free-tier limits line ("…limited to 500
transitions and trees not larger than 20 Nodes." — reads as vendor page
copy rather than a dialog), a second breakpoint-dialog phrasing
("Adding a breakpoint to the tree requires the PRO version." — **conflicts**
with the banked G-4 string and must be reconciled by observation,
plausibly a different trigger path surfacing a different dialog), and a
search gate text ("Disabled. Searching requires the PRO version."). All
four are to be captured (or discarded) in the next free-tier session,
alongside the cap-edge sweep.

## A. Product & version archaeology

| Version | Released | Edition/modules | OS/arch | Notes | Status |
|---------|----------|-----------------|---------|-------|--------|
| Groot 1 (legacy) | pre-2024 | Groot | Windows, Linux | predecessor; BT.CPP 3.x era | NEEDS-OBSERVATION — release binaries for the 1.x-generation lineup downloaded and hashed, and the lineage (BT.CPP 3.x era) confirmed against the vendor's own page history |
| Groot2 1.0 | 2024 | free tier + PRO | Windows | launch; ZMQ live-monitor compat | NEEDS-OBSERVATION — the 1.0 release binary downloaded and hashed, the 2024 release date confirmed, and the ZMQ live-monitor behavior observed on that build |
| Groot2 1.1 | 2024 | free tier + PRO | Windows, Linux | **BT.CPP 3.8+ `Groot2Publisher` compatibility** added; breakpoints/file save | NEEDS-OBSERVATION — the 1.1 binary downloaded and hashed, and the changelog date cross-checked against the GitHub issue tracker and the vendor release page |
| Groot2 1.8.x | 2025 | free tier + PRO | Windows, Linux | themes; Groot1-compat save toggle; port-validation toggle | NEEDS-OBSERVATION — the 1.8.x binaries downloaded and hashed, and the 2025 dates plus the 1.2–1.9 change boundaries cross-checked against the issue tracker; the Linux build confirmed on Linux hardware |
| Groot2 1.10.0 (Auryn Robotics) | 2026-11-09 | free tier (20-node cap on monitor/log) + **PRO** (€590/yr: breakpoints, fault injection, blackboard viz, search) | Windows | **reference version under test** | **ESTABLISHED** — binary in hand (hash recorded); running behavior observed on the reference environment 2026-11-13, including the edition/module split and the free-tier node cap |

ESTABLISHED:

- **Free tier:** full editor; the live monitor and log visualizer are capped
  at 20 nodes. PRO (€590/yr) lifts the cap and adds breakpoints, fault
  injection, blackboard visualization, and cross-project search.
- **Ecosystem pairing:** Groot2 1.10.0 pairs with BehaviorTree.CPP 4.11.0
  (released 2026-11-08/09); the reference study pinned the contract against
  4.9.0, the most recent source tree fully audited at kickoff.
- **Sample corpus:** 15 files embedded in the v1.10.0 binary — the
  vendor's own v3/v4 project and tree samples — provenance recorded,
  banked privately.

NEEDS-OBSERVATION (the table's status column names the per-row evidence that
would promote each row): the full edition/module matrix across all releases,
the OS/arch support matrix — in particular whether any release shipped for
macOS — and the 1.2–1.9 change boundaries.

## B. Binary & runtime inventory — AWAITING RESEARCH

Executables, embedded runtime, the bundled ZMQ, the SQLite dependency, any
scripting surface, the settings store, and where free-vs-PRO feature checks
live — to be inventoried on the reference environment *before* any deeper
work, to find subsystem boundaries that can be tested independently.

## C. Functional inventory — AWAITING RESEARCH (map from vendor help/changelog;
behavior is reference-lab work)

Documented surface to inventory: tree editor (nodes, ports, remapping,
subtrees, conditions), the graph view, the live ZMQ monitor (polling,
reconnect, breakpoints, fault injection, blackboard visualization), the log
replayer (`.btlog`, `.db3`, live-capture), the XML import/export paths,
project settings, theme/UX preferences, and the free-tier node cap. Each
item maps to C1–C9 in the matrix below.

## D. Behavioral specification — TO BUILD (reference-lab)

Executable reference tests, given input + parameters + version → exact
decoded result and project-state change. The graded-fixture ladder has its
first four rungs from the kickoff plus G-1 (the vendor sample corpus, the
pinned-grammar log, the v3↔v4 mode-switch project, and the pinned
`.btproj` save-shape) and the reference IDE's own saved recording (G-2 —
the plain-format result above); the v3-compat grade (G-5) is resolved
(2026-11-14) — the wire does not change with the mode, so the remaining
graded fixtures are the v3 dialect files themselves (the v3 project and
the v3 log/flatbuffer surface, pinned in the G-5 findings above).

## E. Reconstruction architecture — EARLY DECISIONS

- **Base on the open-source BehaviorTree.CPP specification, not the vendor
  binary.** The compatibility-surface study establishes that ~90% of the
  interface a compatible IDE needs is pinned in BehaviorTree.CPP's public
  headers and sources — the `.btlog` grammar, the ZMQ monitor protocol
  (all 12 request tags, the status-buffer idle encoding, MessagePack-JSON
  blackboards, full hook/breakpoint JSON), the `.db3` schema, and the full
  BT.CPP 4 XML dialect including `TreeNodesModel`. Build against that.
- **Landscape (for the reconstruction), corrected 2026-11-15.** An earlier
  version of this section said breakpoints and fault injection had "zero
  open implementations" and that no open project held a live ZMQ client.
  Both claims were wrong; this is the retraction. Open projects that speak
  the Groot2 monitor protocol to an unmodified `Groot2Publisher`, as each
  project describes itself (ESTABLISHED as *cited*; none exercised against
  the reference environment here):
  - **BTick** (SherifFathey/btick_app, Apache-2.0; Electron + React) —
    live monitor with tree, status, transitions and read-only blackboard;
    over the Groot2 protocol its documentation lists breakpoints as
    supported and forced SUCCESS/FAILURE "through hook replacement"; an XML
    editor for BT.CPP v4; session record/replay to its own JSON format (not
    `.btlog`); "unlimited nodes". Sources: the project's docs
    (sheriffathey.github.io/btick_docs, protocol-comparison and XML-editor
    pages) and its announcement on discourse.openrobotics.org (topic 59220).
  - **slci/Groot** (MIT, experimental) — a v4 port of the Groot 1 editor:
    edits and saves v4 XML, monitors via `Groot2Publisher`, replays
    FileLogger2 `.btlog`; its README states it does not implement
    breakpoints, blackboard inspection or log capture.
  - **EvoWorkforce/BehaviorTreeMonitor** (MIT) — read-only monitor with a
    live blackboard and transition timeline; never inserts hooks.
  - Plus the `btview` protocol re-pin and mock (Apache-2.0) and the VS Code
    viewer already cited. Groot 1 (BehaviorTree/Groot) is MIT-licensed
    (licence text in its README).

  What none of them claims, and what remains this reconstruction's
  objective, is **compatibility with the reference IDE's own documents and
  records, measured against it**: the `.btproj` and tree-XML save shape in
  both dialects (the G-1/G-5 serializer rules above), `.btlog` as the
  recording format, and the `.db3` schema — together with editor, monitor
  and replay in one data model. HYPOTHESIS until each project's file
  handling is checked against the fixtures in §F: that none of them
  reproduces the reference serializer byte-for-byte.
- **Subsystems needing new work:** a document layer that round-trips both
  XML dialects losslessly (the serializer delta of G-1/G-5), the `.btlog`
  and `.db3` read/write paths, the live ZMQ client (open clients exist, but
  none is a base this objective can inherit; polling cadence and reconnect
  are ours to choose, the protocol is permissive), the breakpoint and
  fault-injection UI over the hook lifecycle, the free-tier node-cap removal
  (a product decision, not a protocol one), and the v3-compat mode switch.
- **Open bases evaluated:** BehaviorTree.CPP itself (the specification), the
  open `.btlog` writer/replayer (as cross-check and test double), the open
  ZMQ protocol project and its mock (as a second oracle). None is a complete
  IDE; the contribution is the integration, not any single piece.

## F. Verification & release strategy

Per-capability compatibility claims tied to the exact reference versions —
Groot2 **v1.10.0 free tier** and BehaviorTree.CPP **4.9.0** (with a 4.11
cross-check once the delta work is done). Evidence units: the vendor sample
corpus (15 files) as load fixtures; a pinned-grammar `.btlog` round-tripped
against the 4.9.0 writer and played back in the reference IDE; the v3↔v4
mode-switch project as a compatibility-mode fixture (re-verified in V3
mode by G-5, 2026-11-14: the v4 wire carries the v3-dialect XML and
animates identically in both modes); a `.btproj` byte-diff against a
banked sample after a save-as (work package G-1); the G-4 interop proof —
the reference IDE connected as a live client to a spec-built monitor
server (tree rendered, transitions animated, status encodings decoded by
the IDE's own Transitions panel). "Full support"
requires an enumerated feature set + passing reference tests, never the
absence of known bugs.

## Capability coverage matrix (current)

Coverage triplet, recorded independently; none unless noted. Treat as a
coverage sketch — no enumerated feature set exists yet.

| Capability | Reference (observed?) | Implementation | Verification |
|------------|----------------------|----------------|--------------|
| C1 Acquisition & execution | partial — free full v1.10.0 installed and operated; sample corpus banked with provenance | n/a | n/a |
| C2 Data compatibility | partial — `.btlog` verified bidirectionally at the container level (writer/reader round-trip PASS against the 4.9.0 writer; a pinned-grammar log loads and plays back in the reference IDE with 16 transitions exact; a recording saved by the reference IDE itself parses end to end with zero errors by the pinned-grammar reader — G-2, 2026-11-13); `.btproj` save-shape verified bidirectionally at the container and serialization level (G-1, 2026-11-13); live-monitor status records verified against the reference IDE's own Transitions panel, including the `10+prev` IDLE encodings (G-4, 2026-11-13); the v3 file/log/flatbuffer surface pinned from source, and the v3-vs-v4 wire relationship resolved experimentally (G-5, 2026-11-14) | none | partial (kickoff WP1–WP2; G-1; G-2; G-4; G-5) |
| C3 Document compatibility | partial — the `.btproj` save-shape is pinned by save-and-byte-diff (G-1): serializer delta, include-rewrite rule, and V3-vs-V4 behavior all established; the v3 XML dialect pinned (G-5): `SetBlackboard`, `SubTree`/`SubTreePlus`, `BTCPP_format` accepted but not enforced, plus the observed V3 save behavior (declares `BTCPP_format="3"`, preserves `SubTree` spelling, embeds zero `TreeNodesModel` entries, default name `untitled_v3_1`) | none | partial (G-1; G-5) |
| C4 Computation | n/a (a behavior-tree IDE; the C++ library is the compute reference) | none | none |
| C5 Workflow semantics | partial — the v3↔v4 mode switch is resolved (G-5, 2026-11-14): the monitor wire is v4 in both modes; V3 Compatibility is editor-side (XML dialect, node models, serializer), so a compatible IDE needs one wire protocol and two XML dialects; the G-6 boundary map is compiled (2026-11-14): the free/PRO line is a product decision at the UI level, not a protocol boundary | none | partial (G-5; G-6) |
| C6 Programmability | documented only (the ZMQ monitor protocol, fully specified in open source) | partial — a spec-built server drove the reference IDE's live monitor end-to-end (G-4, 2026-11-13), in both editor modes (G-5, 2026-11-14) | partial (G-4; G-5) |
| C7 Instrument integration | n/a (a behavior-tree IDE, not instrument-bound) | n/a | n/a |
| C8 User-facing application | partial — the vendor app is documented as behavioral reference for the v4 project load, log playback, and v3 mode-switch (kickoff WP3), as the live client in the G-4 interop proof (its monitor UI rendered the served tree, animated live transitions, and decoded the status encodings), and as the live client of the G-5 v3-mode wire experiments (connected and animated on the v4 wire in both modes; its V3-mode notification parse rejected both observed frame formats — NEEDS-OBSERVATION) | none | none |
| C9 Replacement coverage | not yet measurable (the work-package table is closed; the matrix above is the coverage sketch against which an enumerated feature set would be measured) | — | — |

## Reference environment

- **VM / workstation:** clean install, Groot2 v1.10.0 free tier, BehaviorTree
  .CPP 4.9.0 as the pinned reference library for fixture generation.
- **Fixture sources:** (a) the 15 files embedded in the vendor binary —
  banked privately with provenance; (b) self-authored `.btproj`/tree files
  round-tripped through the reference IDE; (c) a `.btlog` generated purely
  from the pinned public grammar; (d) the G-4 interop session — the
  reference IDE connected as a live client to a spec-built monitor
  server, its rendered monitor and Transitions panel captured;
  (e) a `.btlog` saved by the reference IDE's own Record control,
  recorded live against the spec-built monitor server (G-2).
- **Recording:** file hashes per saved fixture; header/record inventory per
  grade; decoded transitions compared against the writer's own output; the
  reference IDE's own output captured alongside.
- **Runner:** one input varied at a time (node count across the free-tier
  cap, v3 vs v4, with/without `BTCPP_format`, with/without `TreeNodesModel`,
  breakpoint armed/disarmed); the G-5 wire experiments (2026-11-14)
  varied the monitor grammar — v3 wire vs v4 wire, v3-dialect vs v4-dialect
  XML, v3-binary notification frames — and the G-6 boundary inputs were
  taken from the free-tier save/export artifacts and the PRO-trial
  license-tab capture.

## Legal preconditions

Lawful reference install: the vendor's own free tier (no license key
required for the editor; PRO features gated behind a paid tier, not a
separate binary). Clean-room wall for any logic derived from observing the
proprietary binary. No vendor trademarks — "reads Groot2 projects" /
"Groot2-compatible" — never the product name in an implementation. See
`../docs/legal-basis.md`.

## Work packages (initial)

| WP | Scope | Capability | Verification | Status |
|----|-------|-----------|--------------|--------|
| G-0 | Vendor sample corpus: 15 files, parse + include-resolve + instantiate; record the 3 vendor-generation regressions and the v3/v4 delta fixtures | C2/C3 | all 15 parse; includes resolve; the 3 throws recorded | **done 2026-11-13** |
| G-1 | `.btproj` exact save-shape: save-as from the reference IDE, byte-diff against a banked sample; enumerate the Groot2-only attributes (`editable`, `<description>`, any others) and the per-project settings block | C2/C3 | byte-diff clean; the Groot2 extension attributes enumerated | **done 2026-11-13** — result in the findings above |
| G-2 | The recording-container question: does the reference IDE's own "save log" produce a plain `.btlog`, or a Groot2-side wrapper? (The bundled zstd is the hint.) | C2 | a saved log, hex-inspected against the pinned grammar | **done 2026-11-13** — the recording is the plain public format: a live recording captured from the reference IDE parses end to end with zero errors by the pinned-grammar reader (196 transitions, tree XML identical to the served tree); the IDE's own save dialog declares `FileLogger2 Log file (*.btlog)`; the zstd container hypothesis is closed for the log format |
| G-3 | Breakpoints & fault-injection behavior — the two flagship PRO features. Needs a PRO license, or the free-tier boundary observed to the limit (does the UI surface them, is the node cap the only gate, does the ZMQ hook path respond when armed from an external client) | C5/C6 | recorded breakpoint arm/disarm/clear cycle; fault-injection substitution observed | **done 2026-11-13** — the PRO surface is mapped (see the G-3 findings above): run under the vendor's free 30-day PRO trial, against the spec-conformance monitor server with a correlated server-log census; breakpoints are server-authoritative per the open-source protocol, fault injection is a silent replace-mode hook, and removal is `Hook Config → None → OK` only |
| G-4 | ZMQ live-monitor interop, proven end-to-end: a server built purely from the public spec (all 12 request tags, the status-buffer idle encoding, blackboards, hooks, recording) driven the reference IDE's monitor — tree rendered, transitions animated, the Transitions panel decoded the status encodings (incl. `10+prev`, 11–14). The client's status-validation bounds established: accepts 0–4 and 11–14, rejects all else (notably byte 10, which is unreachable in a conformant stream); a request-mode mismatch stacks error dialogs (296 observed) instead of a clean refusal | C6/C8 | the reference IDE connected as a live client to the spec-built server: tree, live transitions, and status decoding all correct; thousands of requests, zero server-side exceptions | **done 2026-11-13** — result in the findings above |
| G-5 | v3-compat mode: the full v3 project load (not just the mode switch), the "(V3 Compatibility)" retitling behavior, and what the IDE changes on save after the switch | C3/C5 | the v3 project round-trips through the reference IDE without losing the mode | **done 2026-11-14** — the headline: the monitor wire is v4 in both modes, "V3 Compatibility" is editor-side (one wire protocol, two XML dialects); v3 save-side behavior established; the V3-mode SUB notification parse is the NEEDS-OBSERVATION corner |
| G-6 | Free-tier boundary map: exactly what the 20-node cap gates, and whether any other feature is licensed-gated at the protocol level (as opposed to the UI). Informed by the G-4 free-tier observations: blackboard visualization absent from the free-tier UI (PRO-gated), breakpoint feature PRO-gated (upgrade dialog on trigger), Record control present with log export | C2/C8 | recorded cap behavior at 19/20/21 nodes | **done 2026-11-14** — the boundary map compiled (headline: the free/PRO line is a product decision at the UI level, not a protocol boundary); remaining: the 19/20/21-node cap-edge sweep and the unbanked gate strings (NEEDS-OBSERVATION) |

## Open questions / blockers (ranked)

1. **ZMQ live-monitor client behavior** — the open-source side pins the
   *server*, not the *client*: the reference IDE's exact polling cadence
   and its reconnect back-off remain to be quantified. G-4 (2026-11-13)
   removed the interop risk: a spec-built server drove the reference
   IDE's monitor end-to-end (tree rendered, transitions animated, status
   encodings decoded), and the client's status-validation bounds are
   established as observed behavior (accepts 0–4 and 11–14, rejects all
   else, notably byte 10; a mode mismatch stacks error dialogs) — see
   the findings above. G-3 (2026-11-13) adds the PUB-socket half of the
   client vocabulary: the IDE sends nothing on the PUB side beyond the
   six request kinds, and never sends `BREAKPOINT_UNLOCK` (breakpoint
   semantics are server-authoritative per the open-source protocol).
   G-5 (2026-11-14) adds the mode dimension: the client dials the v4
   port layout (REQ + SUB on REP+1) in **both** editor modes — the mode
   switch never changes the wire — and its V3-mode SUB *notification*
   parse rejected both observed frame formats (v4-style and v3-binary;
   "Invalid monitor notification format", ~1474 occurrences over ~45 s).
   What satisfies that parse is NEEDS-OBSERVATION, and practically
   irrelevant: V4 mode handles notifications, and live animation rides
   the STATUS replies, not the PUB socket.
2. ~~**v3-compat mode depth**~~ — **resolved by G-5 (2026-11-14):** the
   monitor wire is v4 in both modes; V3 Compatibility is editor-side (the
   XML dialect, the node models, the serializer). One wire protocol, two
   XML dialects; a v3 robot needs a translator. The v3 save-side behavior
   is established (fresh saves declare `BTCPP_format="3"`, preserve
   `SubTree` spelling, embed zero `TreeNodesModel` entries, default name
   `untitled_v3_1`). See the G-5 findings above.
3. **Free-tier boundary map** — **compiled by G-6 (2026-11-14):** the
   free/PRO line is a product decision at the UI level, not a protocol
   boundary (the gate shapes: PRO-by-absence for blackboard visualization,
   PRO-by-dialog for breakpoints, free and present for record/export; the
   five PRO-gated items per the vendor's published tier description).
   Two NEEDS-OBSERVATION items remain: the **19/20/21-node cap-edge
   sweep** (the last unexercised cell of the table), and the **unbanked
   gate strings** — four vendor strings in the G-6 brief that appear in
   no banked log, including a breakpoint phrasing that conflicts with the
   banked G-4 string. See the G-6 findings above.

(Formerly ranked first: breakpoint and fault-injection behavior. Resolved
by G-3 on 2026-11-13 — see the findings above; the result drops G-3 from
the open list. Remaining format work: G-5 v3-compat mode depth, and the
G-6 free-tier boundary map (partially informed by the G-4 and G-2
free-tier observations).)

(Formerly ranked first: the recording container. Resolved by G-2 on
2026-11-13 — see the findings above; the result drops G-2 from the open
list.)

(Formerly ranked first: the exact `.btproj` save-shape. Resolved by G-1 on
2026-11-13 — see the findings above; the result drops G-1 from the open list.)

(Formerly ranked second: the free-tier boundary map. Compiled by G-6 on
2026-11-14 — the map is in the findings above; the two remaining items are
the 19/20/21-node cap-edge sweep and the unbanked gate strings, both
NEEDS-OBSERVATION.)

(Formerly ranked second: v3-compat mode depth. Resolved by G-5 on
2026-11-14 — see the findings above; the result drops G-5 from the open
list, leaving only the NEEDS-OBSERVATION corners (the V3-mode SUB
notification parse; the G-6 cap-edge sweep and gate strings).)
