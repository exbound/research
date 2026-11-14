# Reconstruction dossier — Groot2 (target #14)

> Worked from the kickoff (issue #9) and the verified findings of its work
> packages 1–3 of 2026-11-13. ESTABLISHED = observed on the reference
> environment or cited from a public source; NEEDS-OBSERVATION = the next
> fixture grade. A dossier is a research record, not an engineering spec.

- **Target:** Groot2 (Auryn Robotics) — the proprietary integrated development
  environment for the open-source BehaviorTree.CPP behavior-tree ecosystem
- **Status:** research · **Last updated:** 2026-11-13 (G-3 breakpoint/fault-injection surface mapped via the PRO tier)
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
the plain-format result above); the next grade is the v3-compat mode depth
(G-5).

## E. Reconstruction architecture — EARLY DECISIONS

- **Base on the open-source BehaviorTree.CPP specification, not the vendor
  binary.** The compatibility-surface study establishes that ~90% of the
  interface a compatible IDE needs is pinned in BehaviorTree.CPP's public
  headers and sources — the `.btlog` grammar, the ZMQ monitor protocol
  (all 12 request tags, the status-buffer idle encoding, MessagePack-JSON
  blackboards, full hook/breakpoint JSON), the `.db3` schema, and the full
  BT.CPP 4 XML dialect including `TreeNodesModel`. Build against that.
- **Landscape (for the reconstruction):** the community already has (a) a
  monitor + replay path — an open `.btlog` writer that the vendor IDE opens,
  and a project that covers the live-monitor protocol deeply with a mock
  server usable as a test double; and (b) partial editors — including a VS
  Code plugin. What no project holds today is **editor + monitor + replay in
  one data model**, and the two flagship PRO features — breakpoints and
  fault injection — have **zero open implementations**. The reconstruction
  objective is exactly that integration: an independent IDE that unifies the
  three lives of a tree (authored, executing, recorded) over the publicly
  specified formats.
- **Subsystems needing new work:** the live ZMQ client (the open-source side
  specifies the server, not the client — polling cadence and reconnect are
  ours to choose, the protocol is permissive), breakpoints and fault
  injection (the C++ side implements every part of the hook lifecycle, but
  no open IDE wires it to a UI), the free-tier node-cap removal (a product
  decision, not a protocol one), and the v3-compat mode switch.
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
mode-switch project as a compatibility-mode fixture; a `.btproj` byte-diff
against a banked sample after a save-as (work package G-1); the G-4
interop proof — the reference IDE connected as a live client to a
spec-built monitor server (tree rendered, transitions animated, status
encodings decoded by the IDE's own Transitions panel). "Full support"
requires an enumerated feature set + passing reference tests, never the
absence of known bugs.

## Capability coverage matrix (current)

Coverage triplet, recorded independently; none unless noted. Treat as a
coverage sketch — no enumerated feature set exists yet.

| Capability | Reference (observed?) | Implementation | Verification |
|------------|----------------------|----------------|--------------|
| C1 Acquisition & execution | partial — free full v1.10.0 installed and operated; sample corpus banked with provenance | n/a | n/a |
| C2 Data compatibility | partial — `.btlog` verified bidirectionally at the container level (writer/reader round-trip PASS against the 4.9.0 writer; a pinned-grammar log loads and plays back in the reference IDE with 16 transitions exact; a recording saved by the reference IDE itself parses end to end with zero errors by the pinned-grammar reader — G-2, 2026-11-13); `.btproj` save-shape verified bidirectionally at the container and serialization level (G-1, 2026-11-13); live-monitor status records verified against the reference IDE's own Transitions panel, including the `10+prev` IDLE encodings (G-4, 2026-11-13) | none | partial (kickoff WP1–WP2; G-1; G-2; G-4) |
| C3 Document compatibility | partial — the `.btproj` save-shape is pinned by save-and-byte-diff (G-1): serializer delta, include-rewrite rule, and V3-vs-V4 behavior all established | none | partial (G-1) |
| C4 Computation | n/a (a behavior-tree IDE; the C++ library is the compute reference) | none | none |
| C5 Workflow semantics | awaiting research (the v3↔v4 mode switch is the first recorded) | none | none |
| C6 Programmability | documented only (the ZMQ monitor protocol, fully specified in open source) | partial — a spec-built server drove the reference IDE's live monitor end-to-end (G-4, 2026-11-13) | partial (G-4) |
| C7 Instrument integration | n/a (a behavior-tree IDE, not instrument-bound) | n/a | n/a |
| C8 User-facing application | partial — the vendor app is documented as behavioral reference for the v4 project load, log playback, and v3 mode-switch (kickoff WP3), and as the live client in the G-4 interop proof, where its monitor UI rendered the served tree, animated live transitions, and decoded the status encodings | none | none |
| C9 Replacement coverage | not yet measurable | — | — |

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
  breakpoint armed/disarmed).

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
| G-5 | v3-compat mode: the full v3 project load (not just the mode switch), the "(V3 Compatibility)" retitling behavior, and what the IDE changes on save after the switch | C3/C5 | the v3 project round-trips through the reference IDE without losing the mode | next |
| G-6 | Free-tier boundary map: exactly what the 20-node cap gates, and whether any other feature is licensed-gated at the protocol level (as opposed to the UI). Partially informed by the G-4 free-tier observations: blackboard visualization absent from the free-tier UI (PRO-gated), breakpoint feature PRO-gated (upgrade dialog on trigger), Record control present with log export | C2/C8 | recorded cap behavior at 19/20/21 nodes | next (partially informed) |

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
2. **v3-compat mode depth** — what the reference IDE does differently after
   the mode switch, and whether it re-writes the project in the v4 shape on
   save (G-5).
3. **Free-tier boundary map** — the exact feature set gated by the 20-node
   cap versus the PRO gate, at the protocol level rather than the UI level
   (G-6). Partially informed by the G-4 free-tier observations: blackboard
   visualization is absent from the free-tier UI (PRO-gated), breakpoints are
   PRO-gated (upgrade dialog on trigger), and the Record control is present
   with log export (G-2 exercised it on the free tier).

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
