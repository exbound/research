# Reconstruction campaign — target inventory

Each entry is a **target-level reconstruction program**, analyzed across the
full C1–C9 capability model (`../docs/capability-model.md`) — not a file
converter. The columns below are the *objective* and the *initial technical
attack*, not a claim that any functionality is currently reverse-engineered.
Each target gets its own dossier (`<target>.md`) following `TEMPLATE.md`; status
is tracked there.

> Framing note: these are **independent compatible implementations**, described
> by compatibility ("reads OriginPro projects," "<vendor>-compatible"), never as
> clones or by the products' names/branding. See `../docs/legal-basis.md`.

| # | Target | Initial technical attack | Full-program objective |
|---|--------|--------------------------|------------------------|
| 1 | Instron Bluehill / Merlin / Series IX | Characterize native experiments, calculations, protocol interfaces, test-state transitions | Independent materials-testing application for compatible machines: methods, acquisition, calculations, reporting, extensibility |
| 2 | Agilent UV-Vis ChemStation | Instrument protocol, spectral acquisition, processing, calibration | Independent workstation for the supported instrument generations |
| 3 | Waters Empower (legacy IEEE) | Instrument command/state protocols; chromatography acquisition/analysis behavior | Replace legacy chromatography control + processing for selected compatible instruments |
| 4 | OriginPro | Project grammar, fitting operations, transformations, graph semantics | Independent Origin-compatible data analysis + plotting environment |
| 5 | TA TRIOS / Universal Analysis | Legacy numerical processing, experiment structures, templates, acquisition APIs | Independent thermal/rheological analysis, instrument support expanded by device |
| 6 | NETZSCH Proteus | Thermal methods, calculations, calibration models, acquisition relationships | Independent thermal-analysis processing + instrument workstation |
| 7 | Agilent MassHunter Quant | Quantitation methods, integrations, calibration curves, batch evaluation | Independent quantitative analytical processing + compatible workflows |
| 8 | Shimadzu LabSolutions | Chromatography acquisition, processing, method definitions, instrument interfaces | Compatible chromatography workstation with native data + workflow support |
| 9 | Malvern HighScore Plus | Project semantics, phase matching, peak analysis, refinement workflows | Independent powder-diffraction analysis environment |
| 10 | Bruker DIFFRAC.EVA | Diffraction-data processing, phase identification workflows | Independent diffraction application + project compatibility |
| 11 | Renishaw WiRE | Raman acquisition formats, mapping, corrections, spectral analysis | Independent Raman spectroscopy + mapping environment |
| 12 | HORIBA LabSpec | Acquisition metadata, map operations, processing, calibration | Independent Raman processing + compatible instrument workflows |

## Shared vs. per-target

Build **per-application** compatibility implementations (native document/project
model, numerical engine, instrument adapters, workflows). Share only what proves
common: verification infrastructure (reference-experiment schemas, structured
evidence, differential tests, compatibility reports), numerical fixtures, and
provenance conventions — and, later, genuinely reusable computational components
(e.g. a shared baseline-correction or peak-analysis library across the
spectroscopy targets; shared diffraction kernels between HighScore and EVA).

Do **not** impose a universal application framework or mandatory plugin
architecture before the target behaviors are understood. Where a mature open
engine exists (SciPy, GSAS-II, OpenChrom, OpenMS, LabPlot/SciDAVis, instrument
libraries), prefer building on it; the vendor app is the behavioral reference.

## Sequencing

Research across all 12 targets can proceed in parallel. Independent
implementation ramps up per target only once its dossier has sufficiently
precise reference versions, functional inventory, behavioral spec, and
verification method. This avoids the failure mode of many impressive pull
requests only weakly connected to real compatibility.

Pilot: **#4 OriginPro** (`origin.md`) — its automation interface makes it the
best first reference environment, and two open Origin-compatible apps
(LabPlot, SciDAVis) already build on liborigin, so it also tests the
build-on-existing path.
