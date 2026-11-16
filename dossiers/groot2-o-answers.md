# Groot2 save-behavior answers — the O-1…O-9 fixture set (2026-11-15)

**Reference:** Groot2 v1.10.0, PRO tier (30-day trial route, dossier G-3),
BT.CPP compatibility target **4.10**, editor mode.

**Fixture set:** [`../fixtures/groot2/o-answers/`](../fixtures/groot2/o-answers/)
— self-authored BT.CPP v4-dialect inputs (`in-*`), the reference's own
written output (`out-*`, and the `-rt.xml` siblings inside
`in-o9-awkward/`), a [`MANIFEST.md`](../fixtures/groot2/o-answers/MANIFEST.md)
with every hash, and the
[`PROTOCOL.md`](../fixtures/groot2/o-answers/PROTOCOL.md) describing how
each measurement was taken. All paths cited below are relative to that
directory unless otherwise noted; every byte-level claim carries the
file that shows it and the MANIFEST hash that pins it.

These answers close the O-items of the implementation spec's
§7 Open questions to the extent this session covered them: **O-1, O-2,
O-4, O-5, O-6, O-8 and O-9 are answered**; O-7 (reconnect back-off) was
not measured and remains open; O-3 (V3 `TreeNodesModel` rule) is
deferred to the later editing milestone per the spec.

---

## The canonical tree-write style (shared by every tree file the reference writes)

Observed identically in at least seven distinct write paths (in-place
per-file save, unmodified per-file save, save-as of a standalone tree,
the export copies in `out-o4/`, and the six O-9 round-trips):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root BTCPP_format="4"
      BTCPP_version="4.10">
  <BehaviorTree ID="Main">
    <Sequence>
      <AlwaysSuccess/>
    </Sequence>
  </BehaviorTree>

  <!-- Description of Node Models (used by Groot) -->
  <TreeNodesModel/>

</root>
```

The byte-level rules, each visible in
[`out-o5/in-o5-single.xml`](../fixtures/groot2/o-answers/out-o5/in-o5-single.xml)
(hash `413174e6a602c1a294f2612966e522ef01c417769c0f732e45a212860c88b5ca`):

- **XML declaration** on line 1, double-quoted attributes, UTF-8.
- **`<root>` tag split across two lines**: `BTCPP_format` on the first
  line, `BTCPP_version` on a **continuation line with a 6-space
  indent**, no comma, no other reordering.
- **2-space indent** in the tree body.
- **After `</BehaviorTree>`**: a blank line, the comment
  `<!-- Description of Node Models (used by Groot) -->`, the
  `<TreeNodesModel>` element (empty on a bare per-file tree save),
  a blank line, then `</root>`.
- **Single trailing LF** after `</root>` (confirmed by byte
  inspection of the file ending: `...Model/>\n\n</root>\n`).

The **project file** write differs from the tree write — see O-1.

---

## O-1 — the `BTCPP_version` stamp bytes: **ESTABLISHED**

**Question** (spec §7): the exact bytes of the stamp — inline in
`.btproj`, continuation line in tree XML, with separators, indentation
width and position among attributes.

**Tree XML:** the stamp is **not** placed on any modified node and
**not** inlined into a single-line root tag. It is a **continuation
line of the split `<root>` tag**:

```
<root BTCPP_format="4"
      BTCPP_version="4.10">
```

Evidence: `out-o5/in-o5-single.xml` (standalone save-as),
`out-o2-same/tree.xml` (hash `c800d2b6…`, identical across all
per-file tree writes), `out-o6/tree.xml`, all `out-o4/*.xml`, and all
six `in-o9-awkward/*-rt.xml`. The continuation-line indent is exactly
**6 spaces** (two levels of the body indent, i.e. aligned to the
`BTCPP_format` attribute on the first line).

**`.btproj`:** the stamp is **inline**, on the same line as the
existing root attributes, in the attribute order **format, version,
project_name**:

```
<root BTCPP_format="4" BTCPP_version="4.10" project_name="O2S">
```

Evidence: `out-o2-same/proj.btproj` (hash
`29234f689893dd54d9ef2b37dc457c0161bf49b03a76322d5238d348b6321e15`).
`out-o2-diff/proj.btproj` (different input, see O-2) and
`out-o4/proj.btproj` show the same order with different
`project_name` values.

**Attribute position:** `BTCPP_version` is **appended after**
`BTCPP_format` (and before `project_name` when present) — not
prepended, not reordered relative to the other attributes. Both file
types preserve existing attribute order otherwise.

**Project file style (differs from the tree style):** the written
`.btproj` uses **4-space indent** for body children (vs 2-space in
tree files), the same blank-line + comment + `TreeNodesModel` block
before `</root>`, and the **same single trailing LF**. Example,
`out-o2-same/proj.btproj` (347 B, hash above):

```
<?xml version="1.0" encoding="UTF-8"?>
<root BTCPP_format="4" BTCPP_version="4.10" project_name="O2S">
    <include path="tree.xml"/>
    <!-- Description of Node Models (used by Groot) -->
    <TreeNodesModel/>
</root>
```

(The tree include is re-emitted at the top of the body, as the
include block precedes the `TreeNodesModel` block.)

**The modified-node claim is settled:** no stamp of any kind is
attached to the modified node; the only stamp is the one on `<root>`,
and its placement is the one described above for each file type.

---

## O-2 — stamp behavior when `BTCPP_version` is already present: **ESTABLISHED**

Two cases were measured through the project-Export route (the only
route that writes a project file; see O-8):

**Same value as the reference target** — input
`in-o2-same-corrected/proj.btproj` already carries
`BTCPP_version="4.10"`:

- **Output** (`out-o2-same/proj.btproj`, hash
  `29234f689893dd54d9ef2b37dc457c0161bf49b03a76322d5238d348b6321e15`):
  the stamp is **preserved verbatim** — `BTCPP_version="4.10"`,
  attribute order `format, version, project_name`, no reordering.

**Different minor version** — input
`in-o2-diff-corrected/proj.btproj` carries `BTCPP_version="4.9"`:

- Opening the project triggers a **version-mismatch dialog** that
  pre-announces the upgrade: it informs that the files will be saved as
  the reference's target the next time they are saved, that nothing is
  written now, and offers three choices — upgrade the files' in-memory
  target, cancel, or retarget the reference to the older version
  (the retarget is changeable later in preferences). See "The
  version-mismatch dialog (functional description)" below for the full
  functional description. *(2026-11-15 correction: the verbatim vendor
  dialog text was replaced with a functional paraphrase.)*
- Choosing the **in-memory upgrade** (verified the on-disk hash was
  unchanged before export) and then exporting produced
  `out-o2-diff/proj.btproj` (hash
  `f63f6f919e5daf7a1a4e47e2f1ff079bdefd21a3974f67f7eaef97bdb2360513`):
  the written stamp is `BTCPP_version="4.10"` — **normalized to the
  reference's own target version**, not preserved as `4.9`.

**Combined O-2 answer:** a same-major/stamp already matching the target
is **preserved verbatim** (position, value, attribute order all
unchanged). A stamp with a **different minor** is **rewritten to the
reference's current target version** on write, and the reference
**pre-announces** this rewrite in a dialog before any bytes change.

---

## O-4 — include-path rewriting: **ESTABLISHED**

Input `in-o4-paths/` has four `<include path=…>` forms (MANIFEST
hashes for each input):

| Input `path=` value | Input file |
|---|---|
| `sub/tree.xml` | `sub/tree.xml` (hash `64e3c096…`) |
| `../outside.xml` | `../outside.xml` (hash `e826c43c…`) |
| `deep/nest/tree2.xml` | `deep/nest/tree2.xml` (hash `56584888…`) |
| `/abs/path/to/…/abs.xml` | `abs.xml` (hash `9fcf7501…`) |

Output `out-o4/proj.btproj` (hash
`cb25fb1419a3fd2b52e135a3cd86c0126ad2f22126e7530eb6308d671bc5743a`)
shows **every** include rewritten to **just the basename**:

```xml
<include path="abs.xml"/>
<include path="outside.xml"/>
<include path="tree.xml"/>
<include path="tree2.xml"/>
```

All four corresponding tree files were copied **beside the project
file** (flattened; `sub/` and `deep/` subdirectories not recreated in
the output), each fully normalized per the canonical tree-write style
(`out-o4/tree.xml` hash `3ed575d4…`, `out-o4/tree2.xml`
`e8d30c26…`, `out-o4/abs.xml` `8698f4d7…`, `out-o4/outside.xml`
`521dd388…`).

The export was also repeated **back to the project's own directory**
(in-place case): the written project file was **byte-identical** to
the new-directory export (same hash `cb25fb14…`), with the three
subdirectory trees flattened into the project directory as siblings.
No input file outside the project directory (`outside.xml`'s parent
sibling; `sub/` and `deep/` originals) was modified.

**Combined O-4 answer:** the project write **always flattens every
include to its bare basename**, regardless of the input form
(subdirectory, parent-relative `..`, deep nesting, absolute path), and
regardless of the export destination directory. The included tree
files are **copied beside the project**, normalized, every time the
project is written.

This **refines and corrects** the G-1 dossier finding ("relative to
the saved file's own location — bare filename when saved alongside,
relative path otherwise"): the reference **always** produces a bare
basename; it never writes a non-trivial relative or absolute `path=`
in the output. The G-1 corpus happened to contain only same-directory
includes, which is why the distinction did not surface there.

---

## O-5 — save-as of a single (standalone) tree: **ESTABLISHED**

**Question** (spec §7, F9): what bytes does the reference write for a
save-as of a single tree opened directly (not via a project)?

Input: `in-o5-single.xml` (hash `e3a012ab…`), opened **directly**
(no project wrapper in the input). The reference loads it as an
implicit single-file project in memory. Right-click the file entry
→ **Save as…** → new path `out-o5/in-o5-single.xml`.

Output `out-o5/in-o5-single.xml` (286 B, hash
`413174e6a602c1a294f2612966e522ef01c417769c0f732e45a212860c88b5ca`):

- **Filename preserved** (`in-o5-single.xml`), no `.btproj` suffix
  added, no extra project file created alongside it.
- **No project wrapper**: the output is a **bare tree file** —
  `<root BTCPP_format="4" BTCPP_version="4.10">` with the
  `<BehaviorTree ID="Main">` directly inside; **no `<include>`**
  element, no `project_name` attribute, no `TreeNodesModel` with
  custom entries (just the empty `<TreeNodesModel/>` the canonical
  style appends).
- **`BehaviorTree ID="Main"` preserved** verbatim.
- All other bytes follow the **canonical tree-write style** exactly
  (split-root stamp on continuation line, comment + empty
  `TreeNodesModel` block, single trailing LF).

**Contrast with the earlier "wrapper" finding (dossier G-1):** G-1's
corpus was a v4 `.btproj` that already contained the tree as an
`<include>`; saving that project produced a project file (the
"wrapper"). Here, the **input** is a bare tree with no project; the
output is a bare tree. The reference's behavior for a
standalone-tree save-as is to write **a bare tree file,
reference-normalized**, not to wrap it in a new one-include
`project` file. (A save-as **from inside an existing project** — i.e.
the export route — does produce a project file; see O-2/O-4.)

---

## O-6 — `TreeNodesModel` property preservation: **ESTABLISHED**

**Question** (spec §7, F6): what does the reference's save do with
`<description>` elements and `editable="true"` attributes in a
`TreeNodesModel`?

Input `in-o6-model/proj.btproj` (hash `660835e08311c72c8fce08f8dfc29c3f312767ae2fc482b43de6ac6ffcb72ebe`)
carries three model entries, isolating each property:

```xml
<Action ID="PlainModel"/>              <!-- neither -->
<Action ID="EditableModel" editable="true">
  <description>My custom editable node</description>
  <input_port name="msg"/>
</Action>                              <!-- both -->
<Condition ID="PlainCondition" editable="true"/>  <!-- editable only -->
```

Export output `out-o6/proj.btproj` (hash
`be9523d942f730f9ac9229f652331cf77976060ec3aa32b318a765d8f119a3dc`):

| Property | Input | Output |
|---|---|---|
| `editable="true"` (both `Action` and `Condition`) | present | **kept** |
| `<description>` child element | present (EditableModel only) | **dropped** |
| `<input_port name="msg"/>` | present (EditableModel only) | **kept** |

The entries are **re-sorted alphabetically by `ID`** on write: input
order is `PlainModel, EditableModel, PlainCondition`; output order is
`EditableModel, PlainCondition, PlainModel`.

**Two additional write-style facts visible in `out-o6/`:**

1. **The written tree carries the models the tree itself uses** —
   **NEEDS-MEASUREMENT** (2026-11-15 correction). The earlier claim
   that "a project export embeds the project's models into **every**
   copied tree file" is contradicted by this set's own bytes:
   `out-o2-same/tree.xml` (hash `c800d2b6…`) and
   `out-o2-diff/tree.xml` (same hash) are export copies whose
   `TreeNodesModel` block is **empty**, even though their projects
   (`in-o2-same-corrected/proj.btproj` hash `63f7dd0c…`,
   `in-o2-diff-corrected/proj.btproj` hash `6880dc39…`) each declare
   the `SayHello` model. Every written tree in this set fits the
   rule "**a written tree file carries exactly the declared models
   the tree uses**": `out-o6/tree.xml` (hash
   `f1a2822e9e492c7b614bf45b2033649bc5b6b9a30a0e49f4791e78b1b45bacf9`)
   uses all three of its project's models and carries all three; the
   two `out-o2-*/tree.xml` files use none and carry none; and
   `in-o9-awkward/comments-rt.xml` (hash `129a8f83…`) drops the
   `X` model that `in-o9-awkward/comments.xml` (hash `b6da0f10…`)
   declared but did not use. The O-14 measurement settles the rule:
   a project declaring two models whose tree uses one, written
   through the same routes.

2. **Tags with two observed attributes wrap in tree files; the
   fixtures cannot separate an attribute-count rule from a length
   threshold** (2026-11-15 correction). The one-line two-attribute
   counter-example exists only in `.btproj` files, which never wrap:
   `<Action ID="SayHello" editable="true"/>` stays on one line in
   `out-o2-same/proj.btproj` (hash `29234f68…`) and
   `out-o2-diff/proj.btproj` (hash `f63f6f91…`). In tree files,
   every observed two-attribute tag wraps: in `out-o6/tree.xml`,
   both `<Action ID="EditableModel" editable="true">` and
   `<Condition ID="PlainCondition" editable="true"/>` are split onto
   continuation lines, while the one-attribute
   `<Action ID="PlainModel"/>` is not. In `.btproj` files,
   the root tag carries **three** attributes — `BTCPP_format`,
   `BTCPP_version` and `project_name` (two where `project_name` is
   absent) — all inline on one line, and project-file roots in
   this set are **never wrapped** (so even three or four
   attributes stay on one line). The fixture set contains no
   tree-file tag that stays on one line with two attributes, so
   "two or more attributes" and a rendered-length threshold are
   **not separable from these bytes**; the O-10 measurement will
   decide.

**Combined O-6 answer:** `editable="true"` is **kept**;
`<description>` is **dropped** (confirming the vendor's own
public-issue report that the description "does not display and does
not preserve on save"); entries are **re-sorted alphabetically** on
write. The model-embedding rule is **NEEDS-MEASUREMENT** — the set
supports "a written tree carries exactly the declared models the
tree uses," but the two-models-one-used case (O-14) has not been
measured. The attribute-wrap behavior is observed but
**unexplained** by these fixtures; the deciding measurement is
O-10.

---

## O-8 — which files an unmodified project save writes: **ESTABLISHED** (premise overturned)

**Question** (spec §7, F6): on a project save, is an **unmodified**
included tree file left untouched (stamp included), or rewritten?

The G-1 finding — "a bare tree file is rewritten **only when its
content has been modified**" — was based on the save-as/export route.
This session measured the **in-place per-file Save** route, which is
the reference's only in-place write path for tree files (the project
file has no in-place save at all — see below).

**Route and setup** (see PROTOCOL.md): corrected `in-o1-proj`
project opened, nothing touched, right-click the tree entry in the
project tree → **Save** (the per-file Save in that context menu;
Ctrl+S is **not bound** in the reference).

**Result:** `tree.xml` **was rewritten despite being unmodified**
— 206 → 309 bytes, hash `e51b8e8d…` →
`c800d2b68ef9720f66d6230378444c959be330eb6094168caf01c524571cb6d2`,
byte-identical to the reference's export-flow tree output (same
split-root stamp, same empty `TreeNodesModel` block appended).
`proj.btproj` was **not** touched (same hash, mtime unchanged).

The project-level context menu offers: Rename / Add New File /
Add File / Upgrade to BT.CPP / Reload / Close — **no Save item**.
The only project-file write in the reference is the **Export
Project** flow (choose a directory; writes a new project file +
copies all includes there). **Ctrl+S is unbound** — pressing it on a
dirty tree produces no dialog and no disk write.

**Combined O-8 answer:** the reference rewrites a tree file
**unconditionally** on the per-file Save route — an unmodified tree
is rewritten to its canonical form; it is **not** left untouched.
The project file is **never** written in place; it is only ever
produced by Export. This **overturns the "only when modified"
premise** for the per-file Save route (the G-1 finding, which was
measured through the export/save-as route where a content change is
still the trigger for rewriting an include's *content* — but the
reference writes a fresh canonical copy regardless).

---

## O-9 — preservation of awkward-but-legal XML properties: **ESTABLISHED**

**Question** (spec §7, F6–F9): does the reference preserve quoting
style, the XML declaration, character references, CRLF line endings,
or a missing trailing newline?

Six single-property variants were each opened standalone and saved
via **Save as…** to a sibling `-rt.xml` file (the original stays
beside it for comparison). All six original inputs and all six
reference-written outputs are in `in-o9-awkward/` (MANIFEST hashes):

| Variant | Property varied | Input hash | Output hash | Survived? |
|---|---|---|---|---|
| `altquote.xml` | single-quoted attributes | `7fd503f3…` | `129a8f83…` | **No** — all quotes rewritten to double |
| `nodecl.xml` | no `<?xml …?>` declaration | `ccc4e514…` | `129a8f83…` | **No** — declaration added |
| `charrefs.xml` | `&#65;` in a name attribute | `5020b9f7…` | `9f3c62c9…` | **No** — `&#65;` resolved to literal `A` (output: `ID="Ain&amp;tree"`) |
| `crlf.xml` | CRLF line endings throughout | `3974a9b3…` | `129a8f83…` | **No** — all lines LF in output |
| `notrail.xml` | no trailing newline | `b1183644…` | `129a8f83…` | **No** — trailing LF added |
| `comments.xml` | comments before root, between elements, after close | `b6da0f10…` | `129a8f83…` | **No** — all source comments stripped; only the writer's own injected comment remains |

**Five of the six outputs are byte-identical**
(`129a8f83e9ec93b9e83727dde5fd1cb4305476da190783a17476588baf821fe4`):
the writer produces one canonical form and **no** lexical property
from the source survives. The `charrefs.xml` output differs only
because `&#65;` decodes to `A`, giving a different `BehaviorTree ID`
value — which then appears literally (and with `&` still
escaped as `&amp;`) in the output.

**Combined O-9 answer:** the reference is a **normalizing writer**:
it rewrites to a single canonical form on every tree-file save, and
**no** lexical or syntactic property of the source (quoting,
declaration, character references, line-ending style, trailing
newline, comments) is preserved. **Byte-stability of the source is
not a reference guarantee** for any of these properties. This means
the implementation's **lossless save (F5) — byte-identical round-trip of the
source — is not the reference's behavior**; the reference-compatible
save (F6) and the lossless save are genuinely different operations,
and F5's "lossless" guarantee cannot be met by simply mimicking the
reference's writer.

---

## Additional recorded behaviors

### The loader rejects an Action node with a child

The as-authored `in-o1-proj/tree.xml`, `in-o2-same/tree.xml` and
`in-o2-diff/tree.xml` (all three byte-identical, hash
`31ce572eff48443be6843eea06ba6e6a9ed6215da3cc3465ee6c6f7630df93af`)
contain `<AlwaysSuccess><Script code="return true;"/></AlwaysSuccess>`.
Opening any project that includes this tree produces the error dialog:

> **"Error at line 6: An Action node must not have any child"**

The project **metadata** (including the custom node model, here
`SayHello`) **does** load; the tree does not. This is consistent with
BehaviorTree.CPP's own loader rule and is recorded here because the
**as-authored** input files are part of this fixture set (published
alongside the corrected copies) and a compatible implementation must
reproduce the **rejection**, not the load. The corrected copies
(`in-o1-proj-corrected/`, `in-o2-same-corrected/`,
`in-o2-diff-corrected/`) collapse the wrapper to
`<AlwaysSuccess/>` so the reference will load them, and the
measurements above ran on the corrected copies.

### The version-mismatch dialog (functional description)

Opening a project whose `BTCPP_version` stamp is **older** than the
reference's current target (here, `4.9` vs target `4.10`) presents a
version-mismatch dialog before the project loads. *(2026-11-15
correction: the verbatim vendor dialog text was replaced with a
functional paraphrase.)*

**Functional description.** The dialog pre-announces the upgrade:
it informs that the files will be saved as the reference's target
version the next time they are saved, and that nothing is written at
open time. It offers three choices:

1. **Upgrade the files' in-memory target** — the files are re-stamped
   to the reference's current target in memory; the on-disk bytes are
   untouched until a subsequent save or export.
2. **Cancel** — the open is abandoned.
3. **Retarget the reference application to the older version** —
   the reference's own compatibility target is switched to the
   files' declared version (here, `4.9`), changing the reference's
   behavior for all subsequent opens and saves. This retarget is
   changeable later in the reference's preferences.

In the O-2 different-minor measurement, choice 1 was exercised; the
on-disk hash was verified unchanged after the open, and the
subsequent export produced the normalized `BTCPP_version="4.10"`
stamp. Choice 3 would have retargeted the reference to `4.9` and was
not exercised in this session. The dialog is the pre-announcement of
the O-2 different-minor normalization behavior.

---

## What remains open

- **O-3** (V3 `TreeNodesModel` embedding rule): deferred to the later
  editing milestone per the spec. Nothing in this set is v3.
- **O-7** (reconnect back-off after a publisher restart): not
  measured this session (a timing property, not a file property);
  non-blocking per the spec — the implementation's own bounded
  exponential-back-off policy stands.
- **O-10** (attribute-wrap rule, tree files): not yet measured; the
  O-6 wrap observations cannot separate an attribute-count rule from
  a rendered-length threshold.
- **O-14** (model-embedding rule for a project whose tree uses only
  some of its declared models): not yet measured; the O-6 embedding
  claim is NEEDS-MEASUREMENT until this settles it.
- **O-15** (the O-4 parent-relative include target, `outside.xml`):
  the input file was omitted from the published fixture set — the
  MANIFEST hash `e826c43c…` was recorded, but only the hash was
  published, not the bytes. **Published 2026-11-15** at
  [`outside.xml`](../fixtures/groot2/o-answers/outside.xml)
  (named for its role as the parent-relative include target, placed
  where the `in-o4-paths/` project's `../outside.xml` include
  resolves it); the file's sha256 matches the recorded MANIFEST row.
