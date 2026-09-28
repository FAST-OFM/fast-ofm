# Release-candidate validation

Release target: `v0.3.0-prototype.1`
Validation date: 2026-09-28  
Scope: source-only private release candidate

This record separates software/source validation from physical prototype
acceptance. No command in this validation flashed firmware, moved the stage,
changed illumination outputs or opened a camera.

## Results

| Area | Environment | Result |
| --- | --- | --- |
| OpenFlexure backend, standalone / with separately installed core | clean CPython 3.13 Linux x86_64 containers, 3 CPU/4 GiB ceiling | standalone: 1,398 passed, 4 skipped, 3 expected xfailed; with core: 1,705 passed, 3 expected xfailed; 0 failed |
| Fast OFM Core, source-only | private Linux x86_64 server | 397 passed, 55 native-backend skips; Ruff clean |
| Fast OFM Core, installed native wheel/package | CPython 3.11 x86_64 and ARM64/QEMU environments, 3 CPU/4 GiB | 452 passed on each architecture; x86 4.13 s, ARM64 64.65 s |
| LGPL stitching worker | private Linux x86_64 server plus ARM64/QEMU SubIFD replay | 18 passed; retained 91-tile baseline, synthetic edge-preservation and QuPath pyramid gates recorded |
| Three-wheel process boundary | fresh CPython 3.11 `site-packages`, 3 CPU/4 GiB | GPL adapter → core → worker completed in 5.89 s, 365,392 KiB peak RSS; exact pixels even with an older OME-TIFF beside the input tiles |
| OpenFlexure webapp | Node 26 clean install | 334 passed, 2 skipped; ESLint, Stylelint and production build passed |
| JavaScript dependency audit | production and full dependency tree | zero reported vulnerabilities |
| Controller Python | Python 3.11 | 765 passed |
| Controller C++ safety contract | host `g++`, C++11 warnings-as-errors | passed |
| Arduino candidate | Arduino CLI 1.5.1, `arduino:avr` 1.8.6 | compiled; 8,156 bytes flash, 472 bytes global RAM |
| Arduino deployed-v4 reference | same pinned toolchain | compiled; 9,688 bytes flash, 424 bytes global RAM |
| Hardware evidence validator | offline Python tests | 79 passed |
| Structured release data | JSON/YAML parse | passed |
| Source/privacy scan | six implementation trees plus organization profile and planning records | zero embedded credential, private host-path or private-key findings; environment-variable names and parser tokens reviewed as non-secrets |
| Unexpected generated-artifact scan | six implementation trees | generated coverage, cache and interpreter artifacts removed from release staging |
| License boundary | REUSE 3.3 file maps and complete license texts | original Fast OFM material marked noncommercial; OpenFlexure/Klipper-derived material remains GPL |

## ARM64 boundary

The current core validation image is built from the digest-pinned official
`python:3.11-bookworm` base in `fast-ofm-core/validation/Dockerfile.arm64`. It
builds and installs the `_rg_nmi` extension as ARM64, then runs the complete
452-test core suite under QEMU with three CPUs and 4 GiB RAM. Camera, GPIO,
serial and live Moonraker access remain outside this image. This is an
architecture and packaging gate, not a claim that the exact Raspberry Pi OS
image, kernel or physical microscope has been validated.

## Physical evidence carried into this release

The accepted reference scan recorded 132 visited fields, 97 saved fields, 35
background skips, 30 RG-autofocus fields, 67 predicted-focus fields, zero
WHITE fallbacks and 572 seconds acquisition time. Its pyramidal OME-BigTIFF
opened in QuPath. Those facts describe one earlier prototype run; this source
validation did not repeat it and does not establish clinical, throughput or
reliability performance.

## Deferred gates

- exact Raspberry Pi model, OS image and kernel capture;
- owner-present bounded hardware smoke test;
- any public visibility change or external publication.

The data-free synthetic OME-BigTIFF is no longer deferred: QuPath 0.5.1 opened
it through Bio-Formats as a two-level pyramid (downsamples 1× and 2×), while
decoded level 0 remained pixel-exact. No real specimen dataset is included in
the source release.
