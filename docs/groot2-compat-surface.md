# Dossier — BehaviorTree.CPP ↔ Groot2 compatibility surface

> Research record: what the **open-source** BehaviorTree.CPP side publicly
> specifies of the interface to the proprietary **Groot2** IDE (closed-source,
> free tier + €590/yr PRO, latest public release 1.10.0, 2026-11-09). Question
> answered: how much of an independent Groot2-compatible IDE is already
> specified by public sources?

- **Status:** research · **Last updated:** 2026-11-13
- **Primary evidence base:** BehaviorTree.CPP **4.9.0** (tag `4.9.0-
  d1db670a84`) source tree, plus GitHub `master` headers, `behaviortree.dev`
  docs (v4.11), `github.com/BehaviorTree/Groot2` (README changelog, issues),
  and wild `.btproj` examples from public ROS repos (husarion, DIT-ROBOTICS).
- **Conventions:** ESTABLISHED = quoted/cited below; NEEDS-OBSERVATION = needs a
  Groot2 binary run. Line refs are `path:line` into the 4.9.0 tree; where
  master differs, noted.

**Local tree** (vcpkg extract, used throughout):
`/home/cam/vcpkg/buildtrees/behaviortree-cpp/src/4.9.0-d1db670a84.clean/`
(cited as `BT/src/…` and `BT/include/…` below).

**Version delta note.** Groot2 1.10.0 pairs with BT.CPP 4.11.0 (both released
2026-11-08/09). The 4.11 release added a `Groot2Publisher(tree, port,
bind_address)` constructor (bind-address argument, security fix per 4.11
changelog, PR #1251) and a `ROOT` reserved name in blackboard dumps (PR #1168,
fixing #1132: subtrees whose blackboard is backed by a distinct root blackboard
now export it under `ROOT`, deduplicated across subtrees, with error replies
for ambiguous requests). Neither change alters any layout pinned below; the
bind-address constructor is an API addition, and `ROOT` is a name-convention
extension of the `BLACKBOARD` payload that a Groot2 client must know.

---

## 1. `.btlog` — `FileLogger2` format

**ESTABLISHED** — the format is fully specified in open source; a file is
deterministically produced by the logger, so a parser can be built and
verified against the writer without ever seeing Groot2.

`BT/include/behaviortree_cpp/loggers/bt_file_logger_v2.h:10-22` (class doc):

```
 * - first 4 bytes: size of the XML string (N)
 * - next N bytes: string containing the XML representing the tree.
 * - next 8 bytes: first timestamp (microseconds since epoch)
 * - next: each 9 bytes is a FileLogger2::Transition. See definition.
```

`BT/include/behaviortree_cpp/loggers/bt_file_logger_v2.h:46-55` (the
transition record):

```cpp
  struct Transition
  {
    // when serializing, we will remove the initial time and serialize only
    // 6 bytes, instead of 8
    uint64_t timestamp_usec;
    // if you have more than 64.000 nodes, you are doing something wrong :)
    uint16_t node_uid;
    // enough bits to contain NodeStatus
    uint8_t status;
  };
```

The writer (`BT/src/loggers/bt_file_logger_v2.cpp`) pins every layout detail
the class doc leaves open:

- **Offset 0:** magic string, `bt_file_logger_v2.cpp:62` —
  `_p->file_stream << "BTCPP4-FileLogger2";` (18 bytes, ASCII, no NUL).
- **Offset 18:** protocol version, `:64-66` — `uint8` value **1**.
- **Offset 19:** int32 XML length `N` (little-endian on this platform —
  written via `flatbuffers::WriteScalar` into a 4-byte slice), `:70-73`.
- **Offset 23 .. 23+N:** tree XML, `:76`. Generated with
  `WriteTreeToXML(tree, true, true)` (`:68`) — i.e. **with** `_uid`/
  `_fullpath` metadata attributes and built-in node models; see §5 for what
  that XML contains.
- **After XML:** 8-byte int64 **first timestamp**, microseconds since the
  system epoch, `:80-83` (`flatbuffers::WriteScalar` of `int64_t
  timestamp_usec`).
- **Then: N × 9-byte transition records**, little-endian, from
  `writerLoop()` `:148-158`:
  - bytes 0-5 (6 bytes): low 48 bits of `timestamp_usec`
  - bytes 6-7 (2 bytes): `node_uid` (`uint16`)
  - byte 8 (1 byte): `status` (`uint8`)

```
.btlog layout (all little-endian)
  0   18 bytes  ASCII "BTCPP4-FileLogger2"
 18   1 byte    uint8 protocol = 1
 19   4 bytes   int32 N  (length of XML)
 23   N bytes   UTF-8 tree XML (WriteTreeToXML(tree, true, true))
 23+N 8 bytes   int64 T0 = first timestamp, µs since epoch
 31+N …         repeated 9-byte records until EOF
```

**What a transition encodes:** *node identity* + *the new status only* —
**not** a previous status. From the callback, `bt_file_logger_v2.cpp:103-115`:

```cpp
void FileLogger2::callback(Duration timestamp, const TreeNode& node,
                           NodeStatus /*prev_status*/, NodeStatus status)
{
  Transition trans{};
  trans.timestamp_usec = uint64_t(ToUsec(timestamp - _p->first_timestamp));
  trans.node_uid = node.UID();
  trans.status = static_cast<uint64_t>(status);
```

- **Timestamp is relative to T0** (`timestamp - first_timestamp`), and the
  header's 8-byte T0 anchors it to wall-clock. Relative offset stored in 6
  bytes ⇒ ~5.4 years of uptime before truncation.
- **`prev_status` is discarded** — unlike the live-monitor status buffer
  (§2), a `.btlog` transition carries no "was IDLE, now SUCCESS" encoding. A
  player that wants "idle-but-succeeded" semantics must track last-seen
  state per UID itself.
- **`status` is the plain `NodeStatus` enum** (no `10+prev` idle-remembrance
  encoding, which is a live-monitor-only construct): `0` IDLE, `1` RUNNING,
  `2` SUCCESS, `3` FAILURE, `4` SKIPPED —
  `BT/include/behaviortree_cpp/basic_types.h:33-39`.
- **IDLE transitions are logged** — the constructor calls
  `enableTransitionToIdle(true)` (`bt_file_logger_v2.cpp:52`), and the
  constructor doc at `:26-32` notes the suffix must be `.btlog` "to
  correctly read this log with Groot2".
- A writer thread batches records (10 ms poll, `:140-144`) — the file is
  append-only; records appear in (nearly) nondecreasing time order.

Cross-check: the docs page (behaviortree.dev) and the Doxygen page for
`FileLogger2` restate the same layout (magic + version byte added to the
4+N+8+9 grammar; the header comment at `bt_file_logger_v2.h:15-20` predates
the magic and omits it). DeepWiki's summary of the same file matches:
"Magic header BTCPP4-FileLogger2 / Protocol version byte / XML length (4) /
XML / First timestamp (8 µs) / transitions (9 bytes: 6-byte µs offset, 2-byte
UID, 1-byte status)".

## 2. Groot2 live-monitor protocol — `Groot2Publisher`

**ESTABLISHED** — fully specified in open source: ports, socket types, wire
format, all request tags, every payload, and the breakpoint/hook semantics.
The public `btview` project (guilyx/btview, Apache-2.0) independently re-pins
the same layout against the same headers and is usable as a second oracle and
test double (`btview mock` implements the publisher side of the protocol).

### Ports and sockets

`BT/src/loggers/groot2_publisher.cpp:62` — two ZMQ sockets:

```cpp
PImpl() : context(), server(context, ZMQ_REP), publisher(context, ZMQ_PUB)
```

- **REQ/REP socket bound at `tcp://*:<port>`, default 1667** —
  `groot2_publisher.cpp:148-151` and `groot2_publisher.h:25`
  (`Groot2Publisher(const BT::Tree& tree, unsigned server_port = 1667)`).
- **PUB socket bound at `tcp://*:<port+1>`** (default **1668**),
  `groot2_publisher.cpp:149`. The PUB socket carries only one message kind:
  the breakpoint notification (`:588-593`).
- 4.11 adds a third constructor with a `bind_address` (IPv4 / interface name /
  `"*"` in ZMQ tcp syntax) — master `groot2_publisher.h`; security note
  quoted in §6.

So: the "Groot2 connection" a client makes is a **ZMQ REQ** connect to
`tcp://<host>:1667`; an independent **ZMQ SUB** connect to `:1668` receives
asynchronous `BREAKPOINT_REACHED` notifications.

### Framing

Messages are **ZMQ multipart** (`zmq::multipart_t`, `zmq_addon.hpp`). The
first frame of a request/reply is a fixed 6-byte header:

`BT/include/behaviortree_cpp/loggers/groot2_protocol.h:158-167` —
`SerializeHeader(RequestHeader)` writes, in order:

| offset | size | field |
|--------|------|-------|
| 0 | 1 | `uint8 protocol` = `kProtocolID` = **2** (`:98`) |
| 1 | 1 | `uint8 type` = `RequestType` tag |
| 2 | 4 | `uint32 unique_id` (request id, echoed in reply) |

All little-endian (`memcpy` of host-order scalons —
`Serialize/Deserialize` at `:144-156`).

A **reply** frame-1 is a 22-byte header: the same 6-byte header **plus** a
16-byte `tree_id` (`TreeUniqueUUID = std::array<char, 16>`, `:99`; serialized
at `:169-178`). The tree_id is a random v4-style UUID chosen once per
publisher instance at server start (`CreateRandomUUID()`,
`groot2_publisher.cpp:36-57` — 16 random bytes with variant/version bits set).

**Error reply:** a two-frame message whose first frame is the literal string
`error`, second frame the error text (`groot2_publisher.cpp:239-244`,
`sendErrorReply`).

### Request tags

`groot2_protocol.h:29-60` — `enum RequestType : uint8_t`:

| Tag | Name | Direction | Payload (request → reply) |
|-----|------|-----------|---------------------------|
| `'T'` | `FULLTREE` | poll | → tree XML (string) |
| `'S'` | `STATUS` | poll | → status buffer (3 bytes/node) |
| `'B'` | `BLACKBOARD` | poll, 2 frames | ← frame 2: `;`-separated blackboard names → MessagePack JSON dump |
| `'I'` | `HOOK_INSERT` | 2 frames | ← frame 2: Hook JSON (object or array) |
| `'R'` | `HOOK_REMOVE` | 2 frames | ← frame 2: `{"uid":n,"position":p}` |
| `'N'` | `BREAKPOINT_REACHED` | **server→Groot on PUB** | → `std::to_string(node_uid)` |
| `'U'` | `BREAKPOINT_UNLOCK` | 2 frames | ← frame 2: `{"uid","desired_status","position","remove_when_done"}` |
| `'D'` | `HOOKS_DUMP` | poll | → JSON array of hooks |
| `'A'` | `REMOVE_ALL_HOOKS` | poll | → (no payload) |
| `'X'` | `DISABLE_ALL_HOOKS` | poll | → (no payload) |
| `'r'` | `TOGGLE_RECORDING` | 2 frames | ← frame 2: `"start"`/`"stop"` → (`"start"` reply: int64 µs epoch as string; `"stop"`: no payload) |
| `'t'` | `GET_TRANSITIONS` | poll | → packed 9-byte transition records (same layout as `.btlog`), buffer cleared after read |

Tag dispatch: `groot2_publisher.cpp:291-491`. Notable payload specifics:

- **FULLTREE** (`:293-295`): reply payload = `WriteTreeToXML(tree, true, true)`
  captured at construction (`:118`) — static for the publisher's lifetime.
- **STATUS** (`:298-301`): reply = the pre-built status buffer
  (`:127-146`): **3 bytes per node, in tree order** — `uint16 uid` +
  `uint8 status`. Status encoding (`callback()`, `:200-226`): normally the
  plain `NodeStatus`, but a node transitioning **to IDLE** is encoded
  `10 + prev_status` (so `12` = "idle, last result SUCCESS" — `:206-209` and
  the `IDLE_FROM_*` constants at `:14-19`). Decoders should treat values
  outside `0..4` and `10..14` as IDLE.
- **BLACKBOARD** (`:304-313`, `:546-565`): request frame 2 is a
  `;`-separated list of subtree names (name = instance name or tree ID, `:134-136`);
  reply = `nlohmann::json::to_msgpack(json)` of `{ name → blackboard dump }`
  via `ExportBlackboardToJSON` — i.e. **MessagePack-serialized JSON**, not
  raw JSON text. 4.11 extension: subtrees whose blackboard resolves to a
  distinct root blackboard are exported under the reserved name `ROOT`.
- **HOOKS** (`groot2_protocol.h:201-262`): a `Hook` is
  `{enabled: bool, uid: uint16, mode: 0|1 (BREAKPOINT|REPLACE),
  position: 0|1 (PRE|POST), once: bool, desired_status: "SUCCESS"|"FAILURE"|
  "RUNNING"|"IDLE"|"SKIPPED"}`. `HOOK_INSERT` accepts a single object or an
  array. Breakpoint mechanics: the hook installs a **pre-tick callback** on
  the target node (`insertHook`, `:567-624`); on fire it sends
  `BREAKPOINT_REACHED` + uid on the PUB socket, then blocks (if mode
  BREAKPOINT) until `BREAKPOINT_UNLOCK` — or until the Groot2 connection is
  deemed dead, in which case all hooks are disabled (heartbeat logic,
  `heartbeatLoop` `:522-544`: default `max_heartbeat_delay` **5000 ms**,
  `:101`; any request from Groot2 counts as heartbeat, `:268-272`).
- **Recording** (`:438-485`): `'r' "start"` begins capturing 9-byte
  transitions (buffer capped at 1000, `:221-224`) and replies with the µs
  epoch; `'t'` drains the buffer as packed 9-byte records (identical layout to
  `.btlog` records).

### Handshake

There is no explicit handshake: Groot2 connects (ZMQ REQ to 1667, SUB to
1668), then **polls** — the publisher is a passive server; "connect"
establishment is inferred by Groot2 from a successful `FULLTREE` round-trip,
and connection liveness by the absence of requests for >5 s (at which point
the *publisher* side disables all breakpoints). A client should therefore
expect `T`/`S`/`B` polling at some interval and must send at least one
request within 5 s or all active hooks self-disable.

### Cross-check (independent)

`btview` (guilyx/btview, docs "Wire protocols") re-derives the same: 6-byte
request header (`protocol=2`, tag, `uint32 id` LE), 22-byte reply header
(+16-byte tree UUID), `FULLTREE` = XML from `WriteTreeToXML(tree, true, true)`
with `_uid`/`_fullpath`, STATUS = 3 bytes/node with the `10+prev` idle
encoding, BLACKBOARD = MessagePack JSON — and provides `btview mock`
(implements `T`/`S`/`B` on a real ZMQ stack) as a test double. Its v3 section
documents the BT.CPP 3.x `PublisherZMQ` wire format (FlatBuffers tree over the
REP socket; PUB socket pushes 12-byte transition records
`int32 t_sec, int32 t_usec, uint16 uid, int8 prev, int8 status`) — relevant
only if a Groot2 replacement must also speak the legacy Groot-1-era protocol,
which Groot2 1.1.0+ does ("Add compatibility with the Groot2Publisher in
BT.CPP 3.8+", changelog).

## 3. `.btproj` — Groot2 project file

**ESTABLISHED (shape): XML, not JSON** — and specifically *BT.CPP XML file
format plus project-level additions*. Evidence:

- Official tutorial (behaviortree.dev, tutorial_11, v4.11): "manually add the
  XML section directly to your `.xml` or **`.btproj`** file", and
  `BT::writeTreeNodesModelXML` output is importable into a `.btproj`.
- Wild real examples, husarion/husarion_ugv_ros
  (`behavior_trees/LightsBT.btproj`), fetched raw:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root BTCPP_format="4" project_name="LightsBT">
    <include path="lights.xml"/>
    <!-- Description of Node Models (used by Groot) -->
    <TreeNodesModel>
        <Action ID="CallSetLedAnimationService" editable="true">
            <input_port name="id">animation ID</input_port>
            ...
        </Action>
        ...
    </TreeNodesModel>
</root>
```

So a `.btproj` is a `<root BTCPP_format="4" project_name="…">` document that
may contain: `<include path="…"/>` references to sibling tree files (a
multi-file project is a `.btproj` plus the files it includes — see
DIT-ROBOTICS/Eurobot-2025-Main, `bt_m_config/Project.btproj` + `bt_*.xml`),
and a `<TreeNodesModel>` (per Discussion #707, models are duplicated into
each self-contained file *and* the project file, deliberately —
facontidavide, 2023-12-04).

`TreeNodesModel` contents — **what it contains is fully specified** by the
writer, `BT/src/xml_parsing.cpp:1237-1293` (`addNodeModelToXML`):

```
<TreeNodesModel>
  <Action|Condition|Control|Decorator|SubTree ID="<registration_name>">
    <input_port  name="…" [type="…"] [default="…"]>description</input_port>
    <output_port name="…" …/>
    <inout_port  name="…" …/>
    <MetadataFields>
      <Metadata k="v"/>          <!-- arbitrary key/value pairs -->
    </MetadataFields>
  </…>
</TreeNodesModel>
```

- Node-type tag is `toStr(model.type)` — one of `Action`, `Condition`,
  `Control`, `Decorator`, `SubTree` (from `NodeType`); `ID` is the
  registration name.
- Port attributes: `name` (always), `type` (demangled C++ type, omitted for
  untyped ports), `default` (omitted when empty); element text = port
  description.
- Subtree models may declare port defaults, giving **subtree default
  remappings** (docs tutorial 18): `<SubTree ID="MoveRobot"><input_port
  name="frame" default="world"/>…` — an XML-level feature Groot2 understands
  but the runtime only uses via the parser, not via the model itself.
- Wild examples add an `editable="true"` attribute on node elements (husarion
  `.btproj`) — a **Groot2-side** attribute with no counterpart in
  `addNodeModelToXML`; Groot2 also has an open issue (#102, 2026-06) about a
  `<description>` element in the TreeNodesModel that Groot2 "does not display
  and does not preserve on save" — i.e. the model schema has a small
  Groot2-specific extension surface (`editable`, `<description>`) whose
  semantics are **not** pinned by the C++ side.

**NEEDS-OBSERVATION (the rest of the .btproj):** anything beyond
`project_name`, `<include>` list, and `TreeNodesModel` — e.g. per-project
settings (preferences mentioned in Discussion #30: hiding built-in nodes;
1.8.x changelog mentions themes, Groot1-compat save toggle, port validation
toggle — these live in Groot2 settings; whether they persist into the
`.btproj` on save or only in Groot2's settings store is unverified),
editor state (selected tree, viewport), and whether breakpoint sets
("Breakpoints can now be saved/loaded from file", 1.1.0) go in the project
file or a separate side file. Saving behavior of the exact attribute set is a
binary-only fact.

## 4. `.db3` — `SqliteLogger` schema

**ESTABLISHED** — schema fully specified; it is in the header and enforced by
the writer, and the header doc explicitly says the tables are
"self-explaining" enough for external tools (`bt_sqlite_logger.h:42-45`).
File extension must be `.db3` or `.btdb` (constructor,
`bt_sqlite_logger.cpp:66-69`). Multiple recordings append to the same file,
each with a distinct `session_id` (`:46-47`, constructor doc).

From `bt_sqlite_logger.h:13-32` (doc) and the actual DDL at
`bt_sqlite_logger.cpp:78-94` (which the writer executes with
`CREATE TABLE IF NOT EXISTS`):

```sql
CREATE TABLE Definitions (
    session_id INTEGER PRIMARY KEY AUTOINCREMENT,
    date       TEXT NOT NULL,   -- strftime: datetime('now','localtime')
    xml_tree   TEXT NOT NULL    -- WriteTreeToXML(tree, true, true)
);
CREATE TABLE Nodes (
    session_id INTEGER NOT NULL,
    fullpath   VARCHAR,         -- node->fullPath(): instance name or tree path
    node_uid   INTEGER NOT NULL
);
CREATE TABLE Transitions (
    timestamp  INTEGER PRIMARY KEY NOT NULL,  -- µs, monotonically increasing
    session_id INTEGER NOT NULL,
    node_uid   INTEGER NOT NULL,
    duration   INTEGER,                    -- µs spent RUNNING; 0 when not entering RUNNING
    state      INTEGER NOT NULL,           -- NodeStatus enum value
    extra_data VARCHAR                     -- user callback string, see below
);
```

Inserts (same file, `:101-131`, `:233-244`):

- One `Definitions` row per recording; `session_id` = max afterwards.
- One `Nodes` row per node (uid ↔ fullpath), so a UID in `Transitions`
  resolves to a human path for that session.
- `Transitions.timestamp` is `max(prev+1, now)` — **strictly increasing,
  gap-tolerant** (`:164`), i.e. it is the logger's monotonic µs counter,
  *not* the wall clock (contrast with `.btlog`, where the header anchors to
  epoch).
- `duration`: 0 unless the transition entered RUNNING (then it is the µs the
  node spent RUNNING until this transition out, `:166-190`).
- `extra_data`: populated by an optional user callback
  (`setAdditionalCallback`, `bt_sqlite_logger.h:66-71`) — free-form per-
  transition annotation the IDE can display.
- Timestamps are **relative to the monotonic start**, not epoch; a player
  that wants wall-clock times must anchor the first transition's timestamp
  to `date` (approximate) — a small semantic gap versus `.btlog`.

A reader of `.db3` needs only the three tables above. Groot2 reads exactly
this shape (that is what "compatible" means here), so an independent IDE can
treat `.db3` as a portable interchange format.

## 5. The XML tree format + metadata — what the IDE *must* read/write

**ESTABLISHED.** This is the part every Groot2-compatible tool already has a
well-specified grammar for:

- **File shape:** `<root BTCPP_format="4" [project_name="…"] …>`, optional
  `<include path="…"/>`, one or more `<BehaviorTree ID="…">` trees, and an
  optional `<TreeNodesModel>`. Parsing rules are enforced by
  `BT/src/xml_parsing.cpp` (e.g. `:355-359`: `BTCPP_format` must be present
  and must be `"4"`; 4.11 adds `BTCPP_version` as well, per 4.11 changelog).
- **Node elements:** tag name = registration name; attributes = port remaps
  (input and output port names/values), `name=` instance name,
  `ID=` for `<SubTree>`, `_autoremap="true"` (auto port-remapping,
  `:1098-1101`), and pre/post-condition script attributes `_failureIf`,
  `_successIf`, `_skipIf`, `_while` (pre) and `_onHalted`, `_onFailure`,
  `_onSuccess`, `_post` (post) — `tree_node.h:78-94`.
- **Runtime metadata** — what `WriteTreeToXML(tree, true, true)` adds (this
  is the exact output FULLTREE and `.btlog` carry,
  `addTreeToXML` in `xml_parsing.cpp:1294-1385`):
  - `_uid` — the runtime `uint16` UID on every node (`:1320`). **This is the
    key that joins tree XML → status buffer → transitions** in every format.
  - `_fullpath` — subtree instance path (`:1309`, `:1367`).
  - Each instantiated subtree written out as its **own**
    `<BehaviorTree ID="…">` block (`:1381-1385`); a `<SubTree>` in the parent
    inlines to the matching block (repeated instantiations consume successive
    blocks with the same ID — note from the btview docs; matches the loop).
  - Trailing `<TreeNodesModel>` with the factory's manifests
    (built-in nodes included when `add_builtin_models=true`), `:1388-1407`.
- **`TreeNodeManifest`** (`tree_node.h:36-42`) — the runtime model object:
  `{type: NodeType, registration_ID: string, ports: PortsList,
  metadata: KeyValueVector}`. This is the in-memory form of what
  `<TreeNodesModel>` serializes; `writeTreeNodesModelXML(factory,
  include_builtin)` produces a standalone models file (used by the Groot2
  "Import Models" button).

An IDE that parses the full BT.CPP 4 XML dialect (nodes, ports, remapping,
subtrees, `_autoremap`, pre/post condition scripts, metadata attributes,
`TreeNodesModel` incl. `MetadataFields`) therefore has the complete file-side
contract. The 4.11 release adds Schematron/XSD validation hooks
(`writeTreeXSD`, `writeTreeSchematron`, `tools/validate_xml.py`) that a
reimplementation can adopt as its own conformance tests.

## 6. Specified in open source vs proprietary-only

**Specified (fully, with byte-level grammar) by the open source:**

1. `.btlog` — magic, version, XML payload (incl. the exact `WriteTreeToXML`
   call), T0, 9-byte transition records, relative-time convention, IDLE
   included, prev_status discarded. §1.
2. Live-monitor protocol — ports 1667 (REQ/REP) / 1668 (PUB), 6/22-byte
   headers, all 12 request tags, STATUS buffer layout and the `10+prev`
   idle-remembrance encoding, MessagePack-JSON blackboard dumps, hook JSON
   shape, breakpoint lifecycle (including heartbeat-driven auto-disable at
   5 s), recording (`'r'`/`'t'`) payload layout. §2.
3. `.db3` — exact SQL schema and insert semantics. §4.
4. XML tree dialect + metadata (`_uid`, `_fullpath`, pre/post condition
   attributes, subtree blocks, `BTCPP_format="4"`) and the `TreeNodesModel`
   grammar including `MetadataFields`, port `type`/`default`/description. §3, §5.
5. The **hook/debugging feature set** an IDE must offer to match Groot2's
   PRO features: breakpoints (PRE/POST), node substitution/fault injection
   (`mode=REPLACE` + `desired_status`), enable/disable-all, dump, remove-all.
   The C++ side implements every one of these.
6. Blackboard custom-type extension path: `BT_JSON_CONVERTER` /
   `RegisterJsonDefinition<T>()` (`json_export.h:14-56`) — a tool showing
   blackboards only needs to handle nlohmann-JSON values, including nested
   objects/arrays/vectors as JSON types.

**Proprietary-only (must be learned from the Groot2 binary or its docs/UX):**

1. **`.btproj` beyond the observed shape** — exact attribute/element set on
   save (per-project settings, editor state, breakpoint persistence,
   Groot1-compat save option), and which settings persist in-file vs in
   Groot2's own settings store. Wild examples + Discussion #30 + 1.1.0
   changelog bound the possibilities; the exact saved form is unverified.
2. **`TreeNodesModel` Groot2 extensions** — `editable="true"` on node
   elements (husarion file), `<description>` elements (Groot2 issue #102:
   "not displayed, not preserved on save"). Small but real: a Groot2
   replacement that round-trips Groot2-saved files must accept (and ideally
   preserve) these.
3. **Gating/limits** — free tier caps the monitor at **20 nodes** and
   disables blackboard/breakpoints/fault-injection (PRO, €590/yr); those are
   product decisions, not protocol constraints. An independent IDE is free to
   lift them (the C++ protocol supports unlimited nodes — `node_uid` is
   `uint16` ⇒ ≤65535 nodes, per the `Transition` comment).
4. **Groot2-specific UX behaviors** — auto-zoom on breakpoint, hierarchy-
   analysis warnings (1.8.0), split view, themes, "Recent project" — none
   of these have an interface counterpart; they are product features to
   re-design, not protocols to match.
5. **License/EULA mechanics** — floating licenses, offline activation files
   (1.1.0/1.8.x changelog) — irrelevant to an independent implementation.
6. **Behavioral semantics of the 3.x compatibility path** in Groot2 itself
   (Groot2 1.1.0 "compatibility with the Groot2Publisher in BT.CPP 3.8+" —
   i.e. Groot2 can also consume a `PublisherZMQ` v3 stream; the wire format
   is open-source — btv/bt_logger FlatBuffers + 12-byte transitions — but
   *how* Groot2 handles it, e.g. which UI differs, is binary-only).

## Assessment

The compatibility surface of Groot2 is **almost entirely on the open-source
side of the wall**. Every byte-level interface a replacement IDE needs is
pinned in BehaviorTree.CPP's public headers and sources — `.btlog`
(magic/version/XML/T0/9-byte records), the live-monitor ZMQ protocol
(1667/1668, 6/22-byte headers, all 12 request tags, `10+prev` status
encoding, MessagePack-JSON blackboards, full hook/breakpoint JSON), the
`.db3` SQL schema, and the entire BT.CPP 4 XML dialect including
`_uid`/`_fullpath` metadata and `TreeNodesModel`. A Groot2-*compatible* IDE
is therefore ~90% specified by public sources: file formats are
deterministically producible (the same loggers can generate the fixtures),
the network protocol has a working independent re-implementation available
for cross-checking (btview, plus `btview mock` as a test double), and the
node-model XML is generated by the library itself (`writeTreeNodesModelXML`).
What is proprietary-only is small and low-stakes: the exact `.btproj`
save-shape beyond `<root project_name>` + `<include>` + `<TreeNodesModel>`
(and a couple of Groot2-only model attributes like `editable`), the free-tier
feature gating, and pure UX/product behavior. The one interface fact that
genuinely requires observing the Groot2 binary is the **polling cadence and
reconnect behavior** Groot2 uses (how often it sends `T`/`S`/`B`, how it
re-establishes after a publisher restart, and whether it sends anything on
the PUB socket beyond listening) — nothing in the open sources pins client
behavior, only server behavior, and the protocol is permissive enough
(client-driven polling) that a replacement IDE can choose its own cadence and
still be fully compatible.

## Open questions

1. **`.btproj` exact save-shape** — resolvable by saving projects in Groot2
   (free tier suffices) and diffing: expected additions are settings and
   possibly breakpoint sets; low effort, high confidence payoff.
2. **Groot2's polling cadence** — resolvable by pointing `btview` (or a
   small ZMQ probe) at a `Groot2Publisher` while Groot2 is attached, or by
   reading Groot2's own monitor settings UI (does it expose a refresh rate?).
3. **`TreeNodesModel` extension attributes** (`editable`, `<description>`,
   any others) — resolvable by the same save-and-diff; Groot2 issue #102
   confirms at least the `<description>` element exists and is
   non-persisted.
4. **Groot2's own monitor-log format** when it records a live session
   (1.1.0: "Log parsing will now use plugins") — whether the saved file is a
   plain `.btlog`/`.db3` or a Groot2-side container is unverified; the
   plugin note suggests Groot2 may define an extra wrapper, which would be
   the single largest remaining format risk for a Log-Replay replacement.
