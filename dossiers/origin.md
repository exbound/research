# Reconstruction dossier — OriginLab Origin / OriginPro (target #4)

> Worked example dossier and campaign pilot. ESTABLISHED = cited/observed;
> HYPOTHESIS / NEEDS-OBSERVATION = to confirm on a reference lab. A dossier is a
> research record, not an engineering spec.

- **Target:** OriginLab Origin / OriginPro (same binary; edition is a license key)
- **Status:** research · **Last updated:** 2026-10-16
- **One-line objective:** an independent Origin-*compatible* scientific data
  analysis & plotting environment, reached by verifiable subsystems — native
  project compatibility first, then numerics, provenance, graphing, scripting.
- **Implementation repo:** `opju` · **Strategy:** extend LabPlot (see §E).

## A. Product & version archaeology — ESTABLISHED

- **Editions:** Origin and OriginPro are the *same* binary; OriginPro unlocks
  surface/3D fitting, STFT, advanced stats, peak analysis, signal/image
  processing. ([Wikipedia](https://en.wikipedia.org/wiki/Origin_(data_analysis_software)), [Version Comparison](https://www.originlab.com/VersionComparison))
- **Naming:** year-style since 2015; two releases/year (main ~April, "b"
  ~October); parallel internal X.Y numbers + build numbers (2026 SR1 = build
  10.300197). Release-notes index spans 2018 … 2026b. ([release notes](https://www.originlab.com/ReleaseNotes/versions.aspx))
- **Confirmed internal versions:** 2018=9.5, 2018b=9.55, 2019=9.6, 2019b=9.65,
  2020=9.7, 2020b=9.75; 2026≈10.3 (anchored by build 10.300197). 2021–2025
  values inferred (+0.05/half-year) — **NEEDS-OBSERVATION** (About dialog).
- **`.opju` introduced in Origin 2018 and made default** (Unicode successor to
  `.opj`). Critically: saving ".opj" from 2018+ writes `Name.opj.opju` unless an
  explicit legacy save is used (LabTalk `save -dij`/`-x`). ([fileinfo](https://fileinfo.com/extension/opju), [LabTalk Save](https://license.originlab.com/doc/LabTalk/ref/Save-cmd))
- **Reference versions chosen:** 2018 (9.5 — first `.opju`), a mid-range
  (e.g. 2022/2023), and current 2026. 32-bit dropped at 2020; 2020+ is x64-only;
  current supports Win10/11 x64.
- **Free editions:** **Origin Viewer** (no install, reads `.opju`+legacy,
  converts `.opju`→`.opj`, **read/convert only — no compute/script**); a
  time-limited full trial exists. ([Viewer](https://www.originlab.com/viewer))

## B. Binary & runtime inventory — PARTIAL (ESTABLISHED + gaps)

- Native **C/C++ core** exposing **COM/ActiveX** automation (INFERRED-high: the
  embedded "Origin C" C/C++ language + COM server). Requires **MSVC 2015–2022
  redistributables** (x86+x64); a bad MSVC redist caused 2024b–2025b startup
  crashes. ([FAQ-1228](https://docs.originlab.com/quick-help/origin-crash-on-startup/))
- Registered COM servers: Orglab (`OrgLab9.dll`+`orglab9.tlb`) and Origin's own
  automation server. **Embedded Python** (~3.11) bundled; **OpenGL** for 3D.
- OriginLab publishes an `SBOM.txt` (third-party components) — **NEEDS-OBSERVATION**
  (both fetch routes failed; pull from an installed copy). Whether any .NET/CLR
  ships and the GUI toolkit (MFC vs Qt) are **UNKNOWN**.

## C. Functional inventory — map ESTABLISHED, behavior is reference-lab work

Worksheets/matrices + column/cell-formula model (cell formulas since 2018);
**NLFit** nonlinear fitting (large function library) + linear/poly/multiple
regression (OriginPro adds surface/global fitting); statistics (tests, ANOVA);
signal processing (FFT, wavelet; STFT in Pro); graphing engine (2D + OpenGL 3D,
templates); **Analysis Templates**; **Project Explorer + the "Recalculate"
analysis-operation system** (stored operations recompute on input change:
Auto/Manual/None) — this is central to `.opj`/`.opju` semantics and the hardest
subsystem to reproduce without the engine; import/export (text, Excel, SPSS,
databases via ADO, instrument formats). Map each to C1–C9 in the matrix below.

## D. Behavioral specification — TO BUILD (reference-lab)

Convert C into executable reference tests via the harness in §"Reference
environment": given an input project + parameters + version → exact numerical
result and project-state change. Priority first targets: a single NLFit on a
known dataset (parameters, covariance, R²) across two Origin versions; a column
formula with units; the Recalculate dependency behavior.

## E. Reconstruction architecture — DECIDED (benchmarked 2026-10-16)

**Extend LabPlot; contribute upstream.** Evidence (full benchmark in campaign
notes):

- LabPlot and SciDAVis are open, Origin-like analysis/plot apps on liborigin.
  **LabPlot is actively developed** (Qt6; a **Python SDK + headless `--no-gui`
  CLI** — the automation substrate we need); SciDAVis's last release is 2022.
- The **live liborigin is LabPlot's vendored fork** (`src/3rdparty/liborigin`,
  patched Oct 2026); the `SciDAVis/liborigin` repo is **dormant since 2021**.
- **Neither reads `.opju`** (liborigin tops out ~Origin 2017 legacy `.opj`),
  **neither imports fitting provenance**, both are **import-only**. These are
  net-new — the core of our contribution.

Subsystem plan:
- **Native format:** OPJU read (then write) as a *cleanly separable module* in
  LabPlot's liborigin fork — effectively "liborigin v2". Keep **GPL-3-compatible**
  (liborigin GPL-3; LabPlot GPL-2-or-later) so it can be split out later.
- **Import mapping + fitting provenance:** LabPlot's import layer.
- **Numerics:** LabPlot (and liborigin consumers) use **GSL**; match Origin
  NLFit behavior against it rather than rebuilding solvers.
- **Graphing gap:** LabPlot's modern line appears **2D-only** (SciDAVis retains
  3D) — HYPOTHESIS-high; if Origin 3D/surface parity is in scope it's a separate
  large effort.

## F. Verification & release strategy

Per-capability compatibility claims tied to exact Origin reference versions,
each backed by passing reference tests (never "no known bugs"). Publish as small
units: a failing OPJU parse fixture, one recovered record type, one NLFit
differential result. Land them upstream in LabPlot where they ship.

## Capability coverage matrix (current)

none unless noted; this is the starting state.

| Capability | Reference (observed?) | Implementation | Verification |
|------------|----------------------|----------------|--------------|
| C1 Acquisition & execution | partial — versions/editions/automation documented; installs not yet stood up | n/a | n/a |
| C2 Data compatibility | partial — legacy `.opj` structure known via liborigin; `.opju` not | partial (legacy `.opj` only, via LabPlot) | none for `.opju` |
| C3 Document compatibility | none | none (all tools import-only) | none |
| C4 Computation | none observed (needs licensed Origin) | none | none |
| C5 Workflow semantics (Recalculate) | none | none | none |
| C6 Programmability (LabTalk/Origin C/Python) | documented, not characterized | none | none |
| C7 Instrument integration | n/a (Origin is not instrument-bound) | n/a | n/a |
| C8 User-facing application | n/a — LabPlot provides the interface | reuse LabPlot | n/a |
| C9 Replacement coverage | not yet measurable | — | — |

## Reference environment — the critical design, split by license boundary

- **No-license tier (fixtures + structural cross-check):**
  - **Orglab** (free COM, 64-bit, no Origin license) authors `.opj` AND `.opju`
    with deterministic worksheet/matrix **data + column metadata**. This is the
    controlled-fixture generator. **Limits:** no graph authoring, **no
    computation**, and OriginLab warns its COM behavior can diverge from
    Origin's. ([Orglab docs](https://docs.originlab.com/orglab/orglab/), [Create a Simple OPJ](https://docs.originlab.com/com/creating-a-simple-opj/))
  - **liborigin** (LabPlot's fork) independently parses legacy `.opj` for a
    structural cross-check.
  - **Origin Viewer** (free) as a read/convert oracle (`.opju`→`.opj`).
- **Licensed tier (ground-truth computed outputs — unavoidable):** a licensed
  Origin driven by **`originpro`** (Python; Windows-only, needs Origin 2021+)
  or the COM Automation Server / LabTalk / Origin C, to produce NLFit results,
  stats, DSP, and recalculated values. ([originpro](https://pypi.org/project/originpro/), [Python docs](https://docs.originlab.com/python/))
- Harness: clean Windows VM(s) per reference version; vary-one-input experiment
  runner; machine-readable + screenshot recorder; artifact manifest linking
  every output to its run.

**Consequence:** fixture generation and format round-tripping need no license;
**behavioral/computation ground truth does.** Securing a lawfully licensed
Origin (and `originpro`) is the gating dependency for C4/C5 verification.

## Legal preconditions

Lawful Origin license for the licensed tier (record basis). Clean-room wall for
any OPJU logic derived from observing proprietary binaries. No vendor
trademarks — "reads OriginPro projects" / "Origin-compatible," never the product
name. `.opj`/`.opju` are proprietary and **undocumented**; the open path is
reverse-engineering + Orglab/Origin ground truth, not a published spec.
See `../docs/legal-basis.md`.

## Work packages (initial)

| WP | Scope | Capability | Verification | Status |
|----|-------|-----------|--------------|--------|
| O-1 | Stand up Orglab (test Python/`comtypes` drivability) + generate graded `.opju` fixtures | C1 | fixtures open in Origin Viewer | not started (needs VM) |
| O-2 | Run LabPlot's liborigin fork against the fixtures; map where OPJU parsing breaks | C2 | break map vs fixtures | not started |
| O-3 | OPJU reader module in the liborigin fork (data+structure) | C2 | value/object equivalence vs Orglab fixtures | not started |
| O-4 | Fitting-provenance import (operation tree) | C4/C5 | reopened project recomputes | not started |
| O-5 | One NLFit differential test vs licensed Origin | C4 | numeric match within tol | blocked on license |

## Open questions / blockers (ranked)

1. **Licensed Origin access** for C4/C5 ground truth — needs-license (gating).
2. **Is `.opju` a zip/OLE/custom container past the `CPYUA` line?** — needs
   reference-lab (Orglab-generated sample + hexdump). Decides reader effort.
3. **Orglab from Python** (comtypes) and exactly what it omits vs Origin —
   needs reference-lab.
4. **SBOM / binary inventory / toolkit (MFC vs Qt) / .NET presence** —
   needs reference-lab.
5. **Internal version numbers 2021–2025** — needs About-dialog check.
