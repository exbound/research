# Reconstruction dossier — Groot2 (target #14)

> Worked from the kickoff (issue #9) and the verified findings of its work
> packages 1–3 of 2026-11-13. ESTABLISHED = observed on the reference
> environment or cited from a public source; NEEDS-OBSERVATION = the next
> fixture grade. A dossier is a research record, not an engineering spec.

- **Target:** Groot2 (Auryn Robotics) — the proprietary integrated development
  environment for the open-source BehaviorTree.CPP behavior-tree ecosystem
- **Status:** research · **Last updated:** 2026-11-13
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
XML tree dialect and `TreeNodesModel`, and the SQLite `.db3` schema. The
compatibility-surface study on this branch cites each of these against the
4.9.0 source and against an independent open re-implementation of the
protocol as cross-check. What remains proprietary-only is small: the exact
`.btproj` save-shape beyond the observed shape, the free-tier feature
gating, and product/UX behavior.

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
first three rungs from the kickoff (the vendor sample corpus, the pinned-
grammar log, and the v3↔v4 mode-switch project); the next grade is to add
the exact `.btproj` save-shape (see work packages) and the recording-
container question.

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
against a banked sample after a save-as (work package G-1); a live
Groot2Publisher session captured by an independent monitor client. "Full
support" requires an enumerated feature set + passing reference tests, never
the absence of known bugs.

## Capability coverage matrix (current)

Coverage triplet, recorded independently; none unless noted. Treat as a
coverage sketch — no enumerated feature set exists yet.

| Capability | Reference (observed?) | Implementation | Verification |
|------------|----------------------|----------------|--------------|
| C1 Acquisition & execution | partial — free full v1.10.0 installed and operated; sample corpus banked with provenance | n/a | n/a |
| C2 Data compatibility | partial — `.btlog` verified bidirectionally at the container level (writer/reader round-trip PASS against the 4.9.0 writer; a pinned-grammar log loads and plays back in the reference IDE with 16 transitions exact) | none | partial (kickoff WP1–WP2) |
| C3 Document compatibility | awaiting research (the `.btproj` save-shape is the open item) | none | none |
| C4 Computation | n/a (a behavior-tree IDE; the C++ library is the compute reference) | none | none |
| C5 Workflow semantics | awaiting research (the v3↔v4 mode switch is the first recorded) | none | none |
| C6 Programmability | documented only (the ZMQ monitor protocol, fully specified in open source) | none | none |
| C7 Instrument integration | n/a (a behavior-tree IDE, not instrument-bound) | n/a | n/a |
| C8 User-facing application | partial — the vendor app is documented as behavioral reference for the v4 project load, log playback, and v3 mode-switch (kickoff WP3) | none | none |
| C9 Replacement coverage | not yet measurable | — | — |

## Reference environment

- **VM / workstation:** clean install, Groot2 v1.10.0 free tier, BehaviorTree
  .CPP 4.9.0 as the pinned reference library for fixture generation.
- **Fixture sources:** (a) the 15 files embedded in the vendor binary —
  banked privately with provenance; (b) self-authored `.btproj`/tree files
  round-tripped through the reference IDE; (c) a `.btlog` generated purely
  from the pinned public grammar; (d) a live `Groot2Publisher` session
  recorded for the ZMQ interop work package.
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
| G-1 | `.btproj` exact save-shape: save-as from the reference IDE, byte-diff against a banked sample; enumerate the Groot2-only attributes (`editable`, `<description>`, any others) and the per-project settings block | C2/C3 | byte-diff clean; the Groot2 extension attributes enumerated | next |
| G-2 | The recording-container question: does the reference IDE's own "save log" produce a plain `.btlog`, or a Groot2-side wrapper? (The bundled zstd is the hint.) | C2 | a saved log, hex-inspected against the pinned grammar | next |
| G-3 | Breakpoints & fault-injection behavior — the two flagship PRO features. Needs a PRO license, or the free-tier boundary observed to the limit (does the UI surface them, is the node cap the only gate, does the ZMQ hook path respond when armed from an external client) | C5/C6 | recorded breakpoint arm/disarm/clear cycle; fault-injection substitution observed | needs-license (or free-tier-boundary observation) |
| G-4 | ZMQ live-monitor interop: an independent client (REQ/SUB to 1667/1668) drives a real `Groot2Publisher` and a live reference IDE session side by side; pin the polling cadence, the reconnect behavior, and whether the reference IDE ever sends on the PUB socket beyond listening | C6/C8 | an independent monitor client renders a live session identically to the reference IDE | next |
| G-5 | v3-compat mode: the full v3 project load (not just the mode switch), the "(V3 Compatibility)" retitling behavior, and what the IDE changes on save after the switch | C3/C5 | the v3 project round-trips through the reference IDE without losing the mode | next |
| G-6 | Free-tier boundary map: exactly what the 20-node cap gates, and whether any other feature is licensed-gated at the protocol level (as opposed to the UI) | C2/C8 | recorded cap behavior at 19/20/21 nodes | next |

## Open questions / blockers (ranked)

1. **The exact `.btproj` save-shape** — what attributes and elements the
   reference IDE writes on save beyond `project_name`, `<include>`, and
   `<TreeNodesModel>`; whether per-project settings and breakpoint sets go
   in-file or in a side file. Resolvable by a save-and-diff (G-1), free
   tier suffices.
2. **The recording container** — whether the reference IDE's own saved log
   is a plain `.btlog`/`.db3` or a Groot2-side wrapper. The bundled zstd is
   the strongest hint of a wrapper, and this is the single largest remaining
   format risk for a log-replay replacement. Resolvable by a hex-inspect of
   a saved log (G-2).
3. **Breakpoint and fault-injection behavior** — the two flagship PRO
   features are the least-specified surface and have zero open
   implementations. Needs a PRO license, or the free-tier boundary mapped
   carefully enough to observe the behavior (G-3).
4. **ZMQ live-monitor interop** — the open-source side pins the *server*,
   not the *client*: the reference IDE's polling cadence, reconnect
   behavior, and any traffic on the PUB socket beyond listening are
   client-side choices we have to match closely enough to be compatible
   (G-4).
5. **v3-compat mode depth** — what the reference IDE does differently after
   the mode switch, and whether it re-writes the project in the v4 shape on
   save (G-5).
6. **Free-tier boundary map** — the exact feature set gated by the 20-node
   cap versus the PRO gate, at the protocol level rather than the UI level
   (G-6).
