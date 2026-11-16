# Manifest — Groot2 O-answer fixture set, second part (O-10…O-16)

Published: 2026-11-15. Continues the save-behavior answers of the
implementation engineering specification's §7 open questions (the first
set, O-1…O-9, is in [`../o-answers/`](../o-answers/)), measured in a
second 2026-11-15 session against **Groot2 v1.10.0** (PRO tier,
BT.CPP 4.10 target) on self-authored BT.CPP v4-dialect inputs (O-12 is
v3). Every file in this set is self-authored; the `out-*` files are the
reference application's own written output, kept byte-for-byte. All
bytes were re-derived at publication; re-derive any entry with
`sha256sum <path>` from this directory.

| path | bytes | sha256 |
|------|------:|--------|
| `in-o10-wrap/proj.btproj` | 119 | `ea9b714677c3802a02b9115b2ec73de79480b6f7a6774edf466750a8b9dc913d` |
| `in-o10-wrap/tree.xml` | 779 | `63a3fb28f91fe7af3807b2cf6443307fb8e12ff85cab7f3ff11c659e04a82cd6` |
| `in-o11-collide/a/tree.xml` | 151 | `a533107280d67af9d4c042ee18015187d10517f9f2389333cbcfc089005aeee9` |
| `in-o11-collide/b/tree.xml` | 194 | `7237d5549ca69c523e5ed5e42bfb8a81f297addfb62eb3caebda39a4b0b60711` |
| `in-o11-collide/proj.btproj` | 152 | `88ec4bab456112cebfa3179cebd4e6ce93ad8940106ed238c905ff0f20a4c737` |
| `in-o12-v3/proj.btproj` | 247 | `f5e5c4a830ce0e195312b8b10e2879ae5bf5c2e0e44906da8aeb71da830bb582` |
| `in-o12-v3/tree-crlf.xml` | 345 | `5a09c356a2af2b781ed0a0494f474a65be2d5a769511f633f760b239d4e92aea` |
| `in-o12-v3/tree.xml` | 331 | `1585e824577ce4604f46fe31ed8675827462154fc5cf0b64b042a076997a0866` |
| `in-o13-escape/attr-escapes.xml` | 224 | `a30cdbad9a23c5cbfa50709f9478dcae452a93528ba186d895abb917c5c903ca` |
| `in-o13-escape/content.xml` | 195 | `c12e9e95084eb18d3959009937338877d48df5d7479c1a26005e1f5258506867` |
| `in-o13-escape/mixed-forms.xml` | 303 | `6c1eb8dd1fae9780da458c5b118ce6ba66c5a1dc073c61c3fa3b7a80ff110822` |
| `in-o13-escape/two-trees-blank.xml` | 277 | `ee61b40e4715a2c44140f6e2da463c932b0157a2d55046194bb8d20e6c2cbcf2` |
| `in-o13-escape/two-trees-tab.xml` | 228 | `a4a4131dfa523a245d7750eaeecac5f92edbc592117dbfd93b501740449cffaa` |
| `in-o14-models/proj.btproj` | 323 | `8a79cd231e00baa4913382d21a394f0e405a330ba44ddb0652693c62d585b615` |
| `in-o14-models/tree.xml` | 211 | `920a1e965bb12437e7ccd5ee72792162cba52a10e9e04269ee2e8f04b886b710` |
| `in-o14-models/unused.xml` | 154 | `b2c2ba67a4b11e761ce4822ed42fb02e62d9496745305281d7114476c46de3bd` |
| `in-o16-stamps/ordered.xml` | 177 | `07f344d63659aaad3884d890d9b088e0538c63b47ee2163debaa895a0df98be4` |
| `in-o16-stamps/proj-411.btproj` | 141 | `b7437050d677c439ef4c8caff4b1b31fe7fba45241e17ac02adeff55d9dc4e9b` |
| `in-o16-stamps/proj-stamp.btproj` | 155 | `52a1c121b85852e0a58ddd5a26e139d94620dec68112f40eef3f059e55f57` |
| `in-o16-stamps/stamped.xml` | 173 | `8d718adac1d0e22a61468f114ee63c1143944ec3db3375b6f1c8967165d9f6e9` |
| `in-o16-stamps/t411.xml` | 150 | `96b01c65633a9645ab7e7aae9546c0dfa8f2d7b632763eafa17619f765a39b18` |
| `out-o10-corrected/proj.btproj` | 430 | `a88cb1dbc8c0af974c07725a39f798138f3c6f6a0eb48361bac9cd4599f4ce45` |
| `out-o10-corrected/tree.xml` | 791 | `a2d4208737c67dbea8f65fd89da530d2ff30e660a1e70caa9541260fb8a29326` |
| `out-o10/tree.xml` | 1679 | `a750cce407a88b3f5b248d31a6f72562b38402c18aab0e13300e970136461042` |
| `out-o11-corrected/a/tree.xml` | 151 | `a533107280d67af9d4c042ee18015187d10517f9f2389333cbcfc089005aeee9` |
| `out-o11-corrected/b/tree.xml` | 138 | `f175ba32443a784a9e9375f1abffc1e994a81066bc175f3992890f4e16d0ef49` |
| `out-o11-corrected/proj.btproj` | 152 | `88ec4bab456112cebfa3179cebd4e6ce93ad8940106ed238c905ff0f20a4c737` |
| `out-o11-export1-proj.evidence.xml` | 297 | `3ef5f16310621eb9a9cb5501719c555a7e6a8fd5c86e244d32aec07780002754` |
| `out-o11/export-inplace-saved-a-tree.evidence.xml` | 255 | `f554dbdfe964d74d3a97300c0cae6c0a0ca366d5f803fff412c0372261776a87` |
| `out-o11/proj.btproj` | 297 | `3ef5f16310621eb9a9cb5501719c555a7e6a8fd5c86e244d32aec07780002754` |
| `out-o12-export/export1-standalone-crlf.evidence.xml` | 440 | `ace4d1ed7b3d3cb0475e53ebf40687050cee9659b44ba82dc86b67581f57cb7e` |
| `out-o12-export/proj.btproj` | 440 | `ace4d1ed7b3d3cb0475e53ebf40687050cee9659b44ba82dc86b67581f57cb7e` |
| `out-o12/tree-crlf.xml` | 647 | `b2e766319afd370e1af6d0e60109471717986b097b55fa4d8677702c90139f4b` |
| `out-o12/tree.xml` | 647 | `b2e766319afd370e1af6d0e60109471717986b097b55fa4d8677702c90139f4b` |
| `out-o13-corrected/attr-escapes.xml` | 410 | `c6b2fd9dae47a91d092d02f1442d754beadfe7d3c7c82086efdb95529b10e715` |
| `out-o13-corrected/mixed-forms.xml` | 273 | `4036450d7d68d5cf6798ba6ac6fbb46f7995d4b9906f67273fd1b5cb0edce18e` |
| `out-o13-corrected/two-trees-blank.xml` | 221 | `58b1669cdae533925dc3d080a4e0a5d08dd42068d8e158a7c331cbd6d47f3952` |
| `out-o13/attr-escapes.xml` | 569 | `8b4d0729a8131729fcd351117081de9d7ab4be94aba2ea6ddf48d9a0563f31c6` |
| `out-o13/content.xml` | 284 | `c36cd2a4ba5d8ec70e1086522afc00309e5bb1d6069dc2d8a6bd4cdbe21e25f4` |
| `out-o13/mixed-forms.xml` | 346 | `6fbabb8624fcadb4c2a60ddc23c41262045b82611872b794cbc00a4978288cb3` |
| `out-o13/two-trees-blank.xml` | 323 | `9e7ef7aac604f52b27acceb0f0292e635f0f13982cb0d809c361efd5a87b21fe` |
| `out-o13/two-trees-tab.xml` | 336 | `96536c65092e6d979b6ab74cf29be252f25f2f957ace91d97cc145e983126f24` |
| `out-o14-export/proj.btproj` | 448 | `37a41f0bf01f39fdd8c805680f1f5489fe6966783461ebc47d05cbbd71d01926` |
| `out-o14/tree.xml` | 432 | `c7ab6811b338585ca92feba24d07682e29ceb2da4967da54ec7c0dd42e1a3027` |
| `out-o14/unused.xml` | 257 | `63843ae3bdf20edb4a7fa21087a31c751f8b6cfcdb6ab96587f7ad0f89feca46` |
| `out-o16a/proj.btproj` | 221 | `51f9c4146f43bb44e801b779b7468d83ec6c15b962f547f3e2451d12a29828ea` |
| `out-o16a/t411.xml` | 253 | `f8a5e2156e149d9e28740cdb2f3b1813649c35bbd56492210ef4236ac2ccef9e` |
| `out-o16b-export/proj.btproj` | 282 | `00c523ed6b043bdfc4ac62cf3624d420ec24053c7728cbc1ea9bad0070ad6210` |
| `out-o16b/ordered.xml` | 256 | `078cd5fcf6c024840db53c0b3ed039ce78e4ab99314ef123c408521ff6e9caa4` |
| `out-o16b/stamped.xml` | 256 | `7e7235d9cddd531483e53314084ea7541280d38bd7395579b328ae9cfc292b34` |

## What each directory is

- `in-o10-wrap/` — as-authored O-10 input (attribute wrapping). Its
  `Script` nodes carry ports the reference's built-in `Script` model
  does not declare, so it **cannot load** as authored — published
  because the rejection is itself a recorded behavior (see the findings
  record). The measurements ran on `out-o10-corrected/`.
- `out-o10-corrected/` — the corrected copy the O-10 measurement ran
  on: the four nodes re-tagged to a custom `WrapProbe` Action model
  (attribute counts and value lengths identical to the as-authored
  `Script` nodes), the project declaring the model with its five ports.
  The wrapping question is attribute-count/length-driven, so the node
  name swap preserves what O-10 measures.
- `out-o10/` — the reference's per-file save of the corrected tree:
  the O-10 bytes under analysis.
- `in-o11-collide/` — as-authored O-11 input (two includes sharing a
  basename, `a/tree.xml` and `b/tree.xml`). `b/tree.xml` carries an
  `Action` node with a child, which the reference refuses to load;
  the measurements ran on `out-o11-corrected/`.
- `out-o11-corrected/` — the corrected project the O-11 measurement
  ran on (the offending wrapper collapsed to a self-closing tag;
  `a/tree.xml` and `proj.btproj` byte-identical to the as-authored
  inputs).
- `out-o11/` — the reference's project-export output into a fresh
  directory: **only `proj.btproj` was written**. No tree file appeared
  in the destination or in pre-created subdirectories.
- `out-o11/export-inplace-saved-a-tree.evidence.xml` — the reference's
  own in-place save of the modified `TreeA` during export: it landed
  **at the tree's source path**, not in the export destination.
- `out-o11-export1-proj.evidence.xml` — a byte-identical copy of
  `out-o11/proj.btproj` kept as capture evidence (same hash
  `3ef5f163…`).
- `in-o12-v3/` — as-authored O-12 input (a v3 project, `tree.xml`, and
  `tree-crlf.xml` — the same content with CRLF line endings).
- `out-o12/` — the reference's V3-mode per-file saves: `tree.xml` and
  `tree-crlf.xml` are **byte-identical** (the CRLF input normalizes to
  LF), both `b2e76631…`.
- `out-o12-export/` — the reference's project export in V3 mode
  (`proj.btproj` only; no tree copies) plus
  `export1-standalone-crlf.evidence.xml`, a byte-identical evidence
  copy.
- `in-o13-escape/` — five as-authored O-13 standalone tree files, each
  varying one property (attribute escaping, node text content, blank
  lines between trees, tab indentation, open/close element forms).
- `out-o13-corrected/` — corrected copies for the three O-13 inputs
  the reference refused as authored (undeclared port on `Script`;
  `Action` nodes with children in two files); attribute values and
  layout variants preserved.
- `out-o13/` — the reference's save-as output for all five O-13 files
  (the two uncorrected inputs loaded and were saved as authored).
- `in-o14-models/` — as-authored O-14 input (priority): a project
  declaring two custom models whose trees use one respectively none of
  them, with the model block authored in reverse-alphabetical order
  (see the O-14 caveat in the findings record).
- `out-o14/` — the reference's per-file saves of both trees: the
  used-model selection rule under analysis.
- `out-o14-export/` — the reference's project export (`proj.btproj`
  only; both includes repointed, both models embedded).
- `in-o16-stamps/` — the O-16 stamp-case inputs: `proj-411.btproj`
  with a **newer** stamp (`4.11` > target `4.10`), `proj-stamp.btproj`
  including a bare tree carrying an **older** stamp (`4.9`) plus
  `ordered.xml` with non-canonical root-attribute order, and the
  project-level `4.11` tree `t411.xml`.
- `out-o16a/` — the reference's per-file save (`t411.xml`, stamped
  `4.11`) and project export (`proj.btproj`, `4.11` preserved) for the
  newer-stamp case.
- `out-o16b/` — the reference's per-file saves for the older-stamp
  case: `stamped.xml` (stamp rewritten `4.9` → `4.10`) and
  `ordered.xml` (root normalized; foreign `project_name` dropped).
- `out-o16b-export/` — the reference's project export for the
  older-stamp project (`proj.btproj` only, includes sorted and
  repointed).

## What is deliberately not in this set

- No screenshots or image files; the record is the bytes above.
- No capture-tooling or rigging details; the set stands on the
  input/output bytes and `PROTOCOL-2.md`.
- The as-authored O-12 `tree-crlf.xml` line endings (CRLF) and the
  as-authored O-13 tab indentation are deliberate variants and are
  preserved byte-exact; every other input is LF with a single trailing
  newline.
