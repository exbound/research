# Open-reader landscape for proprietary scientific formats

A status map, not a to-do list: for each proprietary format, the best open tool,
how completely it reads the format, and what it leaves behind. Verified against
live project repositories/releases; corrections with a reproducible check are
welcome. Last reviewed 2026-10-16.

Coverage: **FULL** (data recovered) · **PARTIAL** (data but not all
metadata/method) · **NONE** (no open native reader).

## Well covered — reuse these, don't re-implement

| Format | Open tool | Coverage | Notes |
|--------|-----------|----------|-------|
| Bruker OPUS (FTIR) | `brukeropus` | FULL (read) | no open writer |
| PicoQuant `.ptu`/`.phu` | `ptufile`, `tttrlib` | FULL | TTTR photon data |
| PANalytical `.xrdml`, Bruker RAW | `xylib`, GSAS-II | FULL (data) | method/calibration metadata dropped |
| Nikon ND2 | `nd2` | FULL | metadata + OME-TIFF export |
| Zeiss CZI | `libczi`/`pylibCZIrw` | FULL | vendor-open, read+write |
| Leica LIF | `readlif`, Bio-Formats | PARTIAL | pixel + basic dims |
| Agilent `.d` / ChemStation | `rainbow` | FULL (data) | pure-Python; acquisition method dropped |
| SnapGene `.dna` | `sgffp`, BioPython | FULL | lossless write unverified |
| ChemDraw CDXML | RDKit | PARTIAL | structure only, not layout |
| Microscopy (many) | Bio-Formats | FULL/PARTIAL | 140+ formats → OME model |

## Partially covered — data yes, provenance no

A recurring pattern: the signal/data layer is open, the method/calibration/
operation layer is not. These are the natural provenance targets.

| Format | Open tool | Recovered | Dropped |
|--------|-----------|-----------|---------|
| OriginLab `.opj` / `.opju` | liborigin (`OriginAnyParser`) | data, windows, project tree; recognizes both formats | newer-version records; the analysis/fitting operation tree |
| XRD (various) | `xylib`, GSAS-II | 2θ / intensity, scan params | vendor method + calibration metadata |
| Gatan DM3/DM4 | HyperSpy/RosettaSciIO | data + calibration + subset of tags | full tag tree |

## Open tooling, but the decoder itself is proprietary

Worth flagging: some "open" paths wrap a closed vendor library, so the decode is
not actually free.

| Format | Tool | Caveat |
|--------|------|--------|
| Thermo `.RAW` | ThermoRawFileParser | wraps Thermo's closed RawFileReader DLL |
| Waters `.raw` (full fidelity) | ProteoWizard `msconvert` | bundles Waters' proprietary DLL; pure-open path is 2-D only |

## No open native reader

The container itself is still closed. (Export-only paths exist for some.)

- Waters Empower database (native; only ASCII `.arw` exports read)
- TA Instruments TRIOS native project (export-only)
- Mettler STARe (nothing open)
- Instron Bluehill native results (CSV export only)

## Umbrella projects worth knowing

Bio-Formats (microscopy), HyperSpy/RosettaSciIO (electron microscopy &
spectroscopy), xylib (x-y / diffraction), OpenChrom (chromatography/MS),
SpectroChemPy (spectroscopy). Anything with a large open user community tends to
be well served; thermal analysis, mechanical testing, and vendor CDS databases
are the thin spots.
