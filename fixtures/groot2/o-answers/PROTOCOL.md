# Protocol — how this set was captured

Purpose: byte-exact capture of the reference IDE's (Groot2 v1.10.0,
PRO tier, target BT.CPP 4.10) save behavior for the O-1…O-9 questions
of the implementation spec (§7). Inputs are self-authored BT.CPP
v4 dialect, small on purpose: the point is the bytes the reference
writes, not tree complexity. No vendor-authored content is in this
set.

## How each answer was obtained

| O-item | Route taken | Why |
|--------|-------------|-----|
| O-1, O-8 | open the corrected `in-o1-proj` project in the reference; right-click the tree entry in the project tree → **Save** (per-file save), first unmodified, then after a one-port edit | there is no project-level in-place save in the reference: the project row offers only rename / add / upgrade / reload / close, and Ctrl+S is unbound. The per-file Save in the tree entry's context menu is the only in-place write for tree files. |
| O-2 | open each corrected `in-o2-*` project; **Export Project** to a fresh directory | same reason as O-8 — the project file is only ever written by Export, never in place. The export output is the reference's project-write bytes. |
| O-4 | open `in-o4-paths/`, Export to a fresh directory; separately Export back to the project's own directory, then restore the inputs from backup | Export is save-as-to-directory: it writes a new project file and copies every include beside it. |
| O-5 | open `in-o5-single.xml` **directly** (not as a project); file entry → **Save as…** to a new path | isolates what the reference writes for a bare tree outside a project. |
| O-6 | open `in-o6-model/`, Export to a fresh directory | the fixture loads cleanly; no deviation needed. |
| O-9 | for each of the six `in-o9-awkward/` variants: open standalone, **Save as…** to a sibling `<name>-rt.xml` | keeps each original intact beside its reference-written twin. |

## Deviation: the as-authored O-1/O-2 trees do not load

The as-authored `in-o1-proj/tree.xml`, `in-o2-same/tree.xml` and
`in-o2-diff/tree.xml` (all three byte-identical) contain an
`<AlwaysSuccess>` **wrapping** a `<Script>` child. The reference
rejects the load with the dialog **"Error at line 6: An Action node
must not have any child"** — the same rule BehaviorTree.CPP's own
loader enforces. Opening such a project loads the project metadata
(including its custom node model) but no tree.

The `-corrected` copies therefore collapse that wrapper to
`<AlwaysSuccess/>` (a one-line change, outer tree shape otherwise
unchanged); the project files are byte-identical to the as-authored
inputs. The corrected copies are the inputs the measurements ran on,
and they are published here. The as-authored forms are also published,
because the rejection itself is a recorded behavior.

## What was verified at capture time

- Every input hash checked before the session; every input re-checked
  after, all unchanged (the in-place experiments ran on the corrected
  working copies; `in-o4-paths/` was intentionally overwritten once by
  the same-directory export and restored from backup, verified exact).
- Every output hash recorded where it was first written; the outputs
  in this set are those first-write bytes, byte-identical across
  repeated exports from the same input.
- The reference was left idle with no project open.

## What is deliberately not in this set

- O-3 (V3 `TreeNodesModel` embedding rule) is a later work-package
  item; nothing here is v3.
- O-7 (reconnect back-off) is a timing property, not a file property;
  nothing here measures it.
- No rig or capture-tooling details: this set stands on the
  input/output bytes and this protocol.
