# Groot2 save-behavior answers, second set — O-10…O-16 (2026-11-15)

**Reference:** Groot2 v1.10.0, PRO tier (30-day trial route, dossier
G-3), editor mode. Baseline target BT.CPP **4.10**; O-12 was measured
in **V3 Compatibility mode** and switched back to V4 afterwards;
O-16(a) temporarily retargets the application to 4.11.

**Fixture set:** [`../fixtures/groot2/o-answers-2/`](../fixtures/groot2/o-answers-2/)
— self-authored BT.CPP v4-dialect inputs (`in-*`, O-12 is v3), the
reference's own written output (`out-*`), the minimally corrected
copies for the three fixtures the reference refused to load as
authored, a [`MANIFEST.md`](../fixtures/groot2/o-answers-2/MANIFEST.md)
with every hash re-derived at publication, and the
[`PROTOCOL-2.md`](../fixtures/groot2/o-answers-2/PROTOCOL-2.md)
describing how each measurement was taken. All paths cited below are
relative to that directory unless otherwise noted; every byte-level
claim carries the file that shows it and the MANIFEST hash that pins
it. This is the continuation of
[`groot2-o-answers.md`](groot2-o-answers.md) (O-1…O-9): **O-10, O-11,
O-12, O-13, O-14 and O-16 are answered here**; O-15 was closed by the
first set's 2026-11-15 publication of `outside.xml`.

---

## O-10 — attribute wrapping in written tree files: **ESTABLISHED**

**Question** (spec §3.2 rule 7): does the reference wrap multi-
attribute elements in written tree files, on what condition, and
where do continuation lines align?

Input: the corrected O-10 project
(`out-o10-corrected/`, MANIFEST hashes `a88cb1db…` / `a2d42087…`) —
a `Sequence` holding four `WrapProbe` nodes whose attribute counts and
value lengths mirror the as-authored `Script` nodes: 2 short
attributes, 3 short, 3 with one 200-char value, and 5 mixed
(short/long). The as-authored input is published too
(`in-o10-wrap/`, `ea9b7146…` / `63a3fb28…`) — it cannot load: the
reference's built-in `Script` model declares exactly one port, and
the nodes here carry several (recorded refusal, see "Loader
behaviors").

Output: [`out-o10/tree.xml`](../fixtures/groot2/o-answers-2/out-o10/tree.xml)
(hash `a750cce4…`, 1679 B). Measured directly from the bytes:

- **The wrap is attribute-count-driven, threshold ≥ 2.** Every element
  with two or more attributes is written one attribute per
  continuation line: the 2-attribute node gets one continuation, the
  3-attribute nodes three, the 5-attribute node five. One-attribute
  elements (`<Sequence>`, the root's first line) stay on one line.
  Nothing is grouped two-per-line anywhere in the file.
- **Attribute value length is irrelevant.** The nodes carrying
  200-character values wrap identically line-for-line to their short-
  value siblings with the same attribute count (compare the
  `A1long` and `A3short` tags in the output).
- **Continuation alignment = the first-attribute column.** For a
  body node at 6-space indent, continuations sit at column 17 (the
  element indent plus the length of the open tag and first attribute
  name position) — e.g. `code="return true;"` starts at the same
  column on every wrapped node, and the model rows' `<input_port>`
  pairs align at the model-row first-attribute column in the embedded
  block.
- **The root stamp is on its own 6-space continuation line:**
  `BTCPP_version="4.10"` indented to the `BTCPP_format` attribute —
  the O-1 rule, confirmed again.
- The embedded `TreeNodesModel` block carries the used `WrapProbe`
  model with its five ports, each `input_port` row wrapped
  one-attribute-per-line on the same rule.

**Bonus finding (feeds the O-14 and spec §3.2 model-port rule): the
writer emits EVERY declared port of a node's model as an attribute on
write, filling unused ports with the empty string.** Input `A2short`
carried only `name` and `code`; the written element carries all six
— `name`, `code`, `x=""`, `y=""`, `z=""`, `port=""`. The attribute set
on write is the node model's declared ports ∪ `name`. (This is a
configuration of the reference's own settings file — the option
controls whether unused ports are emitted as empty attributes — not
an intrinsic property of the format; the reference as configured
emits them.)

---

## O-11 — duplicate basenames on project export: **ESTABLISHED**

**Question** (spec §3.3): two includes sharing a basename
(`a/tree.xml`, `b/tree.xml`) — what does an export into a fresh
destination write, and can the two trees clobber each other?

Input: `in-o11-collide/` (as-authored; `b/tree.xml` refused — an
`Action` with a child; corrected copy published). Output:
[`out-o11/proj.btproj`](../fixtures/groot2/o-answers-2/out-o11/proj.btproj)
(hash `3ef5f163…`).

**A basename collision is not possible through the export flow.**
Measured across three exports (both trees unmodified; the `TreeA`
node modified in-canvas; and with `a/`/`b/` subdirectories
**pre-created** in the destination):

- The destination receives **only the project file** — never a tree
  file. The `a/` and `b/` subdirectories, when pre-created, are left
  **empty**.
- **Both `<include>` lines are written, sorted**, and **repointed to
  the source files** relative to the new project file:
  `path="../out-o11-corrected/a/tree.xml"` and
  `path="../out-o11-corrected/b/tree.xml"`.
- The modified tree is **saved in place at its own source path**, not
  copied into the destination: the reference's own bytes for the
  modified `TreeA` are preserved at
  [`out-o11/export-inplace-saved-a-tree.evidence.xml`](../fixtures/groot2/o-answers-2/out-o11/export-inplace-saved-a-tree.evidence.xml)
  (hash `f554dbdf…`) — reference-normalized, 2-space body, empty
  model block — and the destination still received no tree file.
- The export dialog lists both rows with the flat basename
  `tree.xml` and no visible disambiguation, but no name-based write
  into a shared directory ever happens; no error or refusal at any
  step.

**Combined O-11 answer:** the exporter writes the project file only;
includes are re-pointed at their (last-saved) source locations; a
modified tree is written back to its source. The two-include
same-basename case therefore cannot clobber. This also refines the
first set's O-4 finding: for **subdirectory** includes the exporter
never places a tree file in the destination, while a **flat** include
(next to the project) is copied — the exporter's tree-placement rule
differs by include form.

---

## O-12 — V3-mode save normalization: **ESTABLISHED**

**Question** (spec §3.5, F7): what does a V3-mode save write — does
it normalize, and how?

Input: `in-o12-v3/` — a v3 project (`BTCPP_format="3"`, `f5e5c4a8…`)
whose tree uses the v3 spellings `SetBlackboard` and `SubTree`
(tree hash `1585e824…`), plus `tree-crlf.xml` — the same tree
content with **CRLF line endings** (`5a09c356…`).

Open behavior: **no v3→v4 conversion prompt** in V3 mode (none
expected). Instead a **model-importer offer** appeared for the
undeclared `Script` node type — the reference offered to create the
model manually (action node, detected input port `code`), noting the
model can instead be generated with the library's own model-XML
writer. Accepted with defaults; both `BehaviorTree`s loaded. No
version prompt of any kind on open.

Output: [`out-o12/tree.xml`](../fixtures/groot2/o-answers-2/out-o12/tree.xml)
and [`out-o12/tree-crlf.xml`](../fixtures/groot2/o-answers-2/out-o12/tree-crlf.xml),
**byte-identical** (both hash `b2e76631…`, 647 B).

**A V3-mode save normalizes with the same writer as V4:**

- **2-space indent — not the v3 samples' 4-space.** The written body
  is 2-space indented throughout.
- **The models comment + `TreeNodesModel` are appended** (blank line,
  `<!-- Description of Node Models (used by Groot) -->`, the model
  block, blank line before `</root>`) — the V4 writer's layout, in a
  v3-format file.
- **No version stamp in V3.** The written root stays
  `<root BTCPP_format="3">` — no `BTCPP_version` line, versus the V4
  writer's `BTCPP_version` continuation.
- **CRLF normalized to LF** — the CRLF input's save is byte-identical
  to the LF input's save (both `b2e76631…`). This answers the O-9
  CRLF question for V3 the same way: line endings are not preserved.
- **V3 spellings kept** — `SetBlackboard` and `SubTree` are not
  translated.
- **Attribute order is normalized against the model's declared
  order, not source order.** Input `<SetBlackboard
  output_key="k" value="1"/>` is written `<SetBlackboard
  value="1" output_key="k"/>` (wrapped, continuation aligned at the
  first-attribute column): the writer emits attributes in the
  node model's port-declaration order.
- Model rows sorted alphabetically (`MoveBase`, `Script`), each
  `editable="true"` kept (the importer-created `Script` row is
  `editable="true"` too), ports one-per-line aligned.
- The two `BehaviorTree` elements stay in one file, separated by one
  blank line.

**Project file:** the exported
[`out-o12-export/proj.btproj`](../fixtures/groot2/o-answers-2/out-o12-export/proj.btproj)
(hash `ace4d1ed…`) shows the writer difference that matters for
§3.2-vs-§3.3: **4-space body, model rows on one line** — the tree-
file wrap rule does **not** apply to project files
(`<Action ID="MoveBase" editable="true">` stays on one line with two
attributes). Root: `<root BTCPP_format="3" project_name="O12">` —
**no version stamp** in the V3 project file either. The include is
rewritten to the tree's last-saved location
(`../out-o12/tree-crlf.xml` — see the exporter include rule below).

---

## O-13 — escaping, content and layout in the canonical writer: **ESTABLISHED**

Five standalone tree files, each varying one property; inputs in
`in-o13-escape/`, the reference's saves in `out-o13/` (MANIFEST
hashes throughout).

**Attribute escaping** — input `attr-escapes.xml`
(`a30cdbad…`), value `a &quot;b&quot; &lt;tag&gt; end 'single' é`
(refused as authored — undeclared `Script` port; corrected copy
published, escape payload byte-identical). Output
[`out-o13/attr-escapes.xml`](../fixtures/groot2/o-answers-2/out-o13/attr-escapes.xml)
(`8b4d0729…`) carries, verbatim:

```
note="a &quot;b&quot; &lt; tag &gt; end &apos;single&apos; é"
```

Per character:

| Input | Written |
|-------|---------|
| `"` | `&quot;` (escaped, not raw) |
| `<` | `&lt;` |
| `>` | `&gt;` |
| literal `'` | `&apos;` — the writer **escapes single quotes** |
| non-ASCII (`é`) | **literal UTF-8 byte**, not a numeric reference |
| `&amp;` (from O-9) | kept as the entity |
| numeric character reference (from O-9, `&#65;`) | **resolved** to the literal character |

Together with O-9: the writer resolves numeric references, keeps or
emits the five named-entity forms (`&amp;`, `&lt;`, `&gt;`, `&quot;`,
`&apos;`), and emits non-ASCII raw.

**Node text content is silently dropped.** Input `content.xml`
(`c12e9e95…`) carries `text inside a node` inside the `Script`
element. Output [`out-o13/content.xml`](../fixtures/groot2/o-answers-2/out-o13/content.xml)
(`c36cd2a4…`) writes `<Script name="withText" code="return true;"/>`
— self-closing, no text, not refused, not echoed. Element text is
not part of the dialect's model.

**Multiple blank lines between trees collapse to one.** Input
`two-trees-blank.xml` (`ee61b40e…`, two blank lines between the
trees; refused as authored — an `Action` with a child; corrected copy
preserves the double gap). Output `out-o13/two-trees-blank.xml`
(`9e7ef7aa…`): exactly **one** blank line between the
`BehaviorTree` elements (then the blank line + comment + model block
+ blank line before `</root>`, the canonical style).

**Tab indentation is rewritten to 2 spaces throughout.** Input
`two-trees-tab.xml` (`a4a4131d…`, first tree tab-indented, second
2-space) loads as authored; output `out-o13/two-trees-tab.xml`
(`96536c65…`) is fully 2-space — no per-tree preservation.

**Both open/close element forms collapse to self-closing.** Input
`mixed-forms.xml` (`6c1eb8dd…`, corrected copy keeps an empty
open/close node and an **attribute-bearing** open/close node). Output
`out-o13/mixed-forms.xml` (`6fbabb86…`): every element written
`<X …/>` — **an element with an attribute is never emitted in
open/close form.** (Confirms the O-6 observation from the v4 corpus.)

---

## O-14 — the model-selection rule (priority): **ESTABLISHED — the rule survives**

**Question** (spec §3.2 rule 8, hypothesis: *a written tree carries
exactly the declared models the tree uses*): a project declaring
**two** custom models, one tree using exactly one of them, the other
using neither.

Input: `in-o14-models/` — `proj.btproj` (`8a79cd23…`) declares
`UnusedModel` and `UsedModel` (editable, one `input_port`);
`tree.xml` (`920a1e96…`, tree `UsesOne`) uses only `UsedModel`;
`unused.xml` (`b2c2ba67…`, tree `UsesNone`) uses neither.

- **Per-file save of the using tree** —
  [`out-o14/tree.xml`](../fixtures/groot2/o-answers-2/out-o14/tree.xml)
  (hash `c7ab6811…`): the model block carries **`UsedModel` only** —
  `UnusedModel` is **not** embedded; `editable="true"` and the
  `<input_port>` kept; the `UsedModel` element carries its used port
  with the source value (`p="x"`).
- **Per-file save of the using-none tree** —
  [`out-o14/unused.xml`](../fixtures/groot2/o-answers-2/out-o14/unused.xml)
  (hash `63843ae3…`): the model block is **empty**
  (`<TreeNodesModel/>`) — neither declared model embedded.
- **The exported project** —
  [`out-o14-export/proj.btproj`](../fixtures/groot2/o-answers-2/out-o14-export/proj.btproj)
  (hash `37a41f0b…`): carries **both** declared models. The model
  list is a property of the **project**, not of each tree. (No tree
  copies were written — both rows unmodified — so the "exported tree
  copies carry both" branch was moot and, per O-11, a modified
  tree's export write would have gone to its source path anyway.)

**Caveat, recorded:** the fixture's "reverse-alphabetical" authoring
premise was byte-wrong — `UnusedModel` sorts **before** `UsedModel`
(`n` < `s` in the second byte), so the authored order already read
alphabetically and the written order matching it does not
independently prove the sort. The alphabetical model-block sort
remains evidenced by O-6's own bytes (the
`PlainModel, EditableModel, PlainCondition` → `EditableModel,
PlainCondition, PlainModel` reordering), not by this fixture.

**Combined O-14 answer:** the refined hypothesis **survives** as the
per-file rule — a written tree file embeds exactly the declared
models it uses (and an empty block when none); the project file
embeds the project-level declared list.

---

## O-16 — stamp edge cases: **ESTABLISHED**

Three cases, all v4, reference target 4.10; inputs in
`in-o16-stamps/` (MANIFEST hashes).

**(a) Newer stamp (4.11 > 4.10).** Opening `proj-411.btproj`
(`b7437050…`) prompts: the reference offers to **switch the
application's target** to 4.11 (the change can be made again later in
preferences), or to cancel. Accepting retargets the application —
the title reflects the new target — and **all subsequent saves stamp
the current target**: the per-file save
[`out-o16a/t411.xml`](../fixtures/groot2/o-answers-2/out-o16a/t411.xml)
(`f8a5e215…`) is written `BTCPP_format="4"` +
`BTCPP_version="4.11"` (the 4.11 **preserved**, not forced to 4.10,
not dropped), and the export
[`out-o16a/proj.btproj`](../fixtures/groot2/o-answers-2/out-o16a/proj.btproj)
(`51f9c414…`) is `<root BTCPP_format="4" BTCPP_version="4.11"
project_name="O16A">` — the newer stamp preserved in the project file
too, with the include kept relative (`path="t411.xml"`).

**Lowering the target with the project open is refused**: switching
preferences back to 4.10 while O-16A is open presents a close-first
message — the open files are saved as 4.11, and lowering the target
with them open would relabel them as an older version. The project
had to be closed first.

**(b) Older stamp in a tree file (4.9 < 4.10).** Opening
`proj-stamp.btproj` (`52a1c121…`) — which includes `stamped.xml`
carrying `BTCPP_version="4.9"` — prompts on open with the
**three-file pre-announcement**: the files use an older BT.CPP (the
newest is 4.9), opening keeps the current target, and the files are
saved as 4.10 the next time they are saved (nothing written now), or
the reference's target can be switched to 4.9. Choosing to keep the
target: the per-file save
[`out-o16b/stamped.xml`](../fixtures/groot2/o-answers-2/out-o16b/stamped.xml)
(`7e7235d9…`) **rewrites the 4.9 stamp to 4.10**, exactly as
announced — the older stamp is not preserved and not dropped. The
export [`out-o16b-export/proj.btproj`](../fixtures/groot2/o-answers-2/out-o16b-export/proj.btproj)
(`00c523ed…`) carries the target stamp `BTCPP_version="4.10"` and
writes **both** includes **sorted** and repointed at the trees'
last-saved locations (`ordered.xml` before `stamped.xml`, input order
was `stamped` then `ordered`).

**(c) Root-attribute order normalization.** Input `ordered.xml`
(`07f344d6…`) has a **foreign `project_name` attribute before
`BTCPP_format`**. Output
[`out-o16b/ordered.xml`](../fixtures/groot2/o-answers-2/out-o16b/ordered.xml)
(`078cd5fc…`): the written root is normalized to the canonical
`BTCPP_format="4"` first line + `BTCPP_version="4.10"`
**continuation**, regardless of source attribute order — and the
foreign `project_name` is **dropped** from the written tree-file root
(`project_name` is project-file-only).

---

## The unified exporter include rule

New, measured consistently across O-11, O-12, O-14 and O-16 —
this unblocks the spec's project-write section:

**For each `<include>`: keep the line if the referenced file exists
relative to the new project file's location; otherwise rewrite it to
point at the tree's last-saved location, relative to the new project
file. The lines are sorted.**

Evidence: O-11 (`a/`, `b/` subdirectory includes — kept as
relative-to-new-project paths to the sources, `3ef5f163…`); O-12
(include rewritten to the standalone tree's last-saved location,
`ace4d1ed…`); O-14 (both subdirectory includes repointed and sorted,
`37a41f0b…`); O-16a (flat include **kept** — the file exists relative
to the new project, `51f9c414…`); O-16b (both includes repointed and
**sorted**, `00c523ed…`). The exporter resolves include paths from
the tree files' last-saved association rather than from the loaded
project's include lines (observed persisting across a project
close/open within one reference session), which is why a fresh
re-open of the same project exported the same rewritten paths.

---

## Additional recorded behaviors (all from reference dialogs at load)

- **An `Action` node with a child is refused:** "An Action node must
  not have any child" — hit by the as-authored O-11 `b/tree.xml` and
  two of the O-13 inputs (corrected copies published); same rule as
  the first set's O-1/O-2 rejections.
- **A node port not in its declared model is refused:** the
  as-authored O-10 input (undeclared ports on the built-in `Script`
  model) and the as-authored O-13 `attr-escapes.xml` both failed the
  load on this check (the reference's own dialog names the node and
  port).
- **A builtin model cannot be overridden:** declaring a custom model
  with a builtin name is refused by the reference (its own dialog,
  functional description: the builtin model cannot be overridden).
  This is why the O-10 corrected copy re-tags the nodes to a custom
  `WrapProbe` model rather than extending the built-in `Script`.
- **V3 mode's model-importer offer:** opening a v3 project whose
  tree uses a node type absent from the `TreeNodesModel` presents a
  model-importer dialog — the reference offers to create the model
  manually (type + detected ports pre-filled) and points to the
  library's own model-XML writer as the proper generator. The
  importer-created model participates in the saved file like any
  other custom model (the O-12 output embeds it, `editable="true"`).
- **The O-10/O-13 refusals share one shape:** the reference validates
  instance ports against the node's model at load; a `Script` node
  in Groot2 v1.10.0 carries at most its `name` and its single
  declared port.

---

## Wall statement

This set contains text files only — no screenshots or images. No
capture-tooling or rig details appear in it; the record is the
input/output bytes, the MANIFEST hashes, and this findings record.
The reference's own settings-file option (the O-10 empty-port
emission) is described functionally; no configuration-file paths are
cited.
