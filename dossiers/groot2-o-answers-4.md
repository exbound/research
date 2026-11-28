# Groot2 save-behavior answers, fourth set — O-18b: the include-form sweep (2026-11-28)

**Reference:** Groot2 v1.10.0, PRO tier, editor mode, target BT.CPP **4.10**.

**Fixture set:** [`../fixtures/groot2/o-answers-4/`](../fixtures/groot2/o-answers-4/)
— self-authored BT.CPP v4-dialect inputs, the reference's own written
outputs, a [`MANIFEST.md`](../fixtures/groot2/o-answers-4/MANIFEST.md)
with every hash re-derived at publication, and the
[`PROTOCOL-4.md`](../fixtures/groot2/o-answers-4/PROTOCOL-4.md)
describing how each measurement was taken. All paths cited below are
relative to that directory unless otherwise noted; every byte-level
claim carries the file that shows it and the MANIFEST hash that pins
it. This is the continuation of
[`groot2-o-answers-3.md`](groot2-o-answers-3.md)
(O-18 and the duplicate-model question), which in turn continued
[`groot2-o-answers-2.md`](groot2-o-answers-2.md) (O-10…O-16) and
[`groot2-o-answers.md`](groot2-o-answers.md) (O-1…O-9):
**the include-form variable of the exporter file-copy question is
settled here** — no include form triggers copy-and-flatten, and the
project-file-only rule is established for unmodified trees regardless
of include form.

---

## O-18b — which include form triggers copy-and-flatten? **ESTABLISHED: none**

**Question** (carried open by the third set): the first set's O-4
export (2026-11-15) copied all four included tree files into a fresh
destination, flattened to basenames. The third set's O-18 exports
(project file only, unmodified trees) could not reproduce it with the
third set's simpler input. The sessions differ in at least the include
forms present (parent-relative, absolute, deep subdirectory); O-18
showed the ticked Save-column state controls **in-place** saves for
dirty trees, not destination copies. **O-18b** isolates the include-form
variable: the first set's O-4 project, re-exported unmodified, with
the absolute include removed, and with both the absolute and
parent-relative includes removed — the dialog left at its default
state throughout, the destination set fresh each time.

**Input:** three variants of the first set's O-4 project
(`in-original/`, `in-no-abs/`, `in-no-abs-no-parent/`), plus the
third set's `in-o18-proj/` as a control (its hash rows are in the
third set's MANIFEST). The variants differ only in which include
forms their `proj.btproj` carries:

| variant | includes in `proj.btproj` | unreferenced files on disk |
|---|---|---|
| `in-original/` | all four (subdir-relative, parent-relative, deep-subdir-relative, **absolute**) | none |
| `in-no-abs/` | subdir-relative, parent-relative, deep-subdir-relative | `abs.xml` |
| `in-no-abs-no-parent/` | subdir-relative, deep-subdir-relative | `abs.xml`, `outside.xml` |

**Method:** each variant (and the control) opened fresh, the export
dialog read verbatim before touching it, the destination set through
the Browse chooser to a fresh directory, confirmed, then the whole
destination directory tree enumerated and every file hashed. The
dialog was **not** manipulated — no ticks, no unticks — because the
question is what the **default** state does to each include form.

**The dialog (observed structure, every export in this set):**

- A `Save` | `File Name` table, one row per include. Every row shows
  the **basename** of the included file and the state token
  **`n/a`** — no checkbox control on any row (unmodified trees,
  consistent with the third set's O-18 finding). Row order for the
  four-include project: `tree2.xml`, `tree.xml`, `outside.xml`,
  `abs.xml` — the **reverse** of the project file's include order,
  again consistent with the third set.
- A `Project Path` field **pre-filled with the input project's own
  path** (the known hazard — never accepted); a **Browse** button;
  Cancel / OK. No copy-style or "include files" option anywhere in
  the dialog.

**THE ANSWER — no include form triggers copy-and-flatten:**

| destination | input | bytes | sha256 | files written |
|---|---|---:|---|---|
| `out-o18b-control/` | `in-o18-proj/` (control) | 321 | `4321fbe0…` | `proj.btproj` **only** |
| `out-o18b-original/` | `in-original/` (all four include forms) | 391 | `9ed5b92d…` | `proj.btproj` **only** |
| `out-o18b-no-abs/` | `in-no-abs/` (no absolute include) | 328 | `d45843c7…` | `proj.btproj` **only** |
| `out-o18b-no-abs-no-parent/` | `in-no-abs-no-parent/` (no absolute or parent-relative) | 311 | `58b02d6f…` | `proj.btproj` **only** |

Every destination contained **exactly one file** — the project file.
No tree file of any kind was copied to any destination, referenced or
not, in any of the four exports.

- The **control** (`out-o18b-control/`) reproduces the third set's
  project-file-only result for the same input (`in-o18-proj/`),
  confirming the capture state (unmodified trees, default dialog)
  behaves as expected.
- **`in-original/`** — the first set's O-4 project, **all four include
  forms present** — produced project-file-only. Session 1's
  copy-and-flatten (five files at the destination root, all
  basenames) is **not reproduced** by re-running the same project,
  unmodified, under these conditions.
- **`in-no-abs/`** and **`in-no-abs-no-parent/`** — removing the
  absolute include (and then the parent-relative include as well)
  changed nothing: project-file-only, and the **unreferenced** files
  left on disk (`abs.xml`; `abs.xml` and `outside.xml`) were **not**
  copied.

**The written project file — the exporter's canonical rewrite,
independent of include form.** Every `out-*/proj.btproj` in this set
is the reference's canonical rewrite of its input: 4-space indent,
`BTCPP_version="4.10"` stamp **added**, includes re-expressed
**destination-relative** (subdirectory segments **preserved** — not
flattened to basenames), ordered **ascending on the written path**,
and an empty `<TreeNodesModel/>` **appended**. The absolute include
is **re-expressed as a relative path** — resolved, then rewritten; no
absolute path survives in any output.

`out-o18b-original/proj.btproj` (`9ed5b92d…`, 391 B), the
four-include project's own written output, verbatim:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root BTCPP_format="4" BTCPP_version="4.10" project_name="O4">
    <include path="../in-original/deep/nest/tree2.xml"/>
    <include path="../in-original/sub/tree.xml"/>
    <include path="../outside.xml"/>
    <include path="../../o-fixtures/in-o4-paths/abs.xml"/>
    <!-- Description of Node Models (used by Groot) -->
    <TreeNodesModel/>
</root>
```

Note the last line: the absolute include, in the input, pointed at an
absolute path. In the written output it is **relative**
(`../../o-fixtures/in-o4-paths/abs.xml`) — the reference resolved the
absolute path at write time and re-expressed it relative to the
destination. The `path=` values in this published file are relative to
the **capture layout** (see PROTOCOL-4.md, "Published bytes"); the
rule — destination-relative, subdirs preserved, absolute re-expressed
as relative, ascending order — is visible in the bytes themselves.

`out-o18b-control/proj.btproj` (`4321fbe0…`, 321 B), the control
export, verbatim:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root BTCPP_format="4" BTCPP_version="4.10" project_name="O18">
    <include path="../../o-fixtures-3/in-o18-proj/sub/tree-b.xml"/>
    <include path="../../o-fixtures-3/in-o18-proj/tree-a.xml"/>
    <!-- Description of Node Models (used by Groot) -->
    <TreeNodesModel/>
</root>
```

---

## What this establishes for spec §3.3

For **unmodified trees**, the project-export rule is now
**ESTABLISHED** across the full range of include forms:

1. Write the project file **only** — no tree file is copied into the
   destination, regardless of the include form (subdir-relative,
   parent-relative, deep-subdir-relative, or absolute) or of whether
   other on-disk files sit beside the project unreferenced.
2. Rewrite every include **destination-relative**, preserving
   subdirectory segments; re-express any absolute include as a
   relative path (resolved, then rewritten).
3. Order the includes **ascending on the written path**.
4. Add the `BTCPP_version="4.10"` stamp.
5. Append an empty `<TreeNodesModel/>`.
6. Files on disk beside the project that are **not** referenced by
   any include are **never** copied.

This supersedes the first set's O-4 finding ("always flattens every
include to its bare basename, and copies every tree file beside the
project") — that result is a **dirty-state artifact**, not a property
of the include forms.

---

## What the first session's copy-and-flatten was

The first set's O-4 export — five files at the destination root, all
flattened to basenames, with the written `proj.btproj`'s includes
reduced to basenames — is **not** the behavior of an unmodified-tree
export in any include-form configuration. By elimination against this
set's result and the third set's O-18 result, the copy-and-flatten
behavior must involve **tree-modification (dirty) state at export
time** and/or another dialog-state variable that was different in
session 1 — not the include forms themselves.

What remains unmeasured, and is the natural next experiment: what a
**ticked** export (a dirty tree whose Save row is ticked, the
dialog's default for a dirty tree) does to the **destination** — as
distinct from the **in-place** save at the include path that the
third set's O-18 established for the same ticked state. The third set
measured that the ticked tree's **in-place** save landed at the
include path; it did not measure whether the destination also
received a copy in that state.

---

## What this closes and what stays open

- **Closes O-18b:** no include form triggers copy-and-flatten. The
  first set's O-4 copy-and-flatten result is a dirty-state artifact,
  not an include-form effect. The project-file-only export rule for
  unmodified trees is ESTABLISHED for all four include forms and for
  the unreferenced-file case.
- **Refines the first set's O-4:** the "always flattens to basename"
  claim is superseded — for unmodified trees the writer preserves
  subdirectory segments and re-expresses paths destination-relative;
  the flattening-to-basenames behavior is the dirty-state path.
- **Still open:** what a ticked (dirty-tree) export does to the
  **destination** (as distinct from the in-place save the third set
  established). O-3 (V3 `TreeNodesModel` rule, deferred) and O-7
  (reconnect back-off, non-blocking) remain open from the earlier
  sets, unchanged.

---

## Wall statement

This set contains text files only — no screenshots or images. No
capture-tooling or rig details appear in it; the record is the
input/output bytes, the MANIFEST hashes, and this findings record.
The published `out-*/proj.btproj` files carry the reference's own
written bytes, including `path=` values relative to the capture
layout, for the reason stated in PROTOCOL-4.md ("Published bytes");
the exporter rule is visible in the bytes regardless of the exact
layout at capture time.
