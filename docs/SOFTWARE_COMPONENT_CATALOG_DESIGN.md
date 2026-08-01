# Linxira Software And Component Catalog Design

Status: design baseline, 2026-07-20

This document defines the two UI catalog surfaces and the shared tree model. It
does not make every listed item installable or eligible for the ISO. Each item
must still pass source, license, architecture, security, and VM acceptance
review before it becomes `reviewed`.

## Two Surfaces

### Official Software Manager

Manages individual applications and system utilities. A leaf is a user-
understandable software product that can be installed and observed separately.

Proposed categories and candidates:

| Category | Candidate software |
|---|---|
| Web and browser | Firefox, Chromium, Brave, LibreWolf |
| Mail and communication | Thunderbird, Element, Signal Desktop, Telegram Desktop |
| Office | LibreOffice, ONLYOFFICE, Calligra Suite, WPS Office (pending legal review) |
| Documents and reading | Okular, Evince, Foliate, Calibre, Zathura |
| Passwords and personal data | KeePassXC, Bitwarden Desktop, Seahorse |
| Graphics and illustration | Krita, GIMP, Inkscape, MyPaint |
| Photo and RAW | digiKam, Darktable, RawTherapee |
| 3D and CAD | Blender, FreeCAD, OpenSCAD |
| Desktop publishing | Scribus |
| Video and recording | Kdenlive, OBS Studio, Shotcut |
| Audio | Audacity, Ardour, LMMS, Elisa |
| Media playback | VLC, Haruna, MPV, Celluloid |
| File and sync tools | Syncthing, Nextcloud Desktop, LocalSend, Filelight |
| Development applications | Kate, Qt Creator, Code OSS, Geany, Meld, Git GUI clients |
| Database and API tools | DBeaver Community, SQLite Browser, PostgreSQL tools, Insomnia-compatible client |
| Education and geography | Stellarium, Marble, Kig, KGeography, QGIS |
| Scientific desktop applications | GNU Octave, ParaView, Veusz, SageMath interface |
| Gaming | Steam, Lutris, Heroic Games Launcher, Bottles |
| System utilities | Ark, Dolphin, Spectacle, Partition Manager, GParted, Timeshift, Flatseal |

Rules:

- Firefox is the only default-selected ordinary application.
- Other ordinary applications are initially unselected, even when marked
  recommended.
- The Office category is `bounded`; a release may set `maxSelected` to two.
- Browser, media-player, and similar alternatives can be `bounded` or
  `exclusive` when installing multiple providers would create conflicting
  defaults.
- Development applications are normally `multi` and may be fully selected.
- Gaming is a software-manager workflow, but drivers, kernels, and system
  tuning remain outside the application tree.
- A candidate with uncertain redistribution, proprietary licensing, or an
  unreviewed third-party source remains visible only in an optional review
  channel, not in the default install catalog.

### Bundle And Component Manager

Manages capabilities, runtimes, toolchains, and domain workspaces. A bundle is
not an installation artifact. It expands into component leaves and controlled
configuration operations.

Proposed top-level bundles:

| Bundle | Components and nested bundles |
|---|---|
| Runtime Management | Python Runtime, Python Environments, Miniforge/Conda, uv, Node.js and npm, Rust and Cargo, Go, Java, Julia, R |
| Developer Workstation | Git, Base Development, CMake/Ninja, Clang/LLVM, Debugging Tools, Documentation Tools, Container Tooling |
| Python Development | Python Runtime, uv, JupyterLab, Python Packaging, Developer Workstation (nested) |
| Data Science | Python Scientific Stack (nested), R Data Analysis (nested), JupyterLab, Visualization Tools, Parquet/Arrow Tools |
| Scientific Computing | Python Scientific Stack (nested), R Data Analysis (nested), Julia Scientific (nested), GNU Octave, SymPy/Sage Tools |
| AI and Machine Learning | Python ML Stack (nested), JupyterLab, CPU ML Runtime, GPU ML Runtime (hardware-gated), Experiment Tracking |
| Bioinformatics | Bioinformatics Runtime (nested), Conda/Bioconda Channels, Samtools/Bcftools, Bedtools, BWA/Minimap2, BLAST, FastQC/MultiQC, Snakemake, Nextflow, Apptainer |
| GIS and Geospatial | QGIS, GDAL/OGR, GRASS GIS, SAGA GIS, PostGIS Tools, Python Geospatial Stack (nested) |
| Engineering and Simulation | FreeCAD, KiCad, OpenSCAD, Gmsh, ParaView, OpenFOAM Tools, Engineering Python Stack (nested) |
| Research Writing | LaTeX, Pandoc, Markdown Tooling, JabRef, LyX, Citation and Bibliography Tools |
| Web and Service Development | Node.js Toolchain, Python Web Stack, Go Toolchain, Rust Toolchain, Database Tools, Container Tooling |
| Container Workstation | Podman, Buildah, Skopeo, Distrobox, Apptainer, Container Development (nested) |
| Gaming Setup | Steam, Lutris, Heroic, Bottles, Proton Tools, Gamepad Tools; driver and kernel checks only |

Rules:

- `required` children are locked selected after their parent is selected.
- `recommended` children are selected by a bundle preset but can be cleared.
- `optional` children are initially clear.
- A bundle may include another bundle by stable ID.
- A nested bundle is still expandable; selecting the parent does not erase the
  user's ability to clear non-required descendants.
- Configuration operations such as creating a Conda environment or enabling
  approved channels are explicit plan actions, never shell strings in catalog
  data.
- Hardware-specific components, proprietary drivers, kernels, and system
  updates are referenced as checks or handoffs, not silently installed by a
  scientific or gaming bundle.

## Nested Bundle Contract

The catalog graph is a directed acyclic graph. Every node has one stable ID and
one kind:

```text
application
component
bundle
operation
```

A bundle may contain `required`, `recommended`, and `optional` references to
applications, components, operations, or other bundles. It may not contain
itself, directly or indirectly. Catalog validation must reject cycles,
unknown references, duplicate references, and references that cross a forbidden
provider boundary.

The tree is a projection of this graph:

```text
Data Science [partial]
  [x] Python Scientific Stack [expanded]
      [x] Python Runtime (required)
      [x] NumPy / SciPy (recommended)
      [ ] PyArrow (optional)
  [ ] R Data Analysis
      [ ] R Runtime (required)
      [ ] tidyverse (recommended)
```

Selection is keyed by leaf ID, never by the path. If a leaf occurs through
multiple bundles, it is installed once and the plan records every
`requestedBy` path. Parent states are calculated from effective descendants.

Constraints are evaluated at the narrowest node that declares them and then
again globally during planning:

- `multi`: any number of children; parent may select all.
- `exclusive`: at most one child; selecting one clears sibling alternatives.
- `bounded`: multiple children up to `maxSelected`; parent cannot silently
  exceed the limit.
- `preset`: changes leaf defaults but is not itself installed.

The final selection document contains:

- selected leaf IDs;
- selected bundle/preset IDs;
- required and recommended provenance;
- every nested path that requested a leaf;
- explicit user overrides;
- catalog digest;
- constraint results;
- provider and source requirements.

The backend expands this document into immutable package and configuration
actions. Receipts record the expanded leaves and actions, not only the parent
bundle.

## State And Display

The software manager defaults to managed or observed installed items. It can
switch to all available, not installed, pending, unavailable, drifted, or
reboot-required items. The component manager uses the same state enum but also
shows environment and configuration status.

The checkbox represents the requested target state. A separate status badge
represents observed state. An installed item may therefore be checked with an
`installed-external` badge, or unchecked with a `drifted` warning, without
confusing fact and intent.

## Migration

Catalog v2 remains the current compatibility input while this model is
implemented. The migration must first add explicit `kind`, `primaryCategory`,
`selection`, `children`, `requires`, `recommends`, `conflicts`, `source`,
`license`, and `availability` fields. Existing `profiles[].packages[]` must be
converted into bundle members and leaf components before the old flat chooser
is removed.
