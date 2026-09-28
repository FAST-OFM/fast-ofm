<p align="center">
  <a href="https://github.com/FAST-OFM">
    <img src="https://github.com/FAST-OFM.png?size=200" alt="Fast OFM logo" width="132">
  </a>
</p>

<h1 align="center">Fast OFM</h1>

<p align="center">
  <strong>Modular whole-slide imaging research around the OpenFlexure Microscope</strong>
</p>

<p align="center">
  <a href="https://github.com/FAST-OFM/fast-ofm">Project index</a> ·
  <a href="https://github.com/FAST-OFM/openflexure-wsi">WSI integration</a> ·
  <a href="https://github.com/FAST-OFM/fast-ofm-core">Core</a> ·
  <a href="https://github.com/FAST-OFM/fast-ofm-stitching-openflexure">Stitching worker</a> ·
  <a href="https://github.com/FAST-OFM/controller">Controller</a> ·
  <a href="https://github.com/FAST-OFM/hardware">Hardware</a>
</p>

<p align="center">
  <sub>Research prototype · Not clinically validated · Not for diagnostic use</sub>
</p>

Fast OFM is Alexander Fridman's research project exploring a faster, reusable
whole-slide-imaging workflow around the OpenFlexure Microscope. I am releasing
this checkpoint to support accessible digital microscopy.

The current release is a **prototype checkpoint**, not a finished scanner and
not a medical device. It packages four improvements so that each can be read,
tested and reused separately:

1. MKS/Klipper/Moonraker control of the existing NEMA 11-driven stage.
2. Three-channel illumination with Arduino brightness control and MKS timing
   gates.
3. Red/green autofocus, calibration, sparse focus maps and predicted focus.
4. Faster stitching, disconnected scan regions and pyramidal OME-BigTIFF
   output.

The current prototype is host-orchestrated and stop-and-shoot. Its measured
scan baseline was collected on Raspberry Pi 4; the current deployment uses
Raspberry Pi 5, but a matched Pi 5 timing has not been recorded. Replacing the
Pi-oriented Sangaboard appliance path with a USB-connected MKS/Klipper
controller also makes the compute host replaceable: a future deployment may
use another Linux computer and a selected USB industrial camera after
validation.

A future architecture would place motion, illumination, Z scheduling and
camera-trigger timing on a deterministic controller timeline. The host would
derive tissue and sparse-focus geometry from a prescan, submit a versioned
route, consume coordinate-labelled `RG`/`WHITE` frame events and return bounded
focus-surface updates for future route segments. A companion MCU is a fallback
if the MKS path cannot provide the required trigger semantics. None of this is
a claimed release capability, and the potential greater-than-2× throughput
gain remains an unmeasured engineering target.

## Repository map

| Repository | Purpose |
| --- | --- |
| [`FAST-OFM/fast-ofm`](https://github.com/FAST-OFM/fast-ofm/tree/v0.3.0-prototype.1) | Project index, architecture, status, release manifest and roadmap |
| [`FAST-OFM/openflexure-wsi`](https://github.com/FAST-OFM/openflexure-wsi/tree/v0.3.0-prototype.1) | Modified GPL OpenFlexure server/UI and hardware/workflow adapters |
| [`FAST-OFM/controller`](https://github.com/FAST-OFM/controller/tree/v0.3.0-prototype.1) | MKS/Klipper and Arduino controller source, configuration and experimental sync work |
| [`FAST-OFM/hardware`](https://github.com/FAST-OFM/hardware/tree/v0.3.0-prototype.1) | Motion, illumination, wiring, optics and safe bring-up evidence |
| [`FAST-OFM/fast-ofm-core`](https://github.com/FAST-OFM/fast-ofm-core/tree/v0.1.0-prototype.1) | Independently runnable RG, calibration, focus-surface and route-policy process |
| [`FAST-OFM/fast-ofm-stitching-openflexure`](https://github.com/FAST-OFM/fast-ofm-stitching-openflexure/tree/v0.1.0-prototype.1) | Replaceable LGPL registration, rendering and OME-BigTIFF worker |

All repositories are kept private during release preparation. Repository
visibility is controlled by the owner and is not changed by the release tools.

## Start here

- [STATUS.md](STATUS.md) — what is verified, offline-only, experimental or
  roadmap, including measured timings and explicitly labelled extrapolations.
- [FEATURE_INDEX.md](FEATURE_INDEX.md) — stable feature IDs, dependencies,
  source modules, tests and extraction strategy.
- [ARCHITECTURE.md](ARCHITECTURE.md) — the current prototype and its component
  boundaries.
- [REPRODUCING.md](REPRODUCING.md) — order for software replay and bounded
  hardware validation.
- [ROADMAP.md](ROADMAP.md) — future global-shutter/MCU synchronization work.
- [RELEASE_MANIFEST.yaml](RELEASE_MANIFEST.yaml) — immutable source identities
  and measured prototype facts.
- [VALIDATION.md](VALIDATION.md) — exact source-only RC checks and deferred
  physical/publication gates.
- [DISCLAIMER.md](DISCLAIMER.md) — clinical, safety and non-endorsement limits.
- [LICENSING_BOUNDARY.md](LICENSING_BOUNDARY.md) — noncommercial Fast OFM
  material and mandatory GPL exceptions.

## OpenFlexure attribution

The microscope server work is a modified distribution of the OpenFlexure
Microscope Server under GPL-3.0. The OpenFlexure identity, license and upstream
attribution are retained in `openflexure-wsi`. Fast OFM is independent and is
not affiliated with or endorsed by the OpenFlexure project.

## License summary

Original Fast OFM documentation, controller code and hardware documentation
are source-available for noncommercial use. Independently authored algorithm
implementation in `fast-ofm-core` uses PolyForm Noncommercial 1.0.0; its
neutral protocol schemas use Apache-2.0. OpenFlexure and Klipper-derived
material remains GPL-3.0-only, while the replaceable stitching worker remains
LGPL-3.0-only. The exact boundary is documented in
[LICENSING_BOUNDARY.md](LICENSING_BOUNDARY.md); commercial use of original Fast
OFM material requires a separate written license.
