# Groot2 save-behavior answers, third set — O-18 and the duplicate-model question (2026-11-25)

**Reference:** Groot2 v1.10.0, PRO tier (30-day trial route, dossier
G-3), editor mode, target BT.CPP **4.10**.

**Fixture set:** [`../fixtures/groot2/o-answers-3/`](../fixtures/groot2/o-answers-3/)
— self-authored BT.CPP v4-dialect inputs, the reference's own written
outputs, a [`MANIFEST.md`](../fixtures/groot2/o-answers-3/MANIFEST.md)
with every hash re-derived at publication, and the
[`PROTOCOL-3.md`](../fixtures/groot2/o-answers-3/PROTOCOL-3.md)
describing how each measurement was taken. All paths cited below are
relative to that directory unless otherwise noted; every byte-level
claim carries the file that shows it and the MANIFEST hash that pins
it. This is the continuation of
[`groot2-o-answers-2.md`](groot2-o-answers-2.md)
(O-10…O-16), which in turn continued
[`groot2-o-answers.md`](groot2-o-answers.md) (O-1…O-9): **O-18 is
answered here**, closing the exporter file-copy question the second
set left NEEDS-MEASUREMENT, plus a **duplicate-model-declaration**
behavior measured alongside it.

---

## O-18 — the export dialog's Save column and the file-copy question: **ESTABLISHED**

**Question** (spec §3.3; carried open by the second set): the two
prior recorded sessions disagree on what a project export writes —
one copied all trees into the destination, the other wrote only the
project file. What is the rule?

**Input:** `in-o18-proj/` (`dfeafb16…` / `f3009289…` /
`2a04b394…`) — a minimal project with two includes in
flat-then-subdir order (`tree-a.xml`, then `sub/tree-b.xml`).

**Method:** the same project exported **four** times (PROTOCOL-3.md),
the tree state at export time the only variable: (1) unmodified,
(2) re-opened fresh, unmodified, (3) both trees opened in the canvas,
unmodified, (4) one port of one tree modified and left dirty. Each
destination directory was enumerated after its export.

**The dialog (observed verbatim structure):** a header (paraphrased: the included files are listed and the modified ones selected for saving); a two-column
table (`Save` | `File Name`) with one row per include — the
**basenames** of both files, **in the reverse of the project file's
include order** (`tree-b.xml` before `tree-a.xml`); a `Project Path`
field **pre-filled with the input project's own path** (accepting the
pre-fill exports onto the input tree — hit in this session, see
below); a Browse button beside the field; Cancel and OK buttons.

**Answer 1 — the Save column is driven entirely by tree dirtiness:**

- An **unmodified** tree's row reads `n/a` and carries **no control**:
  clicking the cell and pressing space on the focused row change
  nothing (both tested, exports 1–3). The ticked state is **not
  settable by hand** on an unmodified row.
- A **modified (dirty)** tree's row becomes a **real checkbox, ticked
  by default** (export 4: `tree-a.xml` ticked, `tree-b.xml` still
  `n/a`).

**Answer 2 — the tick does not copy the tree into the destination. It
saves the tree in place, at its own path:**

With the tick left on, export 4's destination received
`proj.btproj` **only** — no tree file. The modified tree was
**rewritten at its source path** (the include path): the reference
canonicalized it in place — the port change committed, the version
stamp added, the model block appended — which is what a reference-mode
save of that tree produces at that location. The save target is
**where the tree's include points**, not the export destination.

**Answer 3 — all four exports produced a byte-identical project file:**

| destination | bytes | sha256 |
|---|---:|---|
| `out-o18-copy/proj.btproj` | 289 | `fc53554a…` |
| `out-o18-only/proj.btproj` | 289 | `fc53554a…` |
| `out-o18-open/proj.btproj` | 289 | `fc53554a…` |
| `out-o18-mod/proj.btproj` | 289 | `fc53554a…` |

The written project file, verbatim (`out-o18-copy/proj.btproj`, and
byte-identical in all four):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root BTCPP_format="4" BTCPP_version="4.10" project_name="O18">
    <include path="../in-o18-proj/sub/tree-b.xml"/>
    <include path="../in-o18-proj/tree-a.xml"/>
    <!-- Description of Node Models (used by Groot) -->
    <TreeNodesModel/>
</root>
```

From the input (`dfeafb16…`): the version stamp **added**, the include
order reversed (matching the dialog's row order; a two-include fixture cannot distinguish reversal from descending-path sort — both fit)
rewritten **destination-relative** — the subdirectory segment
**preserved** (`sub/tree-b.xml` keeps its `sub/` prefix; nothing is
basename-flattened) — 4-space indent, and an empty
`<TreeNodesModel/>` appended. This matches the second set's
exporter include rule; the new fact is that the copy behavior does
not change a single byte of it.

**Answer 4 — the two prior sessions' contradictory results resolve:**

- The **first session's** "trees copied into the destination,
  flattened to basenames" is consistent with the trees' source
  paths **pointing at the destination directory at save time**: the
  ticked save then lands the tree files where the includes point
  (the destination), and the written includes read as bare basenames
  relative to the project file beside them.
- The **second session's** "project file only" is the **unmodified**
  case: nothing tickable, no in-place save triggered, project file
  with repointed includes only.

For this fixture set (two unmodified trees, one flat and one
subdirectory include, no absolute or parent-relative paths), the
exporter wrote the project file only. **HYPOTHESIS** (not
established): the first session's copy-and-flatten result may be a
second variable — possibly the absolute or parent-relative include
forms, possibly dirty state in that session — not explained by the
Save-column rule alone. The first set's O-4 in-place export (into
the project's own directory, with subdirectory, parent-relative and
absolute includes, producing flattened basenames and sibling copies)
remains unexplained by this measurement. A deciding experiment would
re-run the O-4 fixture set unmodified into a fresh directory, then
with the absolute include removed.

**Answer 5 — the dialog's structure (as observed, all four exports):**

Two rows — the subdirectory tree (`tree-b`) is listed **first**, i.e.
in the **reverse of the project file's include order** — a
`Project Path` field **pre-filled with the input project's own path**
(accepting the pre-fill writes the export onto the input tree; hit
three times this session, inputs restored byte-exact each time), and
**Browse** / **Cancel** / **OK** buttons. The Browse and OK hit
areas nearly overlap (the input-rewrite incidents of the next
paragraph).

**Incidents (recorded for future sessions):** confirming the export
dialog while its Project Path field still held the pre-filled **input**
path rewrote the input project file in place, with the reference's
canonical rewrite of that file (stamp added, include order changed, 4-
space indent, model block appended) — hit three times before the
Browse/OK button geometry was measured; the input was restored
byte-exact each time and re-verified against the MANIFEST
(`dfeafb16…`). The Browse and OK buttons' hit areas nearly
overlap, so clicks aimed at Browse fired OK. The port editor
accepts a code-completion popup on Return (it must be escaped
first). The dialog's Project Path field
left-scrolls and can render without the leading slash (the written
path was verified absolute each time).

---

## Duplicate model declarations — **ESTABLISHED**

**Question:** a project's `TreeNodesModel` and an included tree's own
`TreeNodesModel` both declare the **same** model entry — accepted,
warned, or refused? If accepted, what does a save write?

**Fixture:** `o18-dupmodel/` — `proj.btproj` (`55f46f98…`) declares
`<Action ID="X"/>` in its own model block and includes `tree-x.xml`,
whose **own** model block declares the **byte-identical** line
(one `    <Action ID="X"/>` in each file).

**Result:**

- **Accepted silently.** The project loaded with **no dialog, no
  warning, no error**. The model list showed the entry **once** (two
  byte-identical declarations did not produce two UI entries), and
  the `X` node instantiated in the canvas with no complaint.
- **A per-file save does not merge the project's declaration into the tree.** Dirtying the tree and
  saving it (project-tree panel, per-file Save) rewrote
  `tree-x.xml` to the published post-save bytes (`c1af551b…`,
  339 B): the written model block contains the entry **exactly
  once** — `grep -c 'Action ID="X"'` on the written file returns 1 —
  with the port change committed and the version stamp added
  (reference-normalized, 2-space body, same shape as the in-place
  export save above).
- **The project file is untouched by the tree save** —
  `proj.btproj` hash unchanged before and after
  (`55f46f98…`); the project's own declaration survives.

**Combined answer:** a byte-identical duplicate declaration across
the project and its included trees is a **legal, silent** input; the
reference merges it to a single model and a per-file save emits the
entry once. A compatible loader must not refuse this input, and a
compatible writer does not add the project-declared model to the tree file on save.

---

## What this closes and what stays open

- **Closes the second set's NEEDS-MEASUREMENT** on the exporter's
  file-copy behavior: the copy-vs-project-only split is tree
  dirtiness (the Save column's tick), the tick is an in-place save at
  the include path, and the destination receives exactly the project
  file. The first set's O-4 output (trees beside the destination,
  basename includes) is the case where the trees' source paths point
  at the destination. The unified exporter rule for spec §3.3 is
  therefore: *write the project file; rewrite includes relative to
  the destination (sorted, subpaths preserved); stamp it; append the
  project-level model list; and for every dirty tree, save it in
  place at its include path.*
- **Still open from the earlier sets:** O-3 (V3 `TreeNodesModel`
  rule, deferred to the later editing milestone) and O-7 (reconnect
  back-off, non-blocking timing property).

---

## Wall statement

This set contains text files only — no screenshots or images. No
capture-tooling or rig details appear in it; the record is the
input/output bytes, the MANIFEST hashes, and this findings record.
The incident notes describe the reference's own dialogs and buttons,
not the rig that drove them.
