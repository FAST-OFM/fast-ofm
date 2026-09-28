# Fast OFM architecture

## Current prototype

```text
OpenFlexure web UI / scan workflow
                |
                v
Compute host: camera owner, RG focus, focus map, scan orchestration
      |                         |
      | Moonraker               | USB serial brightness
      v                         v
MKS Robin Mini V2.0          Arduino-compatible controller
Klipper motion + gates       RED / GREEN / WHITE PWM setpoints
      |                         |
      +----------+--------------+
                 v
        XYZ stage + illumination
                 |
                 v
       IMX477 stop-and-shoot tiles
                 |
                 v
 stage-assisted correlation / OME-BigTIFF
```

The Raspberry Pi is the orchestration and camera owner in the accepted
prototype. The measured scan baseline was collected on Raspberry Pi 4; the
current deployment uses Raspberry Pi 5, but no matched Pi 5 timing is claimed.
The MKS controller and the camera-facing process are separate host interfaces,
so the same software boundary can later be validated on another Linux SBC or
desktop-class host with a selected USB industrial camera.

The host commands one move, waits for completion, chooses measured or
predicted focus, captures a field and records telemetry. Klipper is not the
camera timing master in this release.

## Responsibility boundaries

### OpenFlexure integration

- operator interface and configuration;
- Moonraker stage adapter and bounded motion policy;
- illumination abstraction;
- image capture and RG autofocus;
- sparse focus-surface prediction;
- scan planning/execution and telemetry;
- stitching and pyramidal OME-BigTIFF output.

### MKS/Klipper controller

- X/Y/Z step generation;
- configured velocity/acceleration limits;
- binary RED/GREEN/WHITE gate outputs;
- motion status exposed through Moonraker.

The accepted configuration has no verified homing or encoder feedback, so the
operator establishes a local zero for each session.

### Arduino-compatible illumination controller

- high-frequency PWM brightness setpoints;
- persisted channel profile;
- no scan trajectory and no frame-timing authority.

The split lets the Arduino hold calibrated brightness while the MKS gates the
selected channels.

### Stitching/output

- stage coordinates seed overlap and component placement;
- image registration refines connected neighbours;
- disconnected regions retain their stage-frame relationship;
- bounded caches/resource settings control memory and concurrency;
- output is pyramidal OME-BigTIFF for pathology viewers such as QuPath.

## Focus path

```text
candidate field
  -> tissue/background decision
  -> valid focus-surface prediction available?
       yes: bounded predicted Z
       no: simultaneous RG measurement
             -> central tissue patches
             -> peripheral patch recovery when needed
             -> WHITE autofocus fallback if RG is invalid
  -> add accepted measurement to sparse focus map
  -> capture brightfield tile
```

The reference scan used 30 RG measurements and 67 predicted-Z fields. A
fallback implementation exists, but that run did not require a WHITE fallback.

## Future boundary

Continuous scanning changes the timing owner:

```text
prescan image
  -> host detects tissue boundary and sparse focus points
  -> host compiles versioned route + initial focus surface
  -> controller executes XY/Z, illumination and camera triggers
  -> controller emits {frame_id, XYZ, mode=RG|WHITE, status}
  -> host retrieves the matching frame
       WHITE -> durable tile storage
       RG    -> focus measurement -> bounded surface update
  -> controller applies accepted updates only to future safe segments
```

The preferred design executes the prepared trajectory, illumination schedule,
scheduled focus actions and global-shutter camera trigger on a deterministic
controller timeline. A dedicated Pico-class companion remains a fallback if
the MKS/Klipper path cannot provide the required trigger/event semantics. The
host may be a Pi, another Linux SBC or a desktop; it remains responsible for
prescan analysis, camera transport, focus-surface computation, storage and WSI
generation.

This design is documented in `ROADMAP.md` and is not part of the accepted
runtime; a greater-than-2× throughput gain remains an unmeasured target.

Controller timing also does not remove image registration. The accepted stage
reports commanded position without encoder feedback. Backlash, missed steps,
lead-screw and compliance errors, trigger uncertainty and optical distortion
can all leave image-space residuals. High-resolution encoders may eventually
reduce correlation to verification or local correction, but only after
camera-to-stage calibration and full-travel residual tests prove that direct
placement stays within the mosaic error budget.
