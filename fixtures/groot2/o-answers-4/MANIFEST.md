# Manifest — Groot2 O-answer fixture set, fourth part (O-18b: the include-form sweep)

Published: 2026-11-28. Continues the save-behavior answers of the
implementation engineering specification's §7 open questions (the first
set, O-1…O-9, is in [`../o-answers/`](../o-answers/), the second set,
O-10…O-16, in [`../o-answers-2/`](../o-answers-2/), and the third set,
O-18 and the duplicate-model question, in
[`../o-answers-3/`](../o-answers-3/)), measured in a 2026-11-28 session
against **Groot2 v1.10.0** (PRO tier, BT.CPP 4.10 target). Every file in
this set is self-authored; the `out-*` files are the reference
application's own written output, kept byte-for-byte. All bytes were
re-derived at publication; re-derive any entry with `sha256sum <path>`
from this directory.

## Files (byte count, sha256)

### Set root

| path | bytes | sha256 |
|------|------:|--------|
| `outside.xml` | 173 | `e826c43c602bb3630481a3b721a7ad6aa709871f35b1f4bfebc23df5c1162613` |

The target of every variant's `../outside.xml` include (see PROTOCOL-4.md
for the resolution layout).

### in-original/

| path | bytes | sha256 |
|------|------:|--------|
| `in-original/proj.btproj` | 268 | `df0aa92651ec541281ae42441b94fcb980c42f1b13ee3d8d41294ab45588be47` |
| `in-original/abs.xml` | 177 | `9fcf75015145f18e8a30064835d7b3e4703c4a16d191b258ca1b9d5a6238444f` |
| `in-original/outside.xml` | 173 | `e826c43c602bb3630481a3b721a7ad6aa709871f35b1f4bfebc23df5c1162613` |
| `in-original/sub/tree.xml` | 173 | `64e3c096866ca46ea28b4364c2077fce32a47a056c2e7a85c98993114dc1b774` |
| `in-original/deep/nest/tree2.xml` | 176 | `5658488876b0ab1f4a6d54e3e4530f1eed32fab3f91d8fdc857ba696952e60f4` |

The O-4 project from the first set, published as-is: all four include
forms (subdir-relative, parent-relative, deep-subdir-relative, absolute).
`proj.btproj` is byte-identical to the first set's `in-o4-paths/proj.btproj`.

### in-no-abs/

| path | bytes | sha256 |
|------|------:|--------|
| `in-no-abs/proj.btproj` | 197 | `fc71799713c4d78eefd1744da90df58f70764d8c880815190e3dad8b4b70ec70` |
| `in-no-abs/abs.xml` | 177 | `9fcf75015145f18e8a30064835d7b3e4703c4a16d191b258ca1b9d5a6238444f` |
| `in-no-abs/outside.xml` | 173 | `e826c43c602bb3630481a3b721a7ad6aa709871f35b1f4bfebc23df5c1162613` |
| `in-no-abs/sub/tree.xml` | 173 | `64e3c096866ca46ea28b4364c2077fce32a47a056c2e7a85c98993114dc1b774` |
| `in-no-abs/deep/nest/tree2.xml` | 176 | `5658488876b0ab1f4a6d54e3e4530f1eed32fab3f91d8fdc857ba696952e60f4` |

`proj.btproj` = `in-original/proj.btproj` minus the absolute-include line;
`abs.xml` is present on disk but **unreferenced**.

### in-no-abs-no-parent/

| path | bytes | sha256 |
|------|------:|--------|
| `in-no-abs-no-parent/proj.btproj` | 162 | `d90b22bb8c9c72e886910c5206010d8569202126b0b1a0862f797c19c4ff1633` |
| `in-no-abs-no-parent/abs.xml` | 177 | `9fcf75015145f18e8a30064835d7b3e4703c4a16d191b258ca1b9d5a6238444f` |
| `in-no-abs-no-parent/outside.xml` | 173 | `e826c43c602bb3630481a3b721a7ad6aa709871f35b1f4bfebc23df5c1162613` |
| `in-no-abs-no-parent/sub/tree.xml` | 173 | `64e3c096866ca46ea28b4364c2077fce32a47a056c2e7a85c98993114dc1b774` |
| `in-no-abs-no-parent/deep/nest/tree2.xml` | 176 | `5658488876b0ab1f4a6d54e3e4530f1eed32fab3f91d8fdc857ba696952e60f4` |

`proj.btproj` = `in-original/proj.btproj` minus both the absolute- and
the parent-relative-include lines; `abs.xml` and `outside.xml` are
present on disk but **unreferenced**.

### out-*/ (the reference's own written project files)

| path | bytes | sha256 |
|------|------:|--------|
| `out-o18b-control/proj.btproj` | 321 | `4321fbe077c0afe48c6ed27f381df51b693f92f4b4b6018cc2d7fb571ae72c3f` |
| `out-o18b-original/proj.btproj` | 391 | `9ed5b92d545d3fd05716a6fb41230d4744d7f520be81cbf5370f4175fe365741` |
| `out-o18b-no-abs/proj.btproj` | 328 | `d45843c7dbe967e8087760f0b8f0b7da76aff6a04bc95be27cf33a6a1957b8c6` |
| `out-o18b-no-abs-no-parent/proj.btproj` | 311 | `58b02d6f9c6ab7dc6c036775faae7e81ed7b5bc1c5eac969905e233f99ac6d27` |

Each `out-*/` directory held **exactly** that one file after its export —
no tree file of any kind was written into any destination. The written
`proj.btproj` files are distinct (each carries its own
destination-relative repointing), and each is the reference's canonical
rewrite of its input: 4-space indent, `BTCPP_version="4.10"` stamp added,
includes re-expressed destination-relative (subdirectory segments
preserved, the absolute include re-expressed as a relative path), and an
empty `<TreeNodesModel/>` appended. The `path=` values inside the written
files are the reference's own output bytes, relative to the capture
layout at the time of the export (PROTOCOL-4.md, "Published bytes").

## What each directory is

- `in-original/` — the first set's O-4 project, **all four include
  forms**: `sub/tree.xml` (subdir-relative), `../outside.xml`
  (parent-relative), `deep/nest/tree2.xml` (deep subdir-relative), and
  the **absolute** include (kept at its original path string, which
  resolves against the O-4 source location still on disk at capture
  time). This is the control for "does the original O-4 project,
  unmodified, reproduce the first session's copy-and-flatten?"
- `in-no-abs/` — `in-original/` with the **absolute** include removed
  from `proj.btproj`; the other three forms unchanged. `abs.xml` stays
  on disk, unreferenced.
- `in-no-abs-no-parent/` — `in-original/` with **both** the absolute
  and the parent-relative includes removed from `proj.btproj`; the two
  subdir-relative includes remain. `abs.xml` and `outside.xml` stay on
  disk, unreferenced.
- `out-o18b-control/` — the reference's written output of a **control**
  export: the third set's `in-o18-proj/` project (two unmodified
  includes, the exact input of the O-18 session) exported under the
  same conditions as the variants (unmodified trees, default dialog
  state, fresh destination). It reproduces the third set's
  project-file-only result, confirming the capture state.
- `out-o18b-{original,no-abs,no-abs-no-parent}/` — the reference's
  written output of each variant, under identical conditions.

## What is deliberately not in this set

- No screenshots or image files; the record is the bytes above.
- No capture-tooling or rigging details; the set stands on the
  input/output bytes and `PROTOCOL-4.md`.
- The `out-*/proj.btproj` files carry the reference's own written
  `path=` values, which are relative to the **capture layout**, not to
  this published layout (PROTOCOL-4.md, "Published bytes"). They are
  the reference's bytes, kept verbatim, for the same reason the
  third set's `out-*/proj.btproj` files were.
