# Fast OFM modular feature index

Status: initial engineering index  
Source baseline: `a4231aa2538c0c67f306a666011abfd0de68d1db`  
Target release: `v0.3.0-prototype.1`

## Purpose

The first public-facing code release is a clean source snapshot with no imported
laboratory Git history. This index preserves modular reviewability: every
meaningful improvement has a stable identity, explicit dependencies, source and
test boundaries, evidence status and an extraction strategy.

This index is not a claim that every feature is already independent at the code
level. `Independent` means it can reasonably become a focused patch. `Layered`
means it requires named lower-level features. `Coupled` means extraction first
requires a deliberate interface boundary.

## Four primary improvement packages

These are the public, human-readable units of the Fast OFM work. The finer IDs
later in this document are subfeatures used for engineering traceability, not
separate headline projects.

### `WP-1 MOTION` — New motors and 3D-printer control board

Scope:

- NEMA 11 motor/stage configuration and lead-screw kinematics;
- MKS Robin Mini V2.0 / STM32F103 controller;
- Klipper configuration and board pin mapping;
- Moonraker-backed OpenFlexure XYZ stage;
- steps/rotation distance, direction, enable and endstop assignments;
- software motion limits, speed, acceleration and completion readback;
- local-zero workflow and explicit absence of verified homing/encoders.

Subfeatures: `STAGE-001`, `MOTION-001`, and the motion-facing part of
`FOCUS-002`.

Independent review target: reproduce XYZ movement on the specified board and
motors without requiring RG autofocus, WSI capture or stitching.

### `WP-2 ILLUMINATION` — Arduino-controlled illumination

Scope:

- Arduino Nano-compatible brightness controller;
- high-frequency PWM and EEPROM/configuration behavior;
- RED/GREEN/WHITE calibration and measured current mapping;
- division of responsibility between Arduino brightness and MKS timing gates;
- fail-safe OFF behavior and hardware current-limit requirements;
- wiring, serial protocol and safe bring-up.

Subfeatures: `LIGHT-001`, `LIGHT-002`, plus the illumination-facing part of
`CAPTURE-001`.

Independent review target: reproduce calibrated three-channel illumination
without requiring scan planning, RG focus estimation or stitching.

### `WP-3 RG-FOCUS` — Red/green autofocus system

Scope:

- prepared RED/GREEN acquisition and simultaneous RG capture;
- flat-field and spectral calibration;
- RG shift measurement, native NMI acceleration and quality control;
- calibration from measured shift to signed Z correction;
- bounded Z approach/backlash policy;
- central and peripheral tissue patch selection;
- WHITE fallback and fail-closed routing;
- sparse hardware focus anchors;
- focus map/surface construction and predicted focus between anchors;
- focus telemetry, actual/predicted points and heatmaps.

Subfeatures: `CAPTURE-001`, `AF-001` through `AF-006`, `FOCUS-001`, and
`FOCUS-002`.

Independent review target: replay calibration, measurement, focus-map updates
and prediction from released data before any hardware integration test.

### `WP-4 STITCHING` — Stitching and WSI output improvements

Scope:

- stage-position-assisted tile placement;
- bounded FFT spectrum cache;
- overlap/correlation acceleration;
- render ownership/Voronoi-style optimization boundary;
- disconnected scan-region support;
- configurable CPU/RAM resource limits;
- pyramidal OME-BigTIFF output for QuPath-compatible review.

Subfeatures: `OUTPUT-001`, `STITCH-001`, and `STITCH-002`.

Independent review target: reproduce stitching and OME-BigTIFF generation from
a released tile/coordinate fixture without microscope hardware.

## Shared integration layer — not a fifth headline improvement

`SCAN-001`, `SCAN-002`, and `UI-001` connect the four work packages inside the
current OpenFlexure prototype: route execution, tissue/background decisions,
capture orchestration, operator UI and telemetry. They must be documented and
tested, but public communication should present them as integration glue rather
than a fifth primary invention.

`SYNC-001` and `FUTURE-001` remain roadmap/experimental work and are not part of
the four demonstrated improvement packages.

## Maturity vocabulary

- `prototype-verified` — exercised on the accepted physical prototype.
- `offline-verified` — replayed/tested without claiming current hardware use.
- `test-verified` — covered by automated tests but not accepted on hardware.
- `experimental-disabled` — preserved for research, disabled in the accepted run.
- `roadmap` — design only; not implemented as a release capability.

## Feature summary

| ID | Feature | Maturity | Separability | Primary repository | Depends on |
| --- | --- | --- | --- | --- | --- |
| `STAGE-001` | Moonraker/Klipper XYZ stage backend | prototype-verified | independent | `openflexure-wsi` | upstream OpenFlexure stage interface |
| `MOTION-001` | Bounded motion policy and readback completion | prototype-verified | layered | `openflexure-wsi` | `STAGE-001` |
| `LIGHT-001` | Multi-channel illumination abstraction | prototype-verified | independent | `openflexure-wsi` | upstream Thing/config interfaces |
| `LIGHT-002` | Arduino brightness and MKS timing-gate split | prototype-verified | layered | `controller`, `hardware` | `LIGHT-001` |
| `CAPTURE-001` | Prepared JPEG/RAW measurement capture | prototype-verified | layered | `openflexure-wsi` | camera interface, `LIGHT-001` |
| `AF-001` | Red/green focus measurement core | prototype-verified | independent math core | `fast-ofm-core` | NumPy/OpenCV; optional native NMI |
| `AF-002` | RG flat-field and spectral calibration | prototype-verified | layered | `fast-ofm-core`, `openflexure-wsi` adapter | `CAPTURE-001`, `LIGHT-001` |
| `AF-003` | Simultaneous RG acquisition and unmixing | prototype-verified | layered | `fast-ofm-core`, `openflexure-wsi` adapter | `AF-001`, `AF-002`, `CAPTURE-001` |
| `AF-004` | RG model calibration and bounded Z correction | prototype-verified | layered | `fast-ofm-core`, `openflexure-wsi` adapter | `AF-001`, `AF-003`, `MOTION-001` |
| `AF-005` | Tissue-window/peripheral patch recovery | prototype-verified | layered | `fast-ofm-core` | `AF-001`, `AF-003` |
| `AF-006` | WHITE fallback and fail-closed focus routing | prototype-verified | layered | `openflexure-wsi` | `AF-004`, `AF-005` |
| `FOCUS-001` | Sparse focus anchors and local surface prediction | prototype-verified | layered | `fast-ofm-core`, `openflexure-wsi` adapter | `AF-004`, `AF-006`, `MOTION-001` |
| `FOCUS-002` | Z preload/backlash control and calibration | prototype-verified | layered | `openflexure-wsi` | `MOTION-001` |
| `SCAN-001` | Tissue-aware bounded snake/spiral scanning | prototype-verified | coupled | `openflexure-wsi` | `MOTION-001`, `CAPTURE-001` |
| `SCAN-002` | Focus strategy integration and scan telemetry | prototype-verified | layered | `openflexure-wsi` | `SCAN-001`, `FOCUS-001` |
| `OUTPUT-001` | Pyramidal OME-BigTIFF output | prototype-verified | replaceable LGPL worker | `fast-ofm-stitching-openflexure` | captured tile set, libvips |
| `STITCH-001` | Bounded FFT cache and render acceleration | offline-verified | LGPL extension | `fast-ofm-stitching-openflexure` | OpenFlexure stitching interfaces |
| `STITCH-002` | Disconnected-region local correlation | offline-verified | layered LGPL extension | `fast-ofm-stitching-openflexure` | `STITCH-001`, stage coordinates |
| `UI-001` | Operator controls for stage, illumination and focus | prototype-verified | coupled | `openflexure-wsi` | corresponding backend features |
| `SYNC-001` | MCU frame-event/scanner synchronization prototype | experimental-disabled | independent experimental package | `controller` | Klipper patch boundary |
| `FUTURE-001` | Global-shutter triggered continuous scanning | roadmap | not yet extractable | `fast-ofm` | future camera, `SYNC-001` |

## Detailed feature cards

### `STAGE-001` — Moonraker/Klipper XYZ stage backend

Problem: use a commodity 3D-printer controller as an OpenFlexure XYZ stage.

Primary source:

- `src/openflexure_microscope_server/things/stage/moonraker.py`
- `src/openflexure_microscope_server/things/stage/moonraker_status.py`

Primary tests:

- `tests/unit_tests/test_moonraker_stage.py`
- `tests/unit_tests/test_moonraker_status.py`
- `tests/unit_tests/test_moonraker_motion_timing.py`

Evidence: accepted physical scan used MKS Robin Mini V2.0 through Moonraker.

Extraction strategy: focused OpenFlexure stage-backend patch with example
configuration; no RG or WSI dependency should be required.

### `MOTION-001` — Bounded motion policy and readback completion

Problem: enforce scanner-specific software ranges, Z segment limits, completion
readback and explicit local-reference state.

Primary source:

- `src/openflexure_microscope_server/things/mapping_motion.py`
- Moonraker stage modules from `STAGE-001`

Primary tests:

- `tests/unit_tests/test_mapping_motion.py`
- `tests/unit_tests/test_moonraker_motion_timing.py`

Extraction strategy: optional policy layer over `STAGE-001`; keep machine
limits in configuration rather than module constants.

### `LIGHT-001` — Multi-channel illumination abstraction

Problem: expose independently controlled WHITE, RED and GREEN illumination with
verified transitions and fail-safe OFF behavior.

Primary source:

- `src/openflexure_microscope_server/things/illumination.py`

Primary tests:

- `tests/unit_tests/test_illumination.py`
- `tests/unit_tests/test_moonraker_illumination.py`
- `webapp/src/tests/unit/illuminationSettings.spec.js`

Extraction strategy: independent OpenFlexure Thing/configuration proposal,
without requiring RG autofocus.

### `LIGHT-002` — Arduino brightness and MKS timing-gate split

Problem: separate slow brightness/current setpoints from timing-critical light
gates.

Primary source:

- controller Arduino firmware and calibration schema;
- MKS/Klipper illumination configuration;
- hardware LED-driver and wiring documents.

Evidence: Arduino v4 at 62.5 kHz plus live MKS RED/GREEN/WHITE gates in the
accepted prototype. The public package must distinguish this from later Arduino
v5/v6 source candidates.

Extraction strategy: reusable controller/hardware reference design with a
documented serial protocol and no dependence on the WSI UI.

### `CAPTURE-001` — Prepared JPEG/RAW measurement capture

Problem: capture measurement frames without repeatedly rebuilding known camera
and illumination state.

Primary source:

- `src/openflexure_microscope_server/acquisition/jpeg_capture.py`
- `src/openflexure_microscope_server/acquisition/raw_capture.py`
- `src/openflexure_microscope_server/acquisition/measurement_preview.py`

Primary tests:

- `tests/unit_tests/test_jpeg_capture.py`
- `tests/unit_tests/test_rg_raw_capture.py`
- `tests/unit_tests/test_measurement_preview.py`

Extraction strategy: camera helper layer. Document ownership/serialization
requirements so it cannot conflict with preview streaming.

### `AF-001` — Red/green focus measurement core

Problem: estimate focus-related red/green displacement from calibrated image
patches with quality control.

Primary source:

- `fast-ofm-core: src/fast_ofm_core/focus/rg/rg_focus_core.py`
- `fast-ofm-core: src/fast_ofm_core/focus/rg/rg_focus_estimator.py`
- `fast-ofm-core: src/fast_ofm_core/focus/rg/rg_focus_field.py`
- `fast-ofm-core: src/fast_ofm_core/focus/rg/_rg_nmi.cpp`
- `openflexure-wsi: src/openflexure_microscope_server/integrations/fast_ofm_core/`

Primary tests:

- `tests/unit_tests/test_rg_focus_core.py`
- `tests/unit_tests/test_rg_focus_estimator.py`
- `tests/unit_tests/test_rg_focus_field.py`
- `tests/unit_tests/test_rg_nmi.py`

Extraction status: the estimator, field selection, native backend and QC live in
the standalone core; OpenFlexure exchanges versioned JSON and verified file
artifacts with the separate process.

### `AF-002` — RG flat-field and spectral calibration

Problem: correct spatial/color response before estimating red/green shift.

Primary source:

- `fast-ofm-core: src/fast_ofm_core/focus/rg/rg_flat_field.py`
- `fast-ofm-core: src/fast_ofm_core/focus/rg/rg_simultaneous.py`
- `openflexure-wsi: src/openflexure_microscope_server/things/focus/rg_flat_field.py`

Primary tests:

- `tests/unit_tests/test_rg_flat_field.py`
- `tests/integration_tests/test_rg_flat_field_api.py`

Extraction status: pure calibration mathematics is staged in the core; the
remaining OpenFlexure preprocessing/action path is explicitly tracked for
process migration. Publish schemas and sanitized examples, not one machine's
calibration as a default.

### `AF-003` — Simultaneous RG acquisition and unmixing

Problem: derive paired red/green measurement planes from one prepared capture
window rather than separate long host-driven exposures.

Primary source:

- `fast-ofm-core: src/fast_ofm_core/focus/rg/rg_simultaneous.py`
- `openflexure-wsi: src/openflexure_microscope_server/things/focus/rg_simultaneous.py`

Primary tests:

- `tests/unit_tests/test_rg_simultaneous.py`
- `tests/integration_tests/test_rg_focus_capture.py`

Extraction status: core math and GPL hardware orchestration are separated in
the staged repositories; the remaining local preprocessing calls must still be
replaced by coarse process operations. The method remains dependent on the
tested camera/color response.

### `AF-004` — RG model calibration and bounded Z correction

Problem: convert accepted RG displacement into a signed Z estimate and apply a
bounded correction through a controlled final approach.

Primary source:

- `fast-ofm-core: src/fast_ofm_core/focus/rg/rg_focus_calibration.py`
- `fast-ofm-core: src/fast_ofm_core/focus/rg/rg_focus_model.py`
- `fast-ofm-core: src/fast_ofm_core/focus/rg/rg_focus_control.py`
- `openflexure-wsi: src/openflexure_microscope_server/things/focus/rg_focus.py`
- `openflexure-wsi: src/openflexure_microscope_server/focus/rg/rg_focus_control.py` (wire contracts only)

Primary tests:

- `tests/unit_tests/test_rg_focus_calibration.py`
- `tests/unit_tests/test_rg_focus_model.py`
- `tests/unit_tests/test_rg_focus_control.py`
- `tests/unit_tests/test_rg_focus_autofocus.py`
- `tests/unit_tests/test_rg_focus_thing.py`

Extraction status: calibration fit and correction decision execute through the
core process; the shipped GPL correction module contains contracts only. The
remaining profile self-validation duplicate is tracked for migration.

### `AF-005` — Tissue-window/peripheral patch recovery

Problem: when the central crop lacks usable tissue, search bounded same-frame
peripheral regions rather than aborting focus immediately.

Primary source:

- RG field/core modules;
- `src/openflexure_microscope_server/things/focus/rg_focus.py`.

Primary tests:

- `tests/unit_tests/test_rg_focus_periphery.py`
- relevant RG focus field/autofocus tests.

Extraction strategy: optional patch-selection policy on top of `AF-001`; emit
telemetry for every central/peripheral decision.

### `AF-006` — WHITE fallback and fail-closed focus routing

Problem: route measurement-quality failures to bounded WHITE autofocus while
keeping calibration, hardware-state and safety failures fail-closed.

Primary source:

- RG focus Thing/control modules;
- OpenFlexure autofocus integration.

Primary tests:

- `tests/unit_tests/test_rg_focus_handoff.py`
- `tests/unit_tests/test_focus_live_failures.py`
- `tests/unit_tests/test_focus_live_normal.py`
- relevant RG autofocus tests.

Extraction strategy: publish the failure taxonomy and routing table separately
from the estimator math.

### `FOCUS-001` — Sparse focus anchors and local surface prediction

Problem: avoid focusing at every field while refusing unsafe extrapolation.

Primary source:

- `fast-ofm-core: src/fast_ofm_core/focus/focus_surface.py`
- `fast-ofm-core: src/fast_ofm_core/focus/sparse_focus.py`
- `openflexure-wsi: src/openflexure_microscope_server/focus/focus_surface.py` (wire contracts only)
- `openflexure-wsi: src/openflexure_microscope_server/focus/sparse_focus.py` (hardware cadence/fallback only)
- `openflexure-wsi: src/openflexure_microscope_server/focus/focus_scan.py`

Primary tests:

- `tests/unit_tests/test_focus_surface.py`
- `tests/unit_tests/test_sparse_focus.py`
- `tests/unit_tests/test_focus_scan.py`
- `tests/unit_tests/test_focus_scan_lifecycle.py`

Evidence: accepted run used 30 hardware RG anchors and 67 predicted-Z saved
fields, with safety anchors requested when local fits were rejected.

Extraction status: exact surface and sparse prediction mathematics execute in
the core process; the GPL workflow owns actual focus acquisition, motion,
fallback and telemetry.

### `FOCUS-002` — Z preload/backlash control and calibration

Problem: make final focus approach direction explicit and bounded.

Primary source:

- `src/openflexure_microscope_server/focus/z_backlash.py`
- `src/openflexure_microscope_server/focus/z_backlash_calibration.py`
- `src/openflexure_microscope_server/things/focus/z_backlash.py`

Primary tests:

- `tests/unit_tests/test_z_backlash*.py`
- `tests/integration_tests/test_z_backlash_api.py`

Extraction strategy: independent stage capability with a generic move adapter.

### `SCAN-001` — Tissue-aware bounded snake/spiral scanning

Problem: visit a bounded operator-approved region, skip background fields and
support efficient snake/spiral routes.

Primary source:

- `src/openflexure_microscope_server/things/scanning/scan_workflows.py`
- `src/openflexure_microscope_server/things/scanning/smart_scan.py`
- scan planner and contract modules.

Primary tests:

- `tests/unit_tests/test_scan_workflows.py`
- `tests/unit_tests/test_smart_scan.py`
- scan planner/physical-unit tests.

Extraction strategy: currently coupled to the prototype workflow/UI. First
separate route generation, tissue decision and acquisition callbacks.

### `SCAN-002` — Focus strategy integration and telemetry

Problem: select hardware RG, predicted Z, WHITE fallback or skip per route
decision and preserve evidence for later analysis.

Primary source:

- scan workflow modules;
- focus scan/surface modules;
- runtime summary and telemetry paths.

Primary tests:

- `tests/unit_tests/test_scan_focus_strategy.py`
- `tests/unit_tests/test_focus_runtime_summary.py`
- focus lifecycle tests.

Extraction strategy: event schema plus focus-strategy interface, separate from
the specific scan wizard.

### `OUTPUT-001` — Pyramidal OME-BigTIFF output

Problem: produce a viewer-compatible tiled pyramid instead of a giant flat JPEG.

Primary source:

- `fast-ofm-stitching-openflexure/src/fast_ofm_stitching_openflexure/pyramidal.py`;
- GPL manifest/process integration in
  `openflexure-wsi/src/openflexure_microscope_server/stitching/stitching.py`.

Primary tests:

- `fast-ofm-stitching-openflexure/tests/test_pyramidal.py`;
- `openflexure-wsi/tests/unit_tests/test_stitching.py`.

Evidence: accepted 4K scan produced a QuPath-verified pyramidal OME-BigTIFF.

License boundary: the final-output backend is an independently replaceable
LGPL-3.0-only worker. The GPL server sends a verified manifest through the
separate noncommercial core process; it does not import the worker or core.

### `STITCH-001` — Bounded FFT cache and render acceleration

Problem: avoid recomputing spectra for every neighbour and avoid all-image mask
comparisons during render.

Primary source:

- `fast-ofm-stitching-openflexure/src/fast_ofm_stitching_openflexure/acceleration.py`;
- process policy in `fast-ofm-core/src/fast_ofm_core/stitching/service.py`;
- GPL adapter in
  `openflexure-wsi/src/openflexure_microscope_server/stitching/stitching.py`.

Primary tests:

- `fast-ofm-stitching-openflexure/tests/test_acceleration.py`;
- `fast-ofm-core/tests/test_stitching_service.py`;
- `openflexure-wsi/tests/unit_tests/test_stitching.py`.

License boundary: algorithms adapted from `openflexure-stitching` remain in the
replaceable LGPL worker. Generic improvements may later be proposed upstream as
an explicitly authorized contribution series.

### `STITCH-002` — Disconnected-region local correlation

Problem: stitch multiple non-touching tissue regions from one scan without
inventing image-overlap edges between islands.

Primary source:

- `fast-ofm-stitching-openflexure/src/fast_ofm_stitching_openflexure/acceleration.py`;
- neutral tile/job contracts in `fast-ofm-core`.

Primary tests:

- disconnected-region cases in stitching tests.

Reuse strategy: graph/component extension over `STITCH-001`, with a small
multi-island fixture and stage-coordinate anchoring policy; keep the
OpenFlexure-dependent implementation under LGPL-3.0-only.

### `UI-001` — Operator controls for stage, illumination and focus

Problem: expose machine limits, illumination, focus readiness and scan strategy
without requiring API calls.

Primary source:

- stage, illumination, RG and scan Vue components;
- corresponding webapp stores/tests.

Extraction strategy: split by backend feature. Restore upstream OpenFlexure
branding before any UI patch is proposed.

### `SYNC-001` — MCU frame-event/scanner synchronization prototype

Problem: move trigger/light/Z scheduling toward deterministic MCU timing and
emit frame-coordinate events to a replaceable compute host.

Primary source:

- controller `scanner_sync` host and MCU sources;
- simulator, protocol fixtures and readiness checks.

Maturity boundary: disabled and unused by the accepted scan. Published only as
experimental code/evidence, never as a proven continuous-scan feature.

Extraction strategy: separate Klipper proposal after protocol and hardware
trigger validation.

### `FUTURE-001` — Global-shutter triggered continuous scanning

Problem: acquire sharp fields during continuous serpentine motion while a Pi
or another compute host updates future focus corrections asynchronously.

Maturity: roadmap only.

Required future evidence:

- selected 4K global-shutter sensor and trigger semantics;
- prescan-derived tissue boundary and sparse focus-point plan;
- versioned route transfer and bounded future-segment update semantics;
- motion-blur and exposure budget;
- MCU event-to-frame identity including coordinates and `RG`/`WHITE` mode;
- focus-compute latency distribution under acquisition load;
- bounded stale-map behavior;
- end-to-end 15 x 15 mm timing.

Registration remains required until closed-loop stage metrology and
camera-to-stage calibration prove direct placement within a declared
image-space error budget. Encoder coordinates are a future registration prior,
not a current replacement for stitching.

## Per-feature public documentation template

Every released feature will eventually have `docs/features/<ID>.md` containing:

1. problem and non-goals;
2. maturity/status;
3. public API/configuration;
4. dependency IDs;
5. source and test files;
6. evidence and measured limits;
7. failure/safety behavior;
8. clean-room reproduction steps;
9. extraction/upstream notes;
10. change log beginning with the clean release, not the private history.

## Independent review workflow

To review one feature separately:

1. select its feature ID;
2. include the minimum dependency closure from the summary table;
3. generate a focused diff from the selected upstream base or clean release;
4. run only the indexed tests plus shared baseline tests;
5. attach the indexed evidence and limitations;
6. submit a dedicated branch/patch/MR without unrelated branding or UI changes.
