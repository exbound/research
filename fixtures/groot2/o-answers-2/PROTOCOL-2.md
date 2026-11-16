# Protocol — how this set was captured (Groot2, O-10…O-16)

Purpose: byte-exact capture of the reference IDE's (Groot2 v1.10.0,
PRO tier, target BT.CPP 4.10) save behavior for the O-10…O-16
questions of the implementation spec's open-questions list, the
continuation of the O-1…O-9 set in
[`../o-answers/PROTOCOL.md`](../o-answers/PROTOCOL.md). Every input is
self-authored BT.CPP v4 dialect (O-12 is v3); no vendor-authored
content is in this set. The point is the bytes the reference writes,
not tree complexity.

**Required state before any input.** The reference launched idle with
no project open; format mode V4, editor mode; O-12 additionally
switched the app to V3 mode for the duration of its measurements and
back to V4 afterwards. Inputs are read-only: every input hash was
checked against this set's MANIFEST before the session and re-checked
after, all unchanged — the reference writes never land on an input
path. All reference writes land in the `out-*` directories below.

**Capture routes (carried over from the first set).** Per-file save =
right-click the tree entry in the project-tree panel → **Save as…**
to the out path. Project write = **Export Project** (the project row's
export control) to a fresh destination — the only project-file write
the reference has. Standalone tree = opened directly, **Save as…** to
the out path.

## How each answer was obtained

| O-item | Route taken | Output |
|--------|-------------|--------|
| O-10 | open the corrected O-10 project (the as-authored input is refused — see "Refusals and corrected copies"); per-file save the tree | `out-o10/tree.xml` |
| O-11 | open the corrected O-11 project (both trees load); Export Project to a fresh directory; repeat with a modified tree; repeat with the destination subdirectories pre-created | `out-o11/proj.btproj` (+ the in-place-save evidence file) |
| O-12 | switch the reference to **V3 mode**; open the as-authored v3 project; per-file save each tree; Export Project | `out-o12/tree.xml`, `out-o12/tree-crlf.xml`, `out-o12-export/proj.btproj` |
| O-13 | open each standalone file directly and **Save as** to `out-o13/<same name>` | the five `out-o13/*` files |
| O-14 | open the as-authored O-14 project; per-file save both trees; Export Project | `out-o14/*`, `out-o14-export/proj.btproj` |
| O-16 (a) | open the newer-stamp project (note the dialog and its choice); per-file save the tree; Export Project; then attempt to lower the target with the project open | `out-o16a/*` |
| O-16 (b,c) | open the older-stamp project (note the dialog and its choice); per-file save each tree; Export Project | `out-o16b/*`, `out-o16b-export/proj.btproj` |

## Refusals and corrected copies

Three inputs are refused as authored; the refusal itself is recorded
behavior, and the measurements ran on minimally corrected copies
published alongside them:

- **O-10** — the as-authored tree's `Script` nodes carry ports the
  built-in `Script` model does not declare; the reference refuses the
  load (the node's port does not match the model). The corrected copy
  (`out-o10-corrected/`) re-tags the four nodes to a custom `WrapProbe`
  Action model declared with the same five ports, keeping attribute
  counts and value lengths identical; the wrapping question is
  count/length-driven, so the name swap preserves the measurement.
- **O-11** — `b/tree.xml` carries an `Action` node with a child, which
  the reference refuses ("An Action node must not have any child").
  The corrected copy collapses the wrapper to a self-closing tag; the
  tree stays a distinct, loadable tree (its content is irrelevant to
  the basename-collision question).
- **O-13** — `attr-escapes.xml` (undeclared `Script` port, corrected by
  re-tagging the node to a custom model with the same ports, escape
  payload byte-identical) and `two-trees-blank.xml` / `mixed-forms.xml`
  (an `Action` node with a child, corrected by collapsing the wrapper
  while keeping the layout and form variants under test).

The as-authored forms are published unchanged because a compatible
implementation must reproduce the **rejection**, not the load.

## What was verified at capture time

- Every input hash checked before the session and re-checked after —
  all unchanged, no input was ever written.
- Every output hash recorded at first write; outputs in this set are
  those first-write bytes.
- The O-11 in-place evidence: the modified tree was saved by the
  reference **at its own source path** during export (the published
  `out-o11/export-inplace-saved-a-tree.evidence.xml` holds the
  reference's own written bytes, after which the source was restored
  to its manifest bytes).
- The O-12 byte-identity: `out-o12/tree.xml` and
  `out-o12/tree-crlf.xml` are byte-identical (both `b2e76631…`); the
  evidence copies for O-11 and O-12 exports are likewise byte-identical
  to their primary files.
- The reference was left idle with no project open, V4 mode, no custom
  models.

## What is deliberately not in this set

- No screenshots or image files.
- No capture-tooling, screen-driving or rig details; the set stands on
  the input/output bytes and this protocol.
- The O-14 model-order caveat (the authored order already reads
  alphabetically at the byte level) is recorded in the findings
  record, not re-measured here.
