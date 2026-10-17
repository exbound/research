# Legal basis for compatible reconstruction

A plain-language orientation to why building independent, *compatible*
reimplementations of proprietary scientific software — and reading one's own
data out of proprietary formats — is generally defensible, and where the limits
are. This is a researched synthesis, **not legal advice**: the supporting case
law is largely US, some of it is contested or circuit-dependent, other
jurisdictions differ, and outcomes turn on specific facts. Treat everything
below as orientation to verify with counsel per target and jurisdiction, not as
settled rules.

The analysis keeps distinct things apart, because they carry very different
risk:

- **Data formats** — reading/parsing lawfully-held files: lowest risk
  (functional facts, not expression).
- **Behavioral compatibility** — matching what the software does (commands,
  results): supported by the interoperability/API case law, with limits.
- **Functional specifications** — documenting observed behavior as a spec:
  fact-finding.
- **Original implementations** — independently written code (clean-room defends
  copyright, not patents).
- **Vendor binaries & expressive assets** — compiled code, UI artwork, icons,
  help content: protected; never copied or redistributed.

Each section notes which of these it concerns.

## Formats are functional facts

File formats and wire protocols are functional — the interface specifications
and the requirements for compatibility are not protected expression. Documenting
"bytes 0–3 are a magic number, the next 4 are a little-endian length" is
fact-finding, not copying. In the US, intermediate copying by disassembly to
reach those unprotected functional elements has been held to be fair use when it
is the only way in (*Sega v. Accolade*, 977 F.2d 1510; *Sony v. Connectix*, 203
F.3d 596). In the EU, the Software Directive (2009/24/EC) Art. 1(2) excludes
ideas and principles from protection, Art. 5(3) permits black-box observation,
and Art. 6 permits decompilation where indispensable for interoperability.

## Reading your own data usually triggers no access-control question

When a data file is not behind a technological protection measure, anti-
circumvention rules simply do not apply; the only questions are copyright
(answered by the functional-facts/fair-use analysis above) and contract. In the
EU, contractual terms that forbid the Art. 5(3) observation right or the Art. 6
decompilation right are void (Art. 8), and a lawful user may even decompile to
correct errors (*Top System v. Belgian State*, C-13/20, 2021).

## Preservation has specific support

US Copyright Office §1201 triennial rulemaking grants exemptions for libraries,
archives, and museums to preserve software and software-dependent materials
(37 CFR §201.40), expanded in 2024, and for diagnosis/maintenance/repair of
devices. The Software Preservation Network and Library Copyright Alliance are
the organized proponents; right-to-repair reinforces the same "access to
maintain your own tools" principle.

## Reimplementing the application, not just the format

Reconstructing an independent, compatible application extends the same
functional-facts principle:

- **Behavior, command sets, and interfaces are not protected expression.**
  *Lotus v. Borland* (1st Cir. 1996, aff'd by an equally divided Supreme Court)
  held a menu-command hierarchy an uncopyrightable "method of operation."
  *Google v. Oracle* (2021) held reimplementing an API's declaring code for a
  new, compatible implementation to be fair use. With Sega/Connectix, this
  supports independently reimplementing *what the software does*.
- **What stays protected:** the vendor's actual code, and expressive elements —
  UI artwork, icons, chrome, help text, sample content. Reimplement behavior;
  never copy expression.
- **Trademarks are separate from copyright.** Product and company names and
  logos are marks. Describe compatibility nominatively ("reads OriginPro
  projects," "<vendor>-compatible"); do not name the implementation after the
  product or use its branding.
- **Observation requires a lawful license.** Running the reference software to
  observe its behavior is the EU Art. 5(3) right / US fair-use territory, but it
  presupposes a lawfully licensed copy and is not a license to redistribute the
  vendor's binaries.
- **Patents still bind.** A patented algorithm or workflow can read on an
  independent implementation regardless of clean-room copyright hygiene. Check
  per target before implementing.

## The line this project holds

- **In scope:** parsing lawfully held data files, reverse-engineering formats and
  instrument protocols for interoperability, recovering processing semantics,
  preserving instrument-bound capability. Act only on lawfully obtained copies,
  and record provenance.
- **Out of scope:** defeating a DRM, dongle, or activation check on
  currently-sold software to enable unlicensed use, and distributing
  circumvention tools. That is a distinct, legally fraught activity, outside the
  interoperability rationale.

## Clean-room practice

Where a format is reimplemented from a proprietary original, the defensible
method is a documented two-phase wall: an analysis phase produces a functional
specification that excludes verbatim code and symbol names; an implementation
phase builds from that specification alone. This grounds the result as
independent creation. It defends copyright, not patents — a patented algorithm
can still read on an independent implementation. Open-source readers are not
proprietary originals and may be used and extended directly.

### Primary sources

17 U.S.C. §1201; *Sega v. Accolade* 977 F.2d 1510 (9th Cir. 1992); *Sony v.
Connectix* 203 F.3d 596 (9th Cir. 2000); Directive 2009/24/EC Arts. 5–6, 8;
CJEU C-13/20 (2021); 37 CFR §201.40.
