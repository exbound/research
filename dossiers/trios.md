# Reconstruction dossier — TA Instruments TRIOS (target #5)

> Worked from the kickoff issue (#7) and the first verified findings of
> 2026-11-07. ESTABLISHED = observed on the reference environment or cited from
> a public source; NEEDS-OBSERVATION = the next fixture grade. A dossier is a
> research record, not an engineering spec.

- **Target:** TA Instruments TRIOS — instrument control + thermal/rheological
  analysis (thermal analysis, rheology, physical testing)
- **Status:** research · **Last updated:** 2026-11-07
- **One-line objective:** an independent TRIOS-*compatible* thermal/rheological
  analysis environment — native `.tri` data support first (C2), then document
  (C3), computation (C4), workflow (C5), reaching device-by-device instrument
  coverage.
- **Kickoff:** issue #7 — "Reconstruction kickoff: TA TRIOS (target #5) —
  reference software is a free public download".

## First verified findings (2026-11-07)

All facts below were established 2026-11-07 on the reference environment in §A,
by observing the vendor application's own output on self-authored data and
cross-referencing it against the published documentation of an independent
open-source reader ([OpenReadout](https://openreadout.github.io/openreadout/),
MIT/Apache-2.0, [not vendor-affiliated](https://github.com/openreadout/openreadout/blob/main/TRADEMARKS.md)).
OpenReadout's TRIOS `.tri` layout page:
<https://openreadout.github.io/openreadout/formats/ta-trios.html>.

**G0 fixture (the first graded fixture).** A 7-row, two-column CSV authored by
us:

    Time,Torque
    0,0
    1,1
    -1,-1
    0.5,0.5
    3.14159,3.14159
    1000000,1000000
    0.0000001,0.0000001

imported through TRIOS's documented **Text Import Wizard** (a single-page
wizard, per the vendor's help: delimited or fixed-width, `.txt`/`.csv`/`.prn`,
header row configurable, column variables user-assigned, one-step tests only;
comma-delimited was auto-detected), with the columns assigned to **Run time
[s]** and **Oscillation torque [µN·m]**. The imported result was then saved
natively:

- size **29,116 bytes**, sha256
  `3340e88ec8787f75b5d00cf1d3677e272f9784a6464339e3a075a59dcc4cc80b` (the
  fixture file itself — not its CSV input — is hash-identified).

**What the natively saved file contains** (observed in G0, cross-referenced
against the OpenReadout TRIOS 5 layout):

- The file begins with magic bytes `00 25`, and the generation byte at offset 2
  is `0x0E` — the **TRIOS 5 generation**. **Current TRIOS 6.1 therefore writes
  the TRIOS 5 `.tri` format family** — the same generation that an existing
  open reader documents and parses. This is the answer to the kickoff
  question: an independent open reading path already exists for the current
  output.
- A header of key/value pairs with the documented key set, observed values
  included: `instrumenttype` (`"AresG2"`), `instrumentserialnumber`,
  `instrumentname`, `companyname`, `rundate`, `culture` (`"en-US"`), `ticks`
  (.NET ticks), `analysispassfail`, and others. NEEDS-OBSERVATION: the exact
  string-length encoding — "7-bit-length" per OpenReadout's published layout,
  to be pinned against its published source.
- An embedded **PNG thumbnail** behind the documented `01 00 01` marker.
- **Three `21 06`-prefixed GUID-tagged records** holding the step's signal data
  as float32 arrays per the published layout. GUIDs observed in G0:
  `d22bc6bd-a1ee-8347-8663-a8887d4d9e73`, `12c18b44-e63f-b74a-8e86-36dca875114c`,
  `f5720636-37f9-4b41-bd06-5b9d80d6975e`. NEEDS-OBSERVATION: the signal values
  have not yet been decoded and compared value-for-value against the source
  CSV — that is the next fixture grade.

**Independent reader landscape.** OpenReadout (Rust; the
`openreadout-thermal` crate) reads TRIOS 5 `.tri` natively, validates against
vendor exports, and documents the layout publicly. Its stated non-coverage —
our remaining format gaps:

- TRIOS 3/4 files (the 2019 generation, generation byte `0x0C`), which it
  refuses;
- variables TRIOS computes and does not store (normalised heat flow,
  derivatives, viscosities of flow steps, DMA moduli, the moduli of
  non-parallel-plate geometries);
- per-point oscillation waveforms;
- analysis results (fits, peak integrations) stored in the file.

**Oracle chain available.** The full TRIOS application itself (behavioral
reference), the vendor's documented export formats (JSON Export with a
[publicly available schema](https://software.tainstruments.com/schemas/TRIOSJSONExportSchema),
CSV, Excel, XML — JSON Export arrived in TRIOS 5.8), the vendor's own
open-source Python ingestion library ([tadatakit](https://github.com/TA-Instruments/tadatakit)),
and OpenReadout as an independent cross-check.

## A. Product & version archaeology

| Version | Released | Notes |
|---------|----------|-------|
| TRIOS 3 / 4 (incl. 4.4) | "2019 generation" per OpenReadout | legacy `.tri` layout, generation byte `0x0C`; 4.4 ships for 32-bit Windows |
| TRIOS 5.x (5.1–5.9) | 5.1 free to all; later 5.x via subscription | the `.tri` family OpenReadout documents (byte 2 = `0E`); JSON Export introduced at 5.8 |
| **TRIOS 6.1 (build 6.1.0.641)** | July 2026 (v6.1 release notes) | **reference version under test**; free full download from the vendor's own site; Excel export moved to `.xlsx`; REST API is a keyed (purchased) feature |

ESTABLISHED:

- **Free full version:** the vendor's download page links
  `TRIOS_Full_v6.1.0.641.exe` and the vendor states *"This download is a full
  version of TRIOS"* — no instrument or license required to install or run
  (kickoff issue #7; confirmed live 2026 and against the 2026-10-08 archive
  snapshot).
- **OS:** 64-bit Windows 10 (Ultimate/Enterprise/Professional) and Windows 11;
  32-bit unsupported, Home editions not supported. Reference environment:
  **Windows 11 Enterprise LTSC evaluation**, with the instrument added via the
  app's documented offline path — *"If you only intend to view and analyze
  data, click Add as Offline to connect to the instrument offline"* (vendor
  help).
- **Product scope:** per the vendor, instrument management, experiment setup
  and execution, data collection and analysis for 20+ thermal-analysis,
  rheology and physical-testing instruments (Discovery DSC/TGA/HR, ARES-G3,
  DMA850, DSC25/2500, TGA550, Coin Cell DSC, and others).
- **Programmable surface (documented, not yet characterized):** a REST API
  (keyed feature) for status/data/experiment control over HTTP, and JSON
  Export (manual, batch, command line, "Send to LIMS").

NEEDS-OBSERVATION: the full edition/module matrix, the Lite-vs-Full split, the
instrument-firmware pairing matrix, and the v6.0/v6.2 change boundaries.

## B. Binary & runtime inventory — AWAITING RESEARCH

Installed executables, .NET assemblies, COM/COM-server surface, the bundled
Python (v6.1 ships Python 3.12.9 per the vendor release notes), drivers and
config stores — to be inventoried on the reference VM *before* any deeper
analysis, to find subsystem boundaries that can be tested independently.

## C. Functional inventory — AWAITING RESEARCH (map from vendor help; behavior
is reference-lab work)

Documented surface to inventory: Text Import Wizard (characterized in G0),
procedures/guided methods, analysis models (90+ per the vendor), the
AutoPilot workflow blocks, JSON/CSV/Excel/XML export, instrument connection
(online/offline), reporting, and the REST API. Each item maps to C1–C9 in the
matrix below.

## D. Behavioral specification — TO BUILD (reference-lab)

Executable reference tests, given input + parameters + version → exact
numerical result and project-state change. The graded-fixture ladder starts at
G0 (above): grade the next fixture by **decoding the three GUID-tagged signal
records of G0 and comparing value-for-value against the source CSV** (units
µN·m as assigned, vs. OpenReadout's documented SI storage), then add
multi-step procedures, real instrument types, and the computed-variable and
analysis-result content OpenReadout does not decode.

## E. Reconstruction architecture — EARLY DECISIONS

- **Native format (C2):** build on / alongside the published OpenReadout TRIOS
  5 layout as the independent cross-check, not as the implementation base
  (licensing and maintainability still to be weighed against implementing the
  reader ourselves against the same fixtures). OpenReadout's known gaps —
  legacy 0x0C files, computed-not-stored variables, per-point oscillation
  waveforms, stored analysis results — define this target's format frontier.
- **Data interchange:** the vendor's own JSON Export + public schema is a
  second oracle and possibly a compatibility path; the native `.tri` remains
  the primary compatibility claim.
- **Subsystems needing new work:** C4 analysis models and C5 workflow/
  AutoPilot semantics have no mature open implementation identified yet.
- Open bases evaluated: OpenReadout (Rust, permissively licensed); tadatakit
  (vendor Python, JSON side only); SciPy-class numerical engines for the C4
  kernel where the behavior is standard numerics.

## F. Verification & release strategy

Per-capability compatibility claims tied to the exact reference build
(6.1.0.641) and to a second version once v6.x coverage is widened. Evidence
units: failing/passing `.tri` parse fixtures with recorded hashes (G0 is
the first), value-for-value signal comparisons against CSV imports,
differential analysis results against TRIOS exports (JSON per public schema,
CSV, Excel), and OpenReadout cross-checks on shared files. "Full support"
requires an enumerated feature set + passing reference tests, never the
absence of known bugs.

## Capability coverage matrix (current)

Coverage triplet, recorded independently; none unless noted. Treat as a
coverage sketch — no enumerated feature set exists yet.

| Capability | Reference (observed?) | Implementation | Verification |
|------------|----------------------|----------------|--------------|
| C1 Acquisition & execution | partial — free full v6.1.0.641 installed and operated offline on Win11 LTSC eval | n/a | n/a |
| C2 Data compatibility | partial — native `.tri` family (TRIOS 5 generation) established on a self-authored fixture; cell data not yet decoded end-to-end | none | none |
| C3 Document compatibility | awaiting research | none | none |
| C4 Computation | awaiting research | none | none |
| C5 Workflow semantics | awaiting research | none | none |
| C6 Programmability | documented only (REST API, JSON Export schema) | none | none |
| C7 Instrument integration | offline-entry path documented/observed; no hardware | none | none |
| C8 User-facing application | awaiting research | none | none |
| C9 Replacement coverage | not yet measurable | — | — |

## Reference environment

- **VM:** Windows 11 Enterprise LTSC evaluation, clean install, TRIOS
  6.1.0.641 (free full download), instrument connected via the documented
  "Add as Offline" path so no hardware is required.
- **Fixture source:** self-authored CSVs imported through the documented Text
  Import Wizard, saved natively — provenance recorded per fixture (G0 above).
- **Recording:** file sizes + sha256 per saved file; header/record inventory
  per grade; vendor exports (JSON/CSV/Excel) captured alongside the native
  file; OpenReadout run over the same file as the independent cross-check.
- **Runner:** one input varied at a time (row count, value range, column
  assignment, step count, instrument type).

## Legal preconditions

Lawful reference install: the vendor's own free full download (no license key
required). Clean-room wall for any logic derived from the proprietary binary.
Compatibility described nominatively — "reads TRIOS `.tri` data" /
"TRIOS-compatible" — no vendor trademarks in an implementation name.
See `../docs/legal-basis.md`.

## Work packages (initial)

| WP | Scope | Capability | Verification | Status |
|----|-------|-----------|--------------|--------|
| T-1 | G0 grade: decode the three signal records; compare values against the source CSV (µN·m → SI storage) | C2 | value-for-value match | next |
| T-2 | Pin the header key/value encoding (string-length scheme) against OpenReadout's published source | C2 | byte-exact header parse | next |
| T-3 | Graded corpus: multi-step procedures, real instrument types, computed variables, analysis results | C2/C4 | fixtures diffed vs. exports + OpenReadout | planned |
| T-4 | Legacy generation (`0x0C`, TRIOS 3/4) sample acquisition + layout | C2 | sample file with provenance | needs-reference-lab |
| T-5 | Analysis-model differential tests (one model to start) | C4 | numeric match vs. TRIOS | planned |
| T-6 | REST API / JSON Export characterization | C6 | recorded API/JSON behavior | planned |

## Open questions / blockers (ranked)

1. **Signal values vs. source CSV** — are the stored float32s exactly the
   imported values (or a unit-converted form)? Resolvable-by-research (T-1).
2. **Header encoding** — exact string-length scheme; pin against
   OpenReadout's published source. Resolvable-by-research.
3. **What TRIOS computes vs. stores** — which of the 90+ analysis outputs
   land in the `.tri` (the OpenReadout gap list is the hypothesis to test).
   Needs-reference-lab (T-3).
4. **Legacy `0x0C` samples** — no public corpus; need a TRIOS 3/4 install or a
   depositor file. Needs-reference-lab.
5. **REST API surface** — keyed feature; behavior observable only with the
   feature enabled. Needs-license/decision on whether to acquire it.
6. **Edition/module boundaries** (Lite vs Full; analysis-model gating by
   instrument type). Resolvable-by-research.
