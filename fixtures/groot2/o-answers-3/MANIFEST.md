# Manifest — Groot2 O-answer fixture set, third part (O-18 + duplicate-model)

Published: 2026-11-25. Continues the save-behavior answers of the
implementation engineering specification's §7 open questions (the first
set, O-1…O-9, is in [`../o-answers/`](../o-answers/), the second set,
O-10…O-16, in [`../o-answers-2/`](../o-answers-2/)), measured in a
2026-11-25 session against **Groot2 v1.10.0** (PRO tier, BT.CPP 4.10
target). Every file in this set is self-authored; the `out-*` files are
the reference application's own written output, kept byte-for-byte. All
bytes were re-derived at publication; re-derive any entry with
`sha256sum <path>` from this directory.

| path | bytes | sha256 |
|------|------:|--------|
| `in-o18-proj/proj.btproj` | 156 | `dfeafb16f0cc4613b907e1e5b854c47bdc2cee55bd9815e0341717c3befc6ee3` |
| `in-o18-proj/tree-a.xml` | 184 | `f3009289d86f3a3b65f9f4fb941d5fd1c6c9e2cd2549efa1240da5f5316d1b9d` |
| `in-o18-proj/sub/tree-b.xml` | 171 | `2a04b394166f9ab74680e3d50bf7f642106149310ba1234340d21e2248ea3746` |
| `out-o18-copy/proj.btproj` | 289 | `fc53554af3a7bea3ad1a2ac9ac9ae519182d469582352b222b949312764ccced` |
| `out-o18-only/proj.btproj` | 289 | `fc53554af3a7bea3ad1a2ac9ac9ae519182d469582352b222b949312764ccced` |
| `out-o18-open/proj.btproj` | 289 | `fc53554af3a7bea3ad1a2ac9ac9ae519182d469582352b222b949312764ccced` |
| `out-o18-mod/proj.btproj` | 289 | `fc53554af3a7bea3ad1a2ac9ac9ae519182d469582352b222b949312764ccced` |
| `o18-dupmodel/proj.btproj` | 186 | `55f46f9801eaed09a1d0133886f48f6daa1ffbc9819d5c61b913c55d9d03fa10` |
| `o18-dupmodel/tree-x.xml` | 339 | `c1af551b6e2372b56e912b4a7d988bd9282a87f664fc23e7e86d2bec91ebeb0c` |

## What each directory is

- `in-o18-proj/` — the O-18 input project, `project_name="O18"`, two
  includes deliberately in **flat-then-subdir** order
  (`tree-a.xml`, then `sub/tree-b.xml`) so flatten-vs-subdir placement
  is testable within one project. Corrected-tree form (no `Action` node
  with children — such a node refuses to load). 2-space indent,
  double-quoted attributes, LF, single trailing newline.
- `out-o18-copy/proj.btproj` — export #1, **all rows `n/a`** (nothing
  tickable: the ticked-save state is not settable by hand on an
  unmodified tree).
- `out-o18-only/proj.btproj` — export #2, same project re-opened fresh,
  rows again all `n/a`.
- `out-o18-open/proj.btproj` — export #3, both trees **opened in the
  canvas**, nothing modified; rows still all `n/a`.
- `out-o18-mod/proj.btproj` — export #4, one port of `tree-a`'s tree
  **modified and left dirty**; its row carried a ticked-by-default
  checkbox. The ticked tree was saved **in place at its source path**
  (see the findings record); the destination received the project file
  only. The written tree file was restored to its manifest bytes before
  re-verification, so this set carries the input as authored.
- **All four `out-o18-*/proj.btproj` files are byte-identical**
  (hash `fc53554a…`): the destination always receives exactly the
  project file, with includes rewritten destination-relative, subdirectory
  segment preserved, order ascending on the written path, the
  version stamp added, and an empty `<TreeNodesModel/>` appended.
- `o18-dupmodel/` — the duplicate-model experiment. `proj.btproj`
  declares the model `X` in its own `TreeNodesModel`; the included
  `tree-x.xml` declares the **byte-identical** entry in its own block
  (one `    <Action ID="X"/>` line in each file, same indentation).
  The reference loaded this silently — no dialog, no warning.
  `tree-x.xml` is the reference's own **post-save** bytes: the
  duplicate was the entry written exactly once to exactly one entry in the written model
  block (`grep -c 'Action ID="X"'` → 1). The project file was untouched
  by the save and is published as authored.

## What is deliberately not in this set

- No screenshots or image files; the record is the bytes above.
- No capture-tooling or rigging details; the set stands on the
  input/output bytes and `PROTOCOL-3.md`.
- The O-18 input hashes were re-verified after every export (the
  in-place save incident of export #4 and the three in-place export
  incidents of the `Browse`/`OK` geometry — see PROTOCOL-3.md — were
  each restored byte-exact against this manifest).
