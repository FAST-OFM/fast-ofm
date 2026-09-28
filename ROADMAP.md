# Fast OFM roadmap

Roadmap items are design directions, not release claims.

## R1 — Harden the current stop-and-shoot prototype

- complete clean-install and saved-scan replay validation;
- resolve exact Pi/OS/kernel identity for the release environment;
- capture machine current limits, wiring photos and safe bring-up measurements;
- add verified homing before treating coordinates as persistent;
- publish a privacy-reviewed replay fixture and focus-surface visualizations;
- upstream independently reviewable stitching improvements.

## R2 — Global-shutter capture and deterministic timing

- select a 4K global-shutter camera with an external trigger and a sustainable
  triggered-frame transport mode;
- derive tissue boundaries and either adaptive or gridded sparse-focus points
  from a prescan on the compute host;
- compile the route, focus schedule and initial bounded focus-surface state into
  a versioned, checksummed controller plan;
- first evaluate the existing MKS/Klipper execution timeline as the
  deterministic owner of motion-adjacent trigger and illumination events;
- use a dedicated Pico-class companion only if the MKS/Klipper interfaces
  cannot provide the required trigger, event and buffering semantics;
- define a versioned trajectory containing movement, scheduled focus actions,
  RED/GREEN focus captures, WHITE WSI captures and bounded future Z
  corrections;
- emit frame ID, commanded coordinates, illumination mode and execution status
  for every trigger;
- accept focus-surface updates only for not-yet-committed safe trajectory
  segments and reject stale or out-of-window corrections;
- keep a replaceable compute host as camera client, image worker and
  supervisory planner; Raspberry Pi 5 is the current deployment, not a hard
  architectural dependency.

The camera does not need to stream every advertised frame. It must reliably
deliver each requested triggered frame with bounded buffering and an identity
that can be matched to the MCU event.

The current cadence suggests that removing repeated settling and host round
trips may increase acquisition throughput by more than 2×. This is a target
hypothesis, not a benchmark. It must be measured against the stop-and-shoot
baseline under the same geometry, exposure and focus-quality requirements.

## R3 — Pipelined focus updates during serpentine motion

The compute host need not finish every RG calculation before the immediately
following WHITE tile. A serpentine scan can use:

- focus observations from the previous row as a prior;
- interpolation/extrapolation from the current sparse surface;
- a bounded update horizon several fields ahead;
- late corrections applied to the next safe future segment;
- WHITE fallback or rescan marking when uncertainty exceeds limits.

Validation must measure end-to-end latency, focus prediction error, camera
buffer behavior and motion blur at target speed. A nominal camera FPS alone is
not sufficient.

## R4 — Closed-loop positioning and registration reduction

- add high-resolution axis metrology and define whether each encoder measures
  motor rotation, screw position or true stage position;
- calibrate scale, orthogonality, backlash, Abbe error, optical distortion and
  camera-to-stage transform across the full travel;
- record commanded, encoder and image-derived coordinates for every validation
  tile;
- compare direct coordinate placement against local image registration under a
  predeclared pixel-error budget;
- bypass expensive correlation only when residuals remain within that budget;
  otherwise use encoder coordinates as the registration prior.

Encoders are a candidate enabler, not proof that stitching is unnecessary.
Mechanical compliance, optical distortion and trigger-position error remain
observable in image space even with accurate motor-axis feedback.

## R5 — Owned scanner implementation

- progressively extract RG math, scan planning and controller protocols behind
  stable interfaces;
- preserve compatibility and attribution while OpenFlexure remains the current
  integration host;
- move to a fully owned runtime only after equivalent behavior, calibration,
  safety and replay tests exist.
