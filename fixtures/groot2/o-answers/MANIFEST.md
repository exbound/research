# Manifest — Groot2 O-answer fixture set (save behavior, O-1…O-9)

Published: 2026-11-15. Answers the open questions (§7) of the
implementation engineering specification, from a reference-observation
session against **Groot2 v1.10.0** (PRO tier, BT.CPP 4.10 target) on
self-authored BT.CPP v4-dialect inputs. Every file in this set is
self-authored; the `out-*` files and the `-rt.xml` siblings are the
reference application's own written output, kept byte-for-byte.
Re-derive any entry with `sha256sum <path>` from this directory.

| path | bytes | sha256 |
|------|------:|--------|
| `in-o1-proj-corrected/proj.btproj` | 249 | `d30066b95f2b090da80f8941553d217bbc83aeba1363e5ffe2000ae4106d0c5b` |
| `in-o1-proj-corrected/tree.xml` | 309 | `c800d2b68ef9720f66d6230378444c959be330eb6094168caf01c524571cb6d2` |
| `in-o1-proj/proj.btproj` | 249 | `d30066b95f2b090da80f8941553d217bbc83aeba1363e5ffe2000ae4106d0c5b` |
| `in-o1-proj/tree.xml` | 266 | `31ce572eff48443be6843eea06ba6e6a9ed6215da3cc3465ee6c6f7630df93af` |
| `in-o2-diff-corrected/proj.btproj` | 270 | `6880dc3969391c89a6cc599ba62065235eb019b346eefa15910984b9d80b3846` |
| `in-o2-diff-corrected/tree.xml` | 206 | `e51b8e8d655f8b82500db271dc0d685c68b695300eb1bca633118a65f912f1ce` |
| `in-o2-diff/proj.btproj` | 270 | `6880dc3969391c89a6cc599ba62065235eb019b346eefa15910984b9d80b3846` |
| `in-o2-diff/tree.xml` | 266 | `31ce572eff48443be6843eea06ba6e6a9ed6215da3cc3465ee6c6f7630df93af` |
| `in-o2-same-corrected/proj.btproj` | 271 | `63f7dd0cdafb5ff529d687b42518768a072e6a842b91102e0e230408eda64557` |
| `in-o2-same-corrected/tree.xml` | 206 | `e51b8e8d655f8b82500db271dc0d685c68b695300eb1bca633118a65f912f1ce` |
| `in-o2-same/proj.btproj` | 271 | `63f7dd0cdafb5ff529d687b42518768a072e6a842b91102e0e230408eda64557` |
| `in-o2-same/tree.xml` | 266 | `31ce572eff48443be6843eea06ba6e6a9ed6215da3cc3465ee6c6f7630df93af` |
| `in-o4-paths/abs.xml` | 177 | `9fcf75015145f18e8a30064835d7b3e4703c4a16d191b258ca1b9d5a6238444f` |
| `in-o4-paths/deep/nest/tree2.xml` | 176 | `5658488876b0ab1f4a6d54e3e4530f1eed32fab3f91d8fdc857ba696952e60f4` |
| `in-o4-paths/proj.btproj` | 242 | `a410c35c09e794e659952b1b017b78b81fa6ebfe23c5d2ecbf106df2876c3a60` |
| `in-o4-paths/sub/tree.xml` | 173 | `64e3c096866ca46ea28b4364c2077fce32a47a056c2e7a85c98993114dc1b774` |
| `in-o5-single.xml` | 183 | `e3a012abdb6b58274969eb5e76dfffba30c747dc9abbc4fbecc1c6b3db591da6` |
| `in-o6-model/proj.btproj` | 403 | `660835e08311c72c8fce08f8dfc29c3f312767ae2fc482b43de6ac6ffcb72ebe` |
| `outside.xml` | 173 | `e826c43c602bb3630481a3b721a7ad6aa709871f35b1f4bfebc23df5c1162613` |
| `in-o6-model/tree.xml` | 223 | `61e061741186ac6b10da3aa9e651404e78ec5ca953e577422a8c6b9a21555746` |
| `in-o9-awkward/altquote-rt.xml` | 273 | `129a8f83e9ec93b9e83727dde5fd1cb4305476da190783a17476588baf821fe4` |
| `in-o9-awkward/altquote.xml` | 170 | `7fd503f398c9641112daa3c1e0f82b7e7c71f407d890926c796b36e4fc93dc96` |
| `in-o9-awkward/charrefs-rt.xml` | 281 | `9f3c62c90de6622d758173b2aa01ad49f1ec1ed92c0b3be9f9b29c4df7b804f9` |
| `in-o9-awkward/charrefs.xml` | 182 | `5020b9f7c8e93e5b44f808e7c0ab9b3b5c0f71e313f61beaef0de669399b8b85` |
| `in-o9-awkward/comments-rt.xml` | 273 | `129a8f83e9ec93b9e83727dde5fd1cb4305476da190783a17476588baf821fe4` |
| `in-o9-awkward/comments.xml` | 389 | `b6da0f10b1544686e4a9d30544957a56fade4aa1d388171623a83b1e2f229595` |
| `in-o9-awkward/crlf-rt.xml` | 273 | `129a8f83e9ec93b9e83727dde5fd1cb4305476da190783a17476588baf821fe4` |
| `in-o9-awkward/crlf.xml` | 178 | `3974a9b30a28e1af4562eb7148ed8ade8c560e4d9871bb14ae1a2d5ab906391b` |
| `in-o9-awkward/nodecl-rt.xml` | 273 | `129a8f83e9ec93b9e83727dde5fd1cb4305476da190783a17476588baf821fe4` |
| `in-o9-awkward/nodecl.xml` | 131 | `ccc4e514490b298d27f5db3de6953f40534ab91bcfc8cd244c7b49ec9fe97765` |
| `in-o9-awkward/notrail-rt.xml` | 273 | `129a8f83e9ec93b9e83727dde5fd1cb4305476da190783a17476588baf821fe4` |
| `in-o9-awkward/notrail.xml` | 169 | `b1183644feda515358233d915353880d6b06fb7abbc16453b32a4c3c80489dc6` |
| `out-o2-diff/proj.btproj` | 347 | `f63f6f919e5daf7a1a4e47e2f1ff079bdefd21a3974f67f7eaef97bdb2360513` |
| `out-o2-diff/tree.xml` | 309 | `c800d2b68ef9720f66d6230378444c959be330eb6094168caf01c524571cb6d2` |
| `out-o2-same/proj.btproj` | 347 | `29234f689893dd54d9ef2b37dc457c0161bf49b03a76322d5238d348b6321e15` |
| `out-o2-same/tree.xml` | 309 | `c800d2b68ef9720f66d6230378444c959be330eb6094168caf01c524571cb6d2` |
| `out-o4/abs.xml` | 280 | `8698f4d71d020652b0a0eec2c84c333a16580c575b5d25e45b1c7462cdf55fd4` |
| `out-o4/outside.xml` | 276 | `521dd388a85e710959e950ac1740ff83b14a64953768c3efc9c6b86ecff5c313` |
| `out-o4/proj.btproj` | 315 | `cb25fb1419a3fd2b52e135a3cd86c0126ad2f22126e7530eb6308d671bc5743a` |
| `out-o4/tree2.xml` | 279 | `e8d30c269a961122b6d877dfa590edb31db5c005641246720add2fb775a092f2` |
| `out-o4/tree.xml` | 276 | `3ed575d4a8819b906c5470805b5dfa6d68261de7c745e6d7e9a420b367d0bc76` |
| `out-o5/in-o5-single.xml` | 286 | `413174e6a602c1a294f2612966e522ef01c417769c0f732e45a212860c88b5ca` |
| `out-o6/proj.btproj` | 438 | `be9523d942f730f9ac9229f652331cf77976060ec3aa32b318a765d8f119a3dc` |
| `out-o6/tree.xml` | 548 | `f1a2822e9e492c7b614bf45b2033649bc5b6b9a30a0e49f4791e78b1b45bacf9` |

## What each directory is

- `in-o1-proj/` — as-authored project: its tree contains an Action
  node **carrying a child**, which the reference's loader rejects
  ("An Action node must not have any child"). Published because the
  rejection itself is a recorded behavior (see
  [`../../dossiers/groot2-o-answers.md`](../../dossiers/groot2-o-answers.md)).
- `in-o1-proj-corrected/` — the same project with the offending node
  collapsed to a self-closing tag so the reference will load it
  (`proj.btproj` byte-identical to `in-o1-proj/`). The `tree.xml`
  here is the **reference's own rewritten bytes** after an unmodified
  in-place save (O-8), kept as evidence of the rewrite.
- `in-o2-same/`, `in-o2-diff/` — as-authored projects (both carry the
  same child-carrying tree) and
- `in-o2-same-corrected/`, `in-o2-diff-corrected/` — the corrected
  copies the measurements ran on.
- `out-o2-same/`, `out-o2-diff/` — the reference's project-Export
  output (the reference's only project-file write) for each.
- `in-o4-paths/` — a project whose four includes exercise nested,
  parent-relative, deep-nested and **absolute** path forms. The
  absolute form is a neutral path (`/abs/in-o4-paths/abs.xml`); the
  capture ran with a machine-specific absolute path, and the
  reference output flattens either to the bare basename `abs.xml`.
- `in-o5-single.xml` — a standalone tree file, no project.
- `outside.xml` — the O-4 parent-relative include target (the
  `../outside.xml` entry in `in-o4-paths/proj.btproj`), published
  2026-11-15 so the O-4 export can be replayed: it sits at the
  include-path destination, the `in-o4-paths/` parent directory in
  the capture layout, so `in-o4-paths/proj.btproj` resolves it
  without modification. The file's bytes match the MANIFEST hash
  that was recorded when the set was first captured (only the hash
  had been published before).
- `in-o6-model/` — a project whose node models isolate the
  `editable` attribute and the `<description>` element across three
  entries.
- `in-o9-awkward/` — six awkward-but-legal XML variants, each beside
  its `-rt.xml` sibling: the reference's own save of that file.
- `out-o4/` — the reference's Export of `in-o4-paths/` to a fresh
  directory (project plus four flattened tree files).
- `out-o5/` — the reference's save-as of `in-o5-single.xml` to a new
  path.
- `out-o6/` — the reference's Export of `in-o6-model/`.
