# Contributing

Exbound is reconstructing independent, compatible open implementations of
proprietary scientific and engineering software (see the [README](README.md) and
the [capability model](docs/capability-model.md)). This is an early-stage
research repository. This page is the entry point; fuller agent-onboarding and
per-target implementation repos come later.

## What's useful right now

Small, independently verifiable units — not large speculative write-ups:

- A **verified format/record finding** (with the bytes/command that show it).
- A **failing native-file parse fixture** for a target.
- A **recovered behavior** of a reference product, with the evidence.
- A **reference computation** showing a numerical discrepancy vs. an
  independent implementation.
- A **documented command/operation** with a reproducible input and the exact
  expected result.
- **Corrections** to the landscape or a dossier — each with a reproducible
  check, not an assertion.

A short finding with six verified facts and ten explicitly open questions beats
a long document full of unsupported claims.

## How to report evidence

- **Every claim carries the command, file, or source that produced it.** No bare
  assertions.
- **Mark each statement** ESTABLISHED (cited or observed) vs.
  HYPOTHESIS / NEEDS-OBSERVATION. Do not promote one into the other.
- Use the **coverage triplet** (reference observed / implemented / verified) —
  see the capability model. Don't report implementation as if it were verified.
- **Negative results are welcome** and get the same evidence standard: a format
  that can't yet be decoded, with the specific blocker, is a valid contribution.
- `full` / `partial` / `none` are only meaningful next to an enumerated feature
  inventory — otherwise say "sketch," not a percentage.

## Proposing a target

Open an issue with: the product and exact versions, why users are locked in
(hardware, proprietary data, required workflow, cost/monopoly), what
verification oracle exists (reference install, sample files, automation API),
and what's obtainable without a license. A target gets a dossier
([template](dossiers/TEMPLATE.md)) once it has verified findings — not before.

## Scope and boundaries

Independent reimplementation for **compatibility**, on **lawfully held** copies,
documented clean-room where a proprietary original is involved. No copying of
vendor code, UI artwork, or trademarks; no circumventing DRM/dongles/activation
to enable unlicensed use. Details and case law: [legal basis](docs/legal-basis.md).
