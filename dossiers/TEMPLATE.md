# Reconstruction dossier — <TARGET NAME>

> One dossier per reconstruction target. It answers *what the original product
> is and does, what we have established vs. not, and how we will reconstruct and
> verify it* — before serious implementation. Keep ESTABLISHED facts (cited or
> observed) strictly separate from HYPOTHESIS / NEEDS-OBSERVATION. A dossier is a
> research record, not an engineering spec and not a contributor guide (those
> are separate documents).

- **Target:** <product / family>
- **Status:** <research | spec | implementation> · **Last updated:** <date>
- **Maintainer of this dossier:** <name>
- **One-line objective:** <the independent compatible capability we aim to reach>

---

## A. Product & version archaeology

Enumerate versions, release chronology, editions, optional modules, installation
media, architectures, supported OS, runtime dependencies, and affected
instrument generations. A generic name ("Bluehill") is not a target — name the
exact reference versions we implement against.

| Version | Released | Edition/modules | OS/arch | Notes |
|---------|----------|-----------------|---------|-------|

- **Reference version(s) chosen:** <which exact versions, and why>
- **Free/trial/viewer editions:** <what's obtainable without full license>

## B. Binary & runtime inventory

Installed executables, DLLs, COM objects, .NET assemblies, scripting engines,
plugins, services, drivers, config stores, registries, local DBs, native comms
libraries. Purpose: find useful subsystem boundaries and which parts can be
tested independently — *before* any decompilation.

## C. Full functional inventory

Commands, menus, dialogs, experiment types, calculation methods, project
objects, scripting functions, supported hardware, import/export, failure
behaviors. Built from manuals, automation interfaces, help files, and controlled
experiments. Map each to a C1–C9 capability (see `../docs/capability-model.md`).

## D. Behavioral specification

Convert the functional inventory into *executable reference tests*: given an
input, parameters, and a version, what exact numerical result and project-state
change occurs? Include defaults, edge cases, invalid inputs, units, rounding,
error behavior, and version-dependent differences. These tests are the oracle.

## E. Reconstruction architecture

Per subsystem: covered by a mature open implementation? needs a compatibility
wrapper? needs a new algorithm? needs reverse engineering of proprietary logic?
Decide build-on-existing vs. new *here*, with evidence — and only then make
language/framework/storage/API choices. Record the open bases evaluated.

## F. Verification & release strategy

Precise compatibility claims + independent acceptance tests, tied to exact
reference-software versions. A "full support" claim requires an enumerated
feature set and passing evidence, never the absence of known bugs.

---

## Capability coverage matrix

Record the three properties independently (see the coverage triplet). Use
none / partial / full, with a link to evidence.

| Capability | Reference (observed?) | Implementation | Verification |
|------------|----------------------|----------------|--------------|
| C1 Acquisition & execution | | | |
| C2 Data compatibility | | | |
| C3 Document compatibility | | | |
| C4 Computation | | | |
| C5 Workflow semantics | | | |
| C6 Programmability | | | |
| C7 Instrument integration | | | |
| C8 User-facing application | | | |
| C9 Replacement coverage | | | |

## Reference environment

The reproducible lab for observing this product (feeds C1 + reference coverage):

- Clean VM / workstation spec; reproducible configuration.
- Versioned install + dependency inventory (from B).
- Representative native project files (provenance recorded).
- Programmable/repeatable interaction mechanism (automation API, or scripted GUI).
- Reference-output recorder (machine-readable + screenshots where needed).
- Experiment runner that varies one input at a time.
- Observation collector (filesystem / registry / process / interface).
- Artifact manifest linking every observation to its originating run.

## Legal preconditions

Lawfully licensed reference install (record license basis). Clean-room wall for
any proprietary-derived reimplementation. No copying of expressive elements (UI
chrome, icons, artwork, code) or vendor trademarks — "<vendor>-compatible," not
the product name. See `../docs/legal-basis.md`.

## Work packages

Bounded, independently verifiable units (a documented operation, a failing
parse fixture, a recovered message definition, a reference computation with a
numerical discrepancy). Each links to its reference tests.

| WP | Scope | Capability | Verification | Status |
|----|-------|-----------|--------------|--------|

## Open questions / blockers

Ranked. Mark each as resolvable-by-research vs. needs-reference-lab vs.
needs-hardware vs. needs-license.
