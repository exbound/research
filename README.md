# Scientific data & format liberation

Open readers, recovered provenance, and reproducibility for scientific and
engineering data held in proprietary or abandoned file formats.

## The problem

Research data is routinely trapped in vendor file formats and instrument-bound
software. When the software is discontinued, licensed only at high cost, or
simply undocumented, the data — and the ability to reproduce the result that
produced it — becomes inaccessible. This directly blocks the Accessible,
Interoperable, and Reusable pillars of the [FAIR principles](https://www.go-fair.org/fair-principles/),
and it is a recognized driver of the reproducibility problem: the Datatractor
work ([Evans et al., 2024](https://arxiv.org/abs/2410.18839)) frames the same
issue — proprietary formats that require reverse engineering and that shift
between software versions.

## Two contributions

Most popular scientific formats already have mature open readers (see
[`docs/landscape.md`](docs/landscape.md)). Re-implementing those adds little.
The useful work is narrower:

1. **A map of what is and isn't liberated** — per-format, which open tool reads
   it, how completely, and what it drops. Kept current, with evidence.
2. **The provenance / reproducibility layer** — most open readers recover the
   *data* but discard the *method*: the processing defaults, calibration, and
   analysis steps the vendor software applied silently. Recovering and measuring
   that is the higher-value, under-addressed contribution. See
   [`docs/reproducibility.md`](docs/reproducibility.md).

The concrete artifact we aim to own is a **benchmark corpus of
format-decoding tasks** — paired (sample file + ground-truth decoded output +
notes) with difficulty labels — which existing readers and registries can be
scored against. No such corpus exists today.

## Positioning

This complements, rather than duplicates, the existing ecosystem:

- Reader libraries — [Bio-Formats](https://www.openmicroscopy.org/bio-formats/),
  [xylib](https://github.com/wojdyr/xylib),
  [HyperSpy/RosettaSciIO](https://github.com/hyperspy/rosettasciio),
  [OpenChrom](https://www.openchrom.net/), GSAS-II, SpectroChemPy.
- Registries/runners — [Datatractor/MaRDA](https://datatractor.org)
  (`yard` registry + `beam` runner), which has no benchmark corpus.
- Preservation/emulation — Software Heritage, EaaSI, the Software Preservation
  Network.

We sit at the evaluation-and-provenance layer the ecosystem lacks: measuring and
preserving the *correctness* of liberation, and exposing the undocumented
behaviors that break reproducibility.

## Scope

Interoperability and preservation: recovering lawfully held data and the
semantics that produced it. Formats and protocols are functional facts, and
reading one's own data is broadly protected activity — see
[`docs/legal-basis.md`](docs/legal-basis.md). This project does not distribute
circumvention tools or defeat access controls on current commercial software.

## Contributing

Format status corrections (with a reproducible check), new decoding-task corpus
entries, and provenance findings are welcome. See the docs for the format of
each.

## License

[CC-BY-4.0](LICENSE) — reuse with attribution.
