# Protocol — how this set was captured (Groot2, O-18b: the include-form sweep)

Purpose: byte-exact capture of the reference IDE's (Groot2 v1.10.0,
PRO tier, target BT.CPP 4.10) **project-export** behavior under the
specific question the third set left open — **which include form, if
any, triggers the first set's O-4 copy-and-flatten result** (all
included trees copied into the destination, flattened to basenames),
as opposed to the project-file-only behavior O-18 established for
unmodified trees. Every input is self-authored BT.CPP v4 dialect; no
vendor-authored content is in this set. The point is the bytes the
reference writes, not tree complexity.

**Required state before any input.** The reference launched idle with
no project open; format mode V4, editor mode, confirmed by the title
bar. Inputs are read-only: every input hash was checked against this
set's MANIFEST before the session and re-checked after every export;
all export writes were directed to fresh `out-o18b-*` directories, and
no input was ever rewritten.

## The three input variants

Each variant is a copy of the first set's O-4 project (`in-o4-paths/`,
project name "O4", all four include forms) with the supporting tree
files laid out so that every relative include resolves to a file that
is present: a sibling `outside.xml` in each variant directory serves
as the target of the variant's `../outside.xml` include, and the
set-root `outside.xml` (this set's own) is the target that resolves
one level up from each variant directory, matching the O-4 source
layout.

- **`in-original/`** — the O-4 project **as-is**: all four includes,
  `sub/tree.xml`, `../outside.xml`, `deep/nest/tree2.xml`, and the
  **absolute**-path include (kept at its original path string, which
  resolves against the O-4 source tree, left in place and untouched
  for the session).
- **`in-no-abs/`** — `in-original/` with the **absolute** include
  removed from `proj.btproj`. `abs.xml` remains on disk in the same
  relative location, **unreferenced**.
- **`in-no-abs-no-parent/`** — `in-original/` with **both** the
  absolute and the parent-relative includes removed from
  `proj.btproj`. `abs.xml` and `outside.xml` remain on disk,
  **unreferenced**.

**Control input (not in this set):** the third set's `in-o18-proj/`
project (two unmodified includes; the exact input of the O-18
session), published in `../o-answers-3/` with its own MANIFEST row. It
is referenced here because it is the control export's input; its
hashes are carried in the third set's MANIFEST.

## The experiment — one export per variant, dialog left at defaults

For each variant (and for the control project, first, to confirm the
capture state), the same minimal sequence: open the project fresh
(close the previous project first), read and record the export dialog
verbatim before touching it, set the destination, confirm, then
enumerate and hash everything written. The dialog state is **not**
manipulated — the O-18b question is what the default state does to
each include form, so any deliberate tick/untick would confound it.

**Dialog observed (verbatim structure, every export in this set):**

- Header: the included files are listed; modified files are selected
  for saving (paraphrased).
- A two-column table, `Save` | `File Name`, one row per include. Every
  row shows the **basename** of the included file and the state token
  **`n/a`** (no checkbox control present on any row — unmodified
  trees, per the third set's O-18 finding). Row order: `tree2.xml`,
  `tree.xml`, `outside.xml`, `abs.xml` for the four-include project —
  the **reverse** of the project file's include order, consistent with
  the third set's dialog-row finding.
- A `Project Path` field **pre-filled with the input project's own
  path** (the known hazard from the third set — never accepted here;
  the destination was set through the Browse chooser only) and a
  **Browse** button beside it, plus **Cancel** / **OK** buttons.
- No copy-style or "include files" option anywhere in the dialog.

**Destination handling.** Each destination was set through the
dialog's **Browse** file chooser: the full destination path (the
directory plus `proj.btproj`) typed into the chooser's file-name
field, confirmed, then the export dialog's OK confirmed. The
export dialog's own `Project Path` field pre-fills the **input**
project's path; accepting that pre-fill would export onto the input
tree (the third set's incident), so it was never accepted — each
destination was verified against the written file's own `path=`
values before continuing.

**Post-export enumeration.** After each export, the whole destination
directory tree was listed; every file present was hashed, and the
expected files (project file; each referenced tree file — or the
absence of one) were recorded, along with whether subdirectory
structure was preserved or flattened.

**Result (every export, control and all three variants):** the
destination directory held **exactly one file** — `proj.btproj` — and
nothing else. No tree file was copied to any destination, referenced
or not, and the written `proj.btproj` in each case is the reference's
canonical rewrite of its input (4-space indent, `BTCPP_version="4.10"`
stamp added, includes re-expressed **destination-relative** with
subdirectory segments preserved — the absolute include re-expressed as
a relative path — and an empty `<TreeNodesModel/>` appended). The four
written project files are byte-distinct only in their repointed
`path=` values and the number of includes; each is the expected
canonical form of its own input.

## Published bytes

The `out-*/proj.btproj` files published here are the reference's own
written bytes, kept verbatim. Their `path=` attribute values are
relative to the **capture layout** (the directory structure the
reference saw when it wrote them), not to this published layout — the
same convention as the third set's `out-*/proj.btproj` files, which
likewise carry paths relative to the capture layout. They are
published as-is because they are the reference's output, not a
reconstruction; the point of publishing them is the rewrite **rule**
(destination-relative, subdirectories preserved, absolute re-expressed
as relative, ascending on the written path), which is visible in the
bytes regardless of the exact capture layout.

## What was verified at capture time

- Every input hash checked before the session and after every export —
  all files in all three variant directories, the set-root
  `outside.xml`, and the control project's files (the third set's
  `in-o18-proj/`) unchanged throughout.
- All four export destinations enumerated by directory listing after
  each export; each held only `proj.btproj`, nothing else.
- The reference was left idle with no project open, V4 mode.

## What is deliberately not in this set

- No screenshots or image files.
- No capture-tooling or rig details; the set stands on the
  input/output bytes and this protocol.
