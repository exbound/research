# Exbound — scientific software reconstruction

Independent, open, *compatible* reimplementations of proprietary scientific and
engineering software — applications that can open, edit, compute, fit, graph,
script, save, and execute the originals' important workflows without the
vendor's software installed.

Recovering data out of proprietary files is one subsystem of this, not the
goal.

## The principle

> Complete compatible functionality is the long-term objective; independently
> verifiable subsystems are how we get there.

The proprietary application is the **behavioral reference, not the architectural
blueprint.** Where a mature open engine already implements the underlying
science, we build on it and match the vendor's behavior rather than rebuild it.
See [`docs/capability-model.md`](docs/capability-model.md) for the C1–C9
capability model and the reference/implementation/verification coverage triplet
that keeps "a parser exists" from ever reading as "the target is done."

## How the work is organized

- [`docs/capability-model.md`](docs/capability-model.md) — the C1–C9 model,
  coverage triplet, and the definition of "done."
- [`docs/legal-basis.md`](docs/legal-basis.md) — why independent
  compatible reimplementation is defensible, and the boundaries it must hold.
- [`dossiers/TEMPLATE.md`](dossiers/TEMPLATE.md) — the per-target research
  dossier structure (version archaeology → binary inventory → functional
  inventory → behavioral spec → reconstruction architecture → verification),
  plus the capability matrix and reference-environment spec.
- `dossiers/<target>.md` — one evidence-backed dossier per reconstruction
  target (the campaign; see the target inventory).
- [`docs/reproducibility.md`](docs/reproducibility.md) and
  [`docs/landscape.md`](docs/landscape.md) — the format & provenance
  liberation subprogram: existing open readers to build on, and the
  undocumented-defaults reproducibility problem that feeds C2–C4.

A dossier is a research record. The engineering specification (what the
independent software must implement, and how compatibility is measured) and the
contributor setup guide are kept as separate documents, so a contributor need
not infer requirements from a research diary.

## Targets

The campaign is a set of target-level reconstruction programs, each analyzed
across the full C1–C9 model — not a list of file converters. The
[target inventory](dossiers/README.md) lists all twelve and tracks each
dossier's status.

The active pilot is **#4 OriginPro** ([`dossiers/origin.md`](dossiers/origin.md)),
whose clean-room `.opju`/`.opj` reader is the first implementation repository,
[`exbound/opj`](https://github.com/exbound/opj). The remaining targets await
research; a dossier is created only once a target has verified findings.

## Scope and boundaries

Independent reimplementation for **compatibility** is defensible — reimplementing
functionality, commands, and interfaces is supported by the interoperability and
API-reimplementation case law (see `docs/legal-basis.md`). The boundaries this
project holds:

- Reference software is run only under a **lawful license**, with provenance
  recorded.
- Any reimplementation derived from a proprietary original uses a **clean-room**
  wall.
- **No copying of expressive elements** — UI chrome, icons, artwork, code — and
  **no vendor trademarks**: implementations are "<vendor>-compatible," never the
  product's name.
- We do not defeat DRM/dongle/activation on current software to enable
  unlicensed use, and we do not distribute circumvention tools.

## Positioning

Reader libraries (Bio-Formats, xylib, RosettaSciIO, OpenChrom, GSAS-II),
registries ([Datatractor/MaRDA](https://datatractor.org)), and
preservation/emulation (Software Heritage, EaaSI) each solve part of the
problem. None reconstructs the *application* — the computation, document model,
workflows, and programmability. That is the gap this project takes.

## Contributing

Progress is published as small, independently verifiable units: one documented
operation implemented, a failing native-project fixture, a recovered instrument
message with evidence, a reference computation showing a numerical discrepancy,
a missing command with a reproducible input and expected result. See a target's
dossier for its open work packages.

## License

[CC-BY-4.0](LICENSE) for documentation and research; implementation
repositories carry their own code licenses.
