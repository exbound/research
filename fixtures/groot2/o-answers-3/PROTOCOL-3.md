# Protocol — how this set was captured (Groot2, O-18 + duplicate-model)

Purpose: byte-exact capture of the reference IDE's (Groot2 v1.10.0,
PRO tier, target BT.CPP 4.10) **project-export** behavior — the
O-18 question the two prior sets could not resolve, because their
recorded sessions disagree on whether an export copies the tree files
into the destination — plus a duplicate-model-declaration experiment.
Every input is self-authored BT.CPP v4 dialect; no vendor-authored
content is in this set. The point is the bytes the reference writes,
not tree complexity.

**Required state before any input.** The reference launched idle with
no project open; format mode V4, editor mode, confirmed by the title
bar. Inputs are read-only: every input hash was checked against this
set's MANIFEST before the session and re-checked after every export,
restored byte-exact where an in-place write hit an input (below).
All export writes were directed to the `out-o18-*` directories.

## The O-18 experiment — four exports of one project, dialog state the only variable

Same project (`in-o18-proj/`), exported four times into four
directories; the only thing that differs between exports is the
in-memory state of the trees at export time:

| # | Tree state at export | Dialog rows | Destination |
|---|----------------------|-------------|-------------|
| 1 | unmodified | both `n/a` (tick-attempted, no change) | `out-o18-copy/` |
| 2 | unmodified (project closed and re-opened fresh) | both `n/a` | `out-o18-only/` |
| 3 | both trees **opened in the canvas**, unmodified | both `n/a` | `out-o18-open/` |
| 4 | `tree-a`'s port **modified, left dirty** | `tree-b.xml` `n/a`; `tree-a.xml` **ticked checkbox** | `out-o18-mod/` |

Each export: read the dialog verbatim first; set the destination
through the dialog's file chooser (type the full destination path
including the `proj.btproj` name, then confirm, then confirm the
export dialog); then hash **every** file present in the destination
directory and record which files were **not** written. The export
dialog's Project Path field pre-fills the **input** project's path;
accepting that pre-fill exports onto the input tree (see
"Restoration below").

**Dialog observed (verbatim structure, all four exports):**

- Header: the included files are listed; modified files are selected
  for saving (paraphrased)
- A two-column table, `Save` | `File Name`, one row per include. Both
  files' **basenames** appear; the row order is **tree-b.xml,
  tree-a.xml** — the **reverse** of the project file's include order
  (`tree-a.xml`, then `sub/tree-b.xml`).
- A `Project Path` field (pre-filled, see above) with a **Browse**
  button beside it, and **Cancel** / **OK** buttons.

**Restoration (inputs hit by in-place writes).** Two classes of
incident landed reference writes on input paths during the session;
each was restored byte-exact and re-verified against this manifest
before continuing:

1. **Pre-filled-path exports.** Confirming the export dialog while its
   Project Path field still held the input's own path rewrote
   `in-o18-proj/proj.btproj` **in place** — the reference's canonical
   rewrite of that file (version stamp added, include order ascending on the written path (coincides with the
   dialog's row order, 4-space indent, empty model block appended),
   156 → 259 bytes, no tree files written anywhere. This happened
   three times before the geometry below was measured; the input was
   restored to `dfeafb16…` each time.
2. **The ticked-save in-place write (export #4).** With `tree-a`
   dirty, the ticked row's save wrote the modified tree **at its
   include path — the source path**: `in-o18-proj/tree-a.xml` was
   rewritten with the committed port change plus the version stamp,
   184 → 288 bytes. Restored to `f3009289…`.

The four published `out-o18-*/proj.btproj` files are the
destination's contents after each export — each directory held
**exactly** that one file.

## The duplicate-model experiment (`o18-dupmodel/`)

Fixture authored in-session: a project (`DupModel`) whose own
`TreeNodesModel` declares `Action ID="X"`, including one tree whose
**own** `TreeNodesModel` declares the byte-identical entry. Procedure:

1. Open the project; record any dialog, warning or model-panel state
   (none; the model list showed `X` **once**).
2. Open the tree in the canvas; confirm the `X` node instantiates.
3. Dirty the tree (change one port value), then **per-file Save**
   (project-tree panel, right-click → Save) and record the written
   model block.

Result recorded: clean load, no prompt of any kind; the written tree
file's model block contains the entry **exactly once**; the project
file's bytes were unchanged before and after the save. The published
`tree-x.xml` is the post-save file; the published `proj.btproj` is
the as-authored input.

## What was verified at capture time

- Every input hash checked before the session and after every export —
  all three `in-o18-proj/` files match the MANIFEST after each
  incident (above).
- All four export destinations enumerated by directory listing after
  each export; each held only `proj.btproj`, all four
  byte-identical (`fc53554a…`).
- The duplicate-model project file's hash compared before and after
  the per-file save — unchanged (`55f46f98…`).
- The reference was left idle with no project open, V4 mode.

## What is deliberately not in this set

- No screenshots or image files.
- No capture-tooling or rig details; the set stands on the
  input/output bytes and this protocol.
