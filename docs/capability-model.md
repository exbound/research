# Reconstruction capability model

The project's objective is **independent, compatible reimplementation** of
proprietary scientific and engineering software — software that can open, edit,
compute, fit, graph, script, save, and execute the original's important
workflows without the vendor's application installed. Reading the vendor's files
is one subsystem of that, not the boundary.

Guiding principle:

> Complete compatible functionality is the long-term objective; independently
> verifiable subsystems are how we get there.

The proprietary application is the **behavioral reference, not the architectural
blueprint.** Where a mature open engine (SciPy, GSAS-II, OpenChrom, OpenMS,
LabPlot/SciDAVis, an instrument library) already implements the underlying
science, we build on it and treat the vendor app as the behavior to match.

## Capability dimensions (C1–C9)

Every target is analyzed across the same model. Not every target needs every
dimension (a numerical package may have no C7; an instrument driver may need no
C8) — but the model always covers the whole original product, not just its
file format.

| ID | Capability | Scope |
|----|------------|-------|
| C1 | Acquisition & execution | Install and operate the reference software; record exact versions and dependencies |
| C2 | Data compatibility | Read source data, files, metadata, proprietary containers |
| C3 | Document compatibility | Reconstruct, edit, create, and save native projects/documents |
| C4 | Computation | Reimplement algorithms, numerical operations, and defaults |
| C5 | Workflow semantics | Reproduce complete user operations and state transitions |
| C6 | Programmability | Scripts, APIs, macros, automation, extensions |
| C7 | Instrument integration | Communicate with hardware; reconstruct acquisition workflows |
| C8 | User-facing application | An independent interface for real work |
| C9 | Replacement coverage | Measure and close outstanding functional differences |

## Coverage triplet

For each capability of each target, three properties are recorded
**independently** — this is the core guard against the "lots of code, claimed
80% complete, never inventoried 80% of the product" failure:

- **Reference coverage** — has the original's behavior actually been *observed*
  (in a reference environment), with recorded evidence?
- **Implementation coverage** — has that behavior been *independently
  implemented*?
- **Verification coverage** — has the implementation been *tested against* the
  reference behavior, with recorded results?

A capability is only as complete as its *lowest* covered property. High
implementation coverage with low reference coverage means we built something we
never confirmed the original does.

## What "done" means — and does not

A target is **never** complete because a parser, reader, or exporter exists.
Completion is defined per capability by an **enumerated feature set plus
evidence**, not by the absence of known bugs. "Full support" for a capability
requires the feature list and the passing reference tests, stated with the exact
reference-software versions they were measured against.

## Relationship to the earlier recovery levels

The earlier L1 (data) / L2 (structure) / L3 (method) / L4 (behavior) ladder was
scoped to file reading only. It is retained as a finer breakdown *inside* C2–C5,
not as the ceiling: L1–L2 ≈ C2, L3 ≈ C4 provenance, L4 ≈ C5 — with C3, C6, C7,
C8, C9 added above them.

## Subprograms

- **Format & provenance liberation** (reading native files, recovering the
  processing/operation tree) is a named subprogram feeding C2–C4. See
  `reproducibility.md` and `landscape.md`.
- **Reference laboratories** (reproducible versioned installs + an
  observe/vary-input/record/diff harness) feed C1 and the reference coverage of
  every other dimension. This is the technical centerpiece; see the dossier
  template's reference-environment section.
