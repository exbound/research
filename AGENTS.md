# AGENTS — Exbound research program

Conventions for any agent or contributor working in this repository. This is the
**public** research program: the target inventory, technical dossiers,
verification method, and legal basis. Read the [README](README.md),
the [capability model](docs/capability-model.md), and
[CONTRIBUTING](CONTRIBUTING.md) first; this file states the working rules.

## What this repository is — and is not

- **Is:** evidence-backed research records. Per-target dossiers (version
  archaeology → inventory → behavioral spec → reconstruction architecture →
  verification), the C1–C9 capability model, the reproducibility and landscape
  subprogram, and the legal basis.
- **Is not:** implementation code (that lives in implementation repositories,
  generated from the organization template), and not a place for recovery
  *mechanics* — see the public/private wall below.

## Public/private wall — the rule that governs everything here

This repository is public. That bounds what may be written:

- **Document format and behavior, never how they were recovered.** Observed
  functional facts — structures, fields, defaults, operations, state transitions
  — belong here. The *mechanics* of recovery do not: no disassembly or
  decompiler output, no mention of the reverse-engineering tools used, no binary
  internals (addresses, symbol names, vtable layouts, raw byte/tag grammars
  beyond what a format description needs). Those stay in the private analysis
  tree.
- **Never name a private repository, link it, or describe its internal
  structure** in any file, issue, PR, or commit here. Describe the standard or
  behavior itself, not its private source. Before naming any repository in
  public output, confirm it is public; treat an inability to confirm as private.
- When in doubt, a fact that is visible in a vendor's own publicly published
  files is publishable; a fact that only exists because of private analysis
  mechanics is not — state the behavior, omit the mechanics.

## Evidence standard

- **Every claim carries the command, file, or source that produced it.** No bare
  assertions.
- **Mark each statement** ESTABLISHED (observed or cited) vs. HYPOTHESIS /
  NEEDS-OBSERVATION. Never silently promote one into the other; a correction
  that retracts an overclaim is normal and expected.
- Use the **coverage triplet** — reference *observed* / independently
  *implemented* / *verified against* the reference — recorded independently. A
  capability is only as complete as its lowest covered property.
- `full` / `partial` / `none` are meaningful only next to an **enumerated
  feature set**. Otherwise say "sketch," never a percentage.
- **Negative results are welcome** and held to the same standard: a format that
  cannot yet be decoded, with the specific blocker, is a valid contribution.
- A short record with verified facts and honest open questions beats a long one
  full of unsupported claims.

## Dossiers

- A target gets a dossier ([template](dossiers/TEMPLATE.md)) once it has verified
  findings — not before. A proposal before then is an issue (see CONTRIBUTING).
- A dossier is a research record. The engineering specification and the
  contributor setup guide are separate documents in the implementation repo, so
  a contributor need not infer requirements from a research diary.
- Keep claims in a dossier inside the evidence standard above; mechanics of
  recovery stay out per the public/private wall.

## Scope and boundaries

Independent reimplementation for **compatibility**, on **lawfully held** copies,
documented clean-room where a proprietary original is involved. No copying of
vendor code, UI artwork, help text, or trademarks; compatibility is described
nominatively ("reads [product] projects," "[vendor]-compatible"), never by
naming an implementation after the product. No defeating DRM, dongles, or
activation; no circumvention tools. Case law and limits:
[legal basis](docs/legal-basis.md). This is orientation, not legal advice.

## Commits

- [Conventional Commits](https://www.conventionalcommits.org):
  `type(scope): description` — imperative, lowercase, no trailing period,
  subject ≤ 72. One logical change per commit; reference issues in the
  body/footer.
- Describe **what changed**, not the process that produced it — no tool or model
  names, no "adversarial review," in commit messages or PR bodies. **No AI
  attribution of any kind.** Read the trailers before committing.
