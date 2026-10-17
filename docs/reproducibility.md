# Undocumented defaults as a reproducibility problem

## The gap

Open readers for scientific formats overwhelmingly recover the *data* and stop
there. What they drop is the *method*: the processing the vendor software
applied — often silently, by default — between the raw signal and the number a
researcher reports. Baseline correction, smoothing windows, peak-integration
bounds, weighting schemes, unit conventions, interpolation, and version-specific
algorithm changes are frequently applied without being recorded in any
human-readable place, and are discarded when the data is read by an open tool.

A paper may state that a fit was done "in <software> v<x>" without the
initialization, bounds, weighting, or convergence criteria. Two labs reading the
same file with different tools can then report different numbers, each correct
for its reading. This is a measurable reproducibility failure, and the
literature on it is thin.

## Why it matters more than the container

For most popular formats the container is already liberated (see
`landscape.md`). The method layer is not, and it is where reproducibility
actually breaks. Recovering it — and, crucially, *measuring* how faithfully a
reader reproduces the vendor's default behavior — is the contribution with the
most leverage.

## Proposed artifact: a decoding-task benchmark corpus

A public corpus of format-decoding tasks, each entry:

- a sample file (lawfully generated, provenance recorded),
- the ground-truth decoded output (from the vendor tool, or — best — a file the
  vendor library *generated* from known inputs),
- notes on structure and on any default processing the vendor applied,
- a difficulty label and the specific blocker where decoding is incomplete.

Negative entries count: a format that cannot yet be fully decoded is a valid,
labeled data point with the blocker recorded.

Existing readers and registries (Datatractor's `yard`/`beam`, Bio-Formats,
RosettaSciIO, xylib) can be scored against it. No such corpus exists today;
digital-preservation corpora target format *identification* and conformance, not
decoding correctness.

## Verification discipline

Every claim about a reader carries the command that produced it. Data
equivalence is checked against the oracle over many files, not asserted. The
method-layer check is explicit: does the reader reproduce the vendor's silent
defaults? A reader that matches raw values but misses the default baseline is a
documented finding, not a pass. Null results ("field X not recoverable") get the
same evidence as positive ones.
